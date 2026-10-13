## 1. Add Context window

- [x] 1.1 Open the add control's picker in its own rooted, resizable Add a Context window instead
      of an in-window dialog
- [x] 1.2 Add the connected context to the opener window through its window handle, then close
      the Add Context window
- [x] 1.3 Keep one Add Context window per workspace window: opening again raises it
- [x] 1.4 Close it with Escape, as the registered command `add_context.close`
- [x] 1.5 Tests: the add control and `context.add` open a window and no dialog; a second open
      raises the first; a connected context lands in the opener and the window closes; Escape
      closes it and adds nothing

## 2. Wide context list

- [x] 2.1 Let a context row shrink so a long name ellipsizes instead of pushing the tunnel
      selector out of the card
- [x] 2.2 Let the picker card fill its window between 480px and 800px
- [x] 2.3 Test: a long context name's row keeps its tunnel selector inside the card, at the
      minimum and a wide window

## 3. Docs

- [x] 3.1 Update `docs/site/latest/clusters.md` "Add another context to a window"
- [x] 3.2 List `add_context.close` in `docs/site/latest/reference/keybindings.md`

## 4. Manual in-app checks

- [x] 4.1 Panels in the opener stay usable while a slow or tunnelled context connects in the Add
      a Context window
- [x] 4.2 A connected context lands in the opener, which comes to the front, and the Add a
      Context window closes
- [x] 4.3 A real long gcloud/aws context name ellipsizes, shows in full on hover, keeps a working
      tunnel selector, and shows more when the window is widened
- [x] 4.4 Escape with the tunnel selector's popover open closes only the popover
- [x] 4.5 Closing the workspace window while its Add a Context window is open, then connecting,
      breaks nothing
