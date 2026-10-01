# Spec Delta

## MODIFIED Requirements

### Requirement: Filtering and sorting

A resource-browser panel SHALL provide a search box in its header that narrows or marks rows by
the typed text, as specified by "List search modes", "List search scope" and "List search
matching", and SHALL let the user sort by any displayed column, cycling ascending, descending,
and unsorted, with the current sort column and direction shown by an indicator on that column's
header.

#### Scenario: Text filter

- **WHEN** the user types `nginx` into the search box with the default settings
- **THEN** only rows whose Pod name contains `nginx`, ignoring case, remain visible
- **AND** clearing the search box restores all rows

#### Scenario: Clicking a header sorts ascending

- **WHEN** the user clicks an unsorted column's header
- **THEN** the table's rows sort by that column ascending, and that header shows an ascending
  indicator

#### Scenario: Clicking again reverses direction

- **WHEN** the user clicks an already-ascending-sorted column's header
- **THEN** the table's rows sort by that column descending, and that header shows a descending
  indicator

## ADDED Requirements

### Requirement: List search box

Every panel that lists resources of a single kind SHALL show a search box in its header. Pressing
`/` while the panel has focus SHALL focus the box. Escape in the box SHALL clear it and return
focus to the table. While the box holds text, it SHALL show the number of matching rows and the
total, such as `12 / 340`. The `/` binding SHALL be a registered, rebindable command scoped to
the list panel, so it doesn't affect the Resource panel's own filter binding.

#### Scenario: Focus from the keyboard

- **WHEN** the Pods panel has focus and the user presses `/`
- **THEN** the search box takes focus

#### Scenario: Escape clears and returns focus

- **WHEN** the search box holds text and has focus, and the user presses Escape
- **THEN** the box is emptied, every row is shown untinted, and focus returns to the table

#### Scenario: Match count

- **WHEN** 12 of 340 rows match the search text
- **THEN** the search box shows `12 / 340`

### Requirement: List search modes

The search box SHALL have a menu offering two modes. *Filter*, the default, hides rows that don't
match. *Highlight* keeps every row visible and tints the rows that match. In Highlight mode,
Enter in the search box SHALL move the table selection to the next matching row after the current
selection, and Shift+Enter to the previous one, both wrapping around the ends of the list.
Switching mode SHALL keep the search text.

#### Scenario: Highlight keeps every row

- **WHEN** the mode is Highlight and the user types `nginx`
- **THEN** every row stays visible, and only the rows matching `nginx` are tinted

#### Scenario: Jumping between matches

- **WHEN** the mode is Highlight, three rows match, the second is selected, and the user presses
  Enter in the search box
- **THEN** the third matching row becomes selected
- **AND** pressing Enter again selects the first matching row

#### Scenario: Switching mode keeps the text

- **WHEN** the search box holds `nginx` in Filter mode and the user switches to Highlight
- **THEN** the box still holds `nginx`, and all rows are shown with the matches tinted

### Requirement: List search scope

The search box's menu SHALL offer a scope that decides what the search text is matched against:
*Name* (the default) matches the resource's name; *Visible columns* matches the displayed text of
every column the table currently shows; *Labels* matches each label as `key=value`. A row matches
when any value in the chosen scope matches. The search SHALL NOT match against anything outside
these scopes, such as a resource's spec or a Secret's data.

#### Scenario: Name scope ignores other columns

- **WHEN** the scope is Name and the user types `Running`
- **THEN** a Pod whose name doesn't contain `running` doesn't match, even if its Status column
  shows `Running`

#### Scenario: Visible columns scope

- **WHEN** the scope is Visible columns and the user types `Running`
- **THEN** every Pod whose Status column shows `Running` matches

#### Scenario: Labels scope

- **WHEN** the scope is Labels and the user types `app=web`
- **THEN** a Pod labelled `app=web` matches, and a Pod named `web` without that label doesn't

### Requirement: List search matching

Matching SHALL be case-insensitive substring by default. The search box's menu SHALL offer a
*Case sensitive* toggle and a *Regular expression* toggle. With Regular expression on, the search
text is a regular expression that a value matches when it occurs anywhere in that value. Text
that isn't a valid regular expression SHALL mark the search box as an error, and SHALL leave the
rows exactly as they were with an empty search box, rather than hiding them all.

#### Scenario: Case sensitive

- **WHEN** Case sensitive is on and the user types `Nginx`
- **THEN** a Pod named `nginx-1` doesn't match

#### Scenario: Regular expression

- **WHEN** Regular expression is on and the user types `^api-.*-[0-9]+$`
- **THEN** only Pods whose name matches that expression match

#### Scenario: Invalid regular expression

- **WHEN** Regular expression is on and the user types `api-(`
- **THEN** the search box shows an error state, and every row is shown untinted

### Requirement: List search state and persistence

Each list panel SHALL keep its own search text. The search text SHALL clear when the panel
switches to a different resource kind, and SHALL survive changes of namespace scope. The menu
settings (mode, scope, Case sensitive, Regular expression) SHALL persist across restarts and SHALL
apply to every newly opened list panel. Changing them in one panel SHALL NOT change panels that
are already open.

#### Scenario: Kind change clears the text

- **WHEN** a panel's search box holds `nginx` and the panel switches from Pods to another kind
- **THEN** the search box is empty

#### Scenario: Settings survive a restart

- **WHEN** the user sets the scope to Labels and the mode to Highlight, then restarts the app and
  opens a Pods panel
- **THEN** the new panel's search box starts with scope Labels and mode Highlight
