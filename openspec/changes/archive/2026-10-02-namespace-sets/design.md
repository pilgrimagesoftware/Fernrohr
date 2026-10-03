# Design

## Context

`namespace-include-exclude` turns a panel's namespace scope into a set of names, and
`warp-all-to-namespace` gives a cluster context a default scope it propagates to every open
namespaced panel. What neither offers is a name: the set a user assembles for staging triage lives
in one panel, is unnamed, and has to be retyped elsewhere. This change adds the name, the place to
keep it, and the keystroke that applies it.

The pieces this change builds on are already specified but not yet implemented, so it is sequenced
after them:

- `namespace-include-exclude` - `NamespaceScope` becomes `All | Single | IncludeExclude`, and a
  "for all" include/exclude propagates across a context. This change switches panels to
  `IncludeExclude { include, names }` and reuses that propagation.
- `warp-all-to-namespace` - a per-context default namespace scope applied to every open namespaced
  panel in that context and to panels opened later. This change reuses that same propagation for
  "apply this set to the context".
- `namespace-picker-filter` - a filterable, keyboard-driven namespace list. The set editor's
  namespace list reuses it; nothing here depends on it landing first.

## Goals / Non-Goals

**Goals:**

- A named set of namespaces that survives a restart and follows the user between clusters.
- One keystroke path to switching a panel to a set, and one to switching a whole context.
- An editor that creates, renames, edits and deletes sets, fully from the keyboard.
- Nothing new to learn per set: switching is the same two keystrokes whichever set it is.

**Non-Goals:**

- Per-cluster sets, or resolving a set's names per cluster. A set is a plain list of names; a name
  that the connected cluster does not have simply matches nothing.
- Storing a reference to a set in a panel (see decision 4).
- Renaming a set while keeping its identity - there is no identity to keep, only an ordered list.
- Exclude-mode sets. A set is always "these namespaces", which is what makes one scope value enough.

## Decisions

1. **Sets live in their own preference file, ordered, as a `Vec`.**
   `preference_dir()/namespace-sets.toml` holds
   `NamespaceSetsConfig { sets: Vec<NamespaceSetConfig { name, namespaces }> }`, loaded and saved
   through the existing `config::load`/`config::save` - the same treatment `keymap.toml` gets,
   because a set list is a file the user may reasonably hand-edit. It is *not* in `workspace.toml`:
   that file describes one window's panels, while sets are meant to be reusable across windows and
   clusters. `Vec` rather than a `BTreeMap` because a set's position *is* its quick-selection digit,
   and a map would make digit 3 mean a different set as soon as an unrelated set was renamed.

2. **The name is the identity, and it must be unique.** Uniqueness is what lets the indicator report
   a set's name from the panel's scope alone (requirement "The namespace scope indicator names a
   matching set") without the panel storing a reference. Duplicate names are refused at save time,
   which is cheaper than a uniqueness check on every render of the title bar.

3. **Quick selection is one command and a digit, not one command per set.**
   The registry's `Command` is `id: &'static str` with a boxed action, so a per-set command would
   need a `String` id and a leaked or boxed action, rebuilt whenever sets change - and nine
   hand-registered actions to cover the nine digits anyway. A single `namespace_sets.switch` command
   opens a picker over the saved sets, and a digit key inside the dialog applies that set. This
   matches how `links.go_to` already turns one key into a list, keeps every set reachable however
   many exist, and needs no registry changes when a set is added or deleted. Digits are shown only
   while a namespaced panel has focus, since they are the fast path and applying anywhere else has
   nothing to act on.

4. **A panel stores the namespaces, not the set.**
   Switching resolves the set to `IncludeExclude { include, names }` at the moment of switching, and
   the panel keeps that. So deleting or editing a set cannot leave a panel pointing at something
   that no longer exists, and switching to a set is one assignment rather than a lookup on every
   refresh. The cost is that the title bar can only infer the name, by matching the panel's
   namespaces against the saved sets; hence the exact-match requirement, which is why the indicator
   falls back to a count when a set is edited out from under a switched panel.

5. **An empty set cannot be saved.** An include set of no namespaces is the same thing as "all
   namespaces" with a different spelling, so the editor refuses it rather than storing a set that
   cannot be told apart from `All`.

6. **Editing saves immediately, deleting asks first.** Adding or removing a namespace in the
   editor writes the file on the spot, so the editor has no separate Save step to forget - but
   deletion is irreversible and can throw away a carefully built list, so it confirms. A panel
   switched to the set being edited does not follow along (decision 4).

7. **Context-wide switching reuses the propagation that `warp-all-to-namespace` already needs.**
   That change has to update every open namespaced panel in the active context and set the context
   default; this change does exactly that with a set's namespaces instead of a single name. Its task
   2.2 names no shared entry point, so this change extracts one -
   `k8s::cluster::apply_scope_to_context(context_name, scope)` - and both commands call it, so the
   two features cannot drift apart on what "for all" means (cluster-scoped panels untouched, other
   contexts untouched, context default updated).

8. **The editor reuses `namespace-picker-filter`'s list.** Same filter behaviour, same escape
   semantics (clear the filter, then close), plus the additions this change needs: the filter's
   matches are marked as in-set or not rather than being toggled, and namespaces the connected
   cluster does not have are still listed from the saved set, marked as absent. Marking absent names
   matters because a set is portable - without it, switching clusters would make a set look
   incomplete.

## As built

- **No `IncludeExclude` yet.** `namespace-include-exclude` hasn't landed, so a panel's scope is
  still a sorted list of namespaces, with empty meaning all. That is already an include set, and a
  set is include-only (Non-Goals), so switching to a set is `PanelScope::scoped_to(names)`. When
  `IncludeExclude` lands, switching becomes `IncludeExclude { include, names }` in one place:
  `MainWindow::on_action_apply_namespace_set`.
- **Decision 7's helper already existed.** `warp-all-to-namespace` built its propagation as
  `MainWindow::warp_context(context_name, namespaces)`. It needs the window's open panels, so it
  lives on the window rather than in `k8s::cluster`. Apply to Context calls it directly, so the two
  features share one definition of "for all", which is what decision 7 is after.
- **The commands are global, with modifier keys:** Switch `cmd-shift-n`, Apply to Context
  `cmd-alt-shift-n`, Create `cmd-alt-n`, Edit `cmd-alt-e`, and Remove palette-only. Each gives a
  plain notice when no namespaced list has focus. None is in the menu bar, since switching and
  creating act on the focused panel.
- **The editor reuses `ui::namespace_filter`** with `without_all` (no pinned "All namespaces") and
  `element_marked`, which labels a set's namespaces the cluster lacks as "not in this cluster".

## Risks / Trade-offs

- [A set is edited away from under a switched panel] -> Intentional (decision 4): the panel keeps
  working on the namespaces it was given, and its indicator falls back to a count. Re-applying the
  set is one keystroke.
- [Sets are global, so a name can collide across clusters with different intents] -> Accepted. The
  editor shows which names the connected cluster has, so the collision is visible at edit time, and
  per-cluster sets are an easy follow-up once someone asks.
- [Two windows editing sets at once] -> Last save wins. Sets change rarely and through a short
  dialog; a file watcher or cross-window notification is not worth it yet.
- [Digits are positional, so inserting a set shifts later digits] -> Accepted for this release, and
  bounded by the picker also being fully arrow-and-Enter operable. Only the first nine are
  addressable by digit at all, which is what the requirement states.
- [The editor is a second dialog in a namespace picker-heavy title bar] -> It opens from a command
  with its own key hint, not as another entry in the picker, so the picker's own behaviour is
  unchanged.

## Migration Plan

A new optional file. First run writes an empty `namespace-sets.toml` (the `config::load` first-run
path) and the quick-selection picker opens empty until the user creates a set. No existing file
changes shape, so there is nothing to migrate and nothing to roll back: deleting the file returns
the feature to its first-run state.

## Open Questions

- Should a set remember the order the user added namespaces, so a switch reorders nothing? Currently
  namespaces are sorted and deduplicated (`PanelScope::scoped_to`), which is why matching a set is a
  sorted comparison. Keeping one ordering rule everywhere is worth more than an unused ordering.
- Should quick selection eventually bind digits directly (`cmd-1` to the first set), skipping the
  picker? That needs per-set dynamic bindings, which the current registry cannot express; the picker
  is the honest form until that changes.
