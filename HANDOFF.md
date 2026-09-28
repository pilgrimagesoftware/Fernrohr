Change in progress: cluster-picker-and-navigation
---
Need proper fonts: UI (Manrope), Terminal (Monaco?)
---
Need to flesh out design language
---
Panels that can accept keybound commands need to show them or make them discoverable
---
App needs the start of an application menu
- App
- Context (this is the "File" menu)
- Edit
- View
- Navigate
- Window
- Help
---
Several issues:
- YAML should be rendered in a monospace font
- Keybindings should be apparent or discoverable, like with k9s
- The pod listing should include more information and be rendered in a proper table
- Logs should be rendered one-per-line, scrollable vertically and horizontally, and in a monospace font
- Opening a second window and selecting a context just says that it connected, but never switches to the cluster view

I don't want to hear any complaints about "that's out of scope", because implementing these correctly is the scope. No shortcuts, no hacks, no roll-your-own, no "we'll get to that later".
---
Additionally, the resource list has no apparent way to move it to the right side of the window, which was apparently implemented in a previous session.
There's no application menu, so it's hardly discoverable how to do this.
Also, "Pods" and "Pod" should be separate panels. When I'm looking at a list of pods and open one, it should open in its own panel.
---
Resource panel still needs search/filter and organization (might already be a change spec for it)
