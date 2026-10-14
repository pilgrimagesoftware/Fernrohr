# Design

## Context

See proposal.md - Why. The current code that shapes the approach:

- `MainWindow` files every open panel under a `PanelKey` derived from its `PanelScope`: target,
  context and namespaces. `open_target_in` focuses an open panel whose key equals the request's,
  and builds one only when none matches. That equality is the whole "focus the existing panel"
  rule, for every route that opens a panel.
- Open requests carry an `OpenMode`: `Foreground` or `Background`. Rows open in the background
  through `ui::background_rows`, with `cmd`-click and middle-click, and through each list's Open in
  Background command (`cmd-enter`).
- Each panel kind saves its own state (`context_name`, `namespaces` and kind-specific fields), and
  `panels::panel_key` reads a saved panel's key back. Layout Add (`saved_layouts::load_add`)
  re-opens each saved panel through `open_target_in` in the foreground, so it relies on the
  equality rule not to duplicate an open panel.
- `selector-logs-multiple-panels` gave typed label Logs panels an identity inside their target
  (`TypedId`), because every typed open must be new. This change needs the opposite default -
  focus unless asked - for every kind, so the instance lives beside the target, not in it.

## Goals / Non-Goals

**Goals:**
- One instance mechanism for every panel kind, kept in `PanelScope` and so in `PanelKey`, with no
  per-kind special cases in the dedup rule.
- Default opens behave exactly as before.

**Non-Goals:**
- Copying a panel's live state - its sort, filter, selection, or detail view - into the new
  instance. A new instance opens as a fresh panel on the same resource and scope.
- Opening a new instance from a link or from the Resource panel directly. The global command
  duplicates the panel those open.
- Folding `TypedId` into the new mechanism. Typed label panels keep their own identity (D6).

## Decisions

### D1: `PanelScope.instance`, an ordinal

`PanelScope` gains `instance: u32`, where 0 is the first panel on a resource. `PanelKey` copies it,
so two instances never share a key. The number shown to the user is `instance + 1`. A new instance
takes the lowest `n >= 1` that no open panel with the same target and context uses. Namespaces are
left out of that check, so a later re-scope can't make two panels' keys collide. An ordinal rather
than a random id gives the titles their numbers directly, and it's stable across save and restore.

*Alternative considered:* a random id, as `TypedId`, plus a separately computed display number.
Rejected: the number would have to be recomputed from the open panels on every render, and two
windows' layouts would number the same panels differently.

### D2: Three ways to match an open panel

`open_target_in` resolves a request in one of three ways:

- **any instance** for every default request: the first open panel, in opening order, whose
  target, context and namespaces match. That's today's behavior when one instance is open.
- **a new instance** for `OpenMode::NewPanel`: never matches; builds a panel with the lowest free
  number (D1) and focuses it.
- **exactly instance N** for layout Add: matches only an open panel with the same key including
  the instance, so a saved duplicate opens beside the window's first instance while a saved first
  instance doesn't duplicate an open one.

### D3: `OpenMode::NewPanel`

The new mode travels on the existing open requests (`OpenListedObject`, and a new
`OpenPodInNewPanel` beside `OpenPodInBackground`), so the window's single open path handles it. It
always opens in the foreground. A background new instance isn't offered.

### D4: The routes

- **Global** `panel.open_new_instance` ("Open in New Panel", `cmd-shift-d`, View menu,
  `PanelLayout` group): `MainWindow` reads the focused panel's key and opens it again with
  `NewPanel`, in the same tab group. It does nothing with focus outside the dock.
- **Rows:** `object_list.open_new_panel` and `pods.open_new_panel` ("List: Open Selected in New
  Panel" and "Pods: Open Selected Pod in New Panel", `shift-enter`, in each list's
  `!Input` context), an `alt`-click handled in `ui::background_rows::row` beside the `cmd`-click,
  and an Open in New Panel item in each table's row context menu.

### D5: Saving the instance

Each panel kind's save writes `"instance"` when it isn't 0, and its restore reads it into its
`PanelScope`, defaulting to 0. `panels::panel_key` reads it the same way. A layout saved before
this change has no `instance` and restores every panel as instance 0. A shared
`panel_title::saved_instance` helper keeps the default in one place. Layout Add matches exactly
(D2).

### D6: Typed label Logs panels

A typed label panel already opens new on every request. Open in New Panel on one opens a new empty
typed panel (`LabelLogs::new_typed()`, instance 0), as Open Logs by Label does, rather than a
numbered copy that would start empty anyway. Workload Logs panels follow D1 like any other kind.

## Risks / Trade-offs

- [Every panel's save and restore must learn `instance`] → It's one shared helper call per kind,
  and a restore test per kind would be heavy, so one test per path covers it: a list and a detail
  panel round-trip through a dock dump, and an older layout restores as instance 0.
- [A re-scope can make two instances' keys equal, if the user scopes them alike] → The instance
  stays in the key, so they still differ. Equal scopes on two lists in different instances are what
  the user asked for.
- [The number of panels grows only by deliberate commands] → No cap is added.
