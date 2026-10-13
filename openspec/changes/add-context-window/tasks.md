## 1. Add Context window

- [ ] 1.1 Open the add control's picker in its own rooted, resizable window instead of an
      in-window dialog, titled after the window it adds to
- [ ] 1.2 Add the connected context to the opener window through its window handle, then close
      the Add Context window
- [ ] 1.3 Keep one Add Context window per workspace window: opening again raises it
- [ ] 1.4 Tests: the add control and `context.add` open a window and no dialog; a second open
      raises the first; a connected context lands in the opener and the window closes

## 2. Wide context list

- [ ] 2.1 Let the picker card fill its window between 480px and 800px
- [ ] 2.2 Test: a long context name's row keeps its tunnel selector inside the card

## 3. Docs

- [ ] 3.1 Update `docs/site/latest/clusters.md` "Add another context to a window"
