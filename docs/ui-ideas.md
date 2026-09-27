# Main Window

- When the Main Window opens without a saved layout, and a cluster, the cluster picker is shown
- Once a cluster is chosen, the Main Window restores the last saved layout for that cluster
- If no saved layout is available, the Main Window shows a Resource panel of all the resources available, including CRDs
- When a Main Window is open with a selected cluster, the user can select another cluster to connect to
- If more than one cluster is connected for that Main Window, the Resource panel shows a dropdown at the top to choose which cluster's resources to view
- When selecting a resource from the Resource panel, either by double-clicking or from the right-click context menu, a dockable panel is opened for that resource type
- The panel for the chosen resource shows the type of resource in its "title bar", plus the name of the cluster (if more than one is available in the window), and a namespace picker (if the resource is namespaced), plus controls (via a vertical dot menu?) and a close button
- Dockable panels can be moved around the main window, and organized by tiling, grouping, locking to a position, etc.
- The Resource panel is anchored to either the right or the left of the Main Window (user preference) and can be moved to either side as the user chooses during runtime; it can also be collapsed to recover space in the Main Window
- Dockable panels have a focus, so that keyboard shortcuts that are used are sent to that panel, and are visually distinguished as focused
- Dockable panels can be "maximized" to take up the entire Main Window (except for the Resource panel's space); only one dockable panel can be maximized at a time
- Dockable panels can be "minimized" to recover space in the Main Window (not sure how docking will work to restore them)
- Dockable panels can be resized and repositioned, and will snap to the edge of a nearby panel when close enough (snapping can be disabled in the setting, or temporarily by holding a modifier key; if disabled in settings, the modifier will temporarily enable snapping for that panel)
