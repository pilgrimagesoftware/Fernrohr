## MODIFIED Requirements

### Requirement: Add a context to a window

The status bar SHALL offer an add control after its capsules that lists every kubeconfig context
the window does not already use, with each context's tunnel binding. Choosing one SHALL add it to
the window, reusing the existing connection if one is already open for that context, open a Pods
panel for it, and make it the Resource panel's selected cluster.

#### Scenario: Add a second context

- **WHEN** a window using `cluster-a` adds `cluster-b`
- **THEN** a `cluster-b` capsule appears, a Pods panel for `cluster-b` opens in the same dock, and
  the Resource panel lists `cluster-b`'s kinds

#### Scenario: Already-used contexts are not offered

- **WHEN** the user opens the add control in a window using `cluster-a`
- **THEN** `cluster-a` is not in the list

#### Scenario: Reuse another window's connection

- **WHEN** a second window adds a context that the first window is already connected to
- **THEN** no second connection or tunnel is started, and both windows show live data from the one connection

#### Scenario: Add fails

- **WHEN** the chosen context cannot connect
- **THEN** the Add Context window shows the connection failure, no capsule is added for that
  context, and no panel is opened for it

## ADDED Requirements

### Requirement: Add Context opens a non-modal window

In a window that already has a context, the add control SHALL open the context list in a separate
Add Context window that does not block the window it was opened from. Each workspace window SHALL
have at most one Add Context window; opening it again SHALL bring the open one to the front. Once
the chosen context connects, the Add Context window SHALL close.

#### Scenario: The workspace stays usable

- **WHEN** the user opens Add Context in a window using `cluster-a` and a connection to `cluster-b`
  is still in progress
- **THEN** the user can still focus, scroll, and use `cluster-a`'s panels in the workspace window

#### Scenario: Opening it again raises the open one

- **WHEN** an Add Context window is already open for a workspace window and the user chooses Add
  Context again in that workspace window
- **THEN** no second Add Context window opens, and the open one comes to the front

#### Scenario: A successful add closes the window

- **WHEN** the context chosen in the Add Context window connects
- **THEN** the context is added to the window it was opened from and the Add Context window closes

#### Scenario: Closing it adds nothing

- **WHEN** the user closes the Add Context window without connecting a context
- **THEN** the workspace window's contexts are unchanged
