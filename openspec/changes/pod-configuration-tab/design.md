# Design

## Context

- Pod detail already projects each reference as a typed `ObjectRef`: volume sources, container
  `envFrom`/`valueFrom`, and image pull secrets (`pod_detail/references.rs`). The Configuration tab
  reuses those, so it can't disagree with the links the other tabs show.
- The object viewer fetches one object as a `DynamicObject` and redacts a Secret before storing it
  (`object_detail/redact.rs`). That redaction stays for everything the panel stores, the YAML view
  included.
- `resource-links` made "Secret values are never shown" a requirement. This change replaces it, at
  the user's request, with "hidden until revealed", plus the rules below.

## Goals / Non-Goals

**Goals:**
- See a pod's whole configuration without leaving the pod.
- Reveal a Secret value when you need it, and nowhere else.
- The keyboard and the mouse both reach every card, link and reveal button.

**Non-Goals:**
- Editing ConfigMaps or Secrets.
- Copying a revealed value to the clipboard. Worth adding, but it widens where a value can go, so
  it's a separate decision.
- Showing values in the YAML view.
- Watching the objects for changes. They're read when the tab is opened, like the rest of pod
  detail.

## Decisions

- **Large ConfigMap values collapse; Secret values never do** (amendment, 2026-10-01). Secrets keep
  a single state machine (hidden until revealed, then shown in full) so "collapsed" can never be
  mistaken for "hidden". Collapse state is per value on the panel, never persisted. The object
  viewer's ConfigMap section is out of scope here; it can adopt the same rendering later.

### What the tab lists, and in what order

One card per referenced ConfigMap or Secret, deduplicated by object across all the ways the pod uses
it, and listed in first-seen order: volumes, then containers' env, then image pull secrets. A card
shows:

- its link (`ConfigMap/app-config`);
- its uses, one line each: "volume `config` mounted at `/etc/app` in `web`", "`envFrom` in
  `web`", "`LOG_LEVEL` from key `level` in `web`", "image pull secret";
- its contents: for a ConfigMap, each key with its value (a value longer than a few lines is
  collapsed behind Show, the way Tolerations is today); for a Secret, each key with its size and a
  reveal button.

An object that's missing (a Secret mounted `optional`) or forbidden shows that on its card; the
other cards still load.

### Fetching: lazily, once per panel

The referenced objects are fetched when the Configuration tab is first shown, not when the panel
opens, so a pod detail panel costs nothing extra until the tab is used. The fetch is one `get` per
object, run concurrently, with results arriving per card. A Secret's `get` result is redacted as
soon as it arrives, exactly as the object viewer does today: keys and sizes are kept, values are
dropped.

### Revealing one value

- Pressing a key's reveal button issues a fresh `get` of that Secret and keeps **only** that key's
  decoded value, as a `SecretValue` on the panel's reveal map, keyed by (Secret, key). The rest of
  the response is dropped.
- The value is shown until one of these happens: the button is pressed again, Hide Secret Values
  runs, the Configuration tab is left, or the panel closes. Any of them drops the `SecretValue`.
  Leaving the tab hiding the value is deliberate: a revealed value doesn't sit on screen behind a tab
  switch.
- Why fetch again rather than keeping the values from the first `get`: an unrevealed value then
  never exists in the process at all, which is a much smaller claim to check than "it exists but is
  never shown".
- Non-UTF-8 values show as "binary, N bytes" rather than being rendered.

### `SecretValue`: making "never logged" structural

A newtype over the decoded bytes, whose `Debug` and `Display` print `<secret: N bytes>`. The only
way to the bytes is an explicit `expose(&self) -> &str`, called in exactly one place, the text
element that draws a revealed value. It doesn't implement `Serialize`, so it can't end up in a
saved layout. A `grep` for `expose(` lists every place a value is read.

### Keyboard

- Reveal buttons are tab stops, with Enter or Space toggling one. They are the only in-body tab
  stops in the panel, and the one exception to the panels' `tab_stop(false)` convention, because a
  reveal has no other keyboard route that picks which value.
- A registry command, `pod_detail.hide_secret_values` (Hide Secret Values, gated to the panel's
  key context, no menu slot, default `h`), hides every revealed value at once. The object panel
  registers the same command in its own context.
- The Configuration tab key is `3`. See the proposal for the positional shift.
- `g` ("Go to…") lists the tab's links like any other references. They are the same `ObjectRef`s.

## Risks / Trade-offs

- [A revealed value is on screen for anyone nearby] → Deliberate, and scoped: one value at a time,
  hidden on tab switch, and a command to hide all at once. The YAML view never shows values.
- [A value in memory could reach a log through a `Debug` derive on a containing type] → That's why
  it's a `SecretValue`, whose `Debug` is the placeholder. A test asserts `format!("{:?}")` of the
  reveal state contains no fixture value.
- [Existing users' muscle memory for `3`/`4`/`5`] → A two-day-old binding set; a `keymap.toml`
  override restores the old keys, and the hint bar always shows the live ones.
- [Many referenced objects means many `get`s] → Only when the tab is opened, concurrently, and
  bounded by what one pod references (typically fewer than ten).
