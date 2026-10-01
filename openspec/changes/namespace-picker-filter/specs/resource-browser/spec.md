# Spec Delta

## ADDED Requirements

### Requirement: Filtering the namespace picker

A namespaced panel's namespace picker SHALL show a filter input when it opens, with keyboard focus
in it. Typing SHALL narrow the listed namespaces to those whose name contains the typed text,
ignoring case. The "All namespaces" entry SHALL remain listed first regardless of the filter.
Toggling a namespace SHALL keep the picker open with its filter text, so several namespaces can be
picked in a row. Up and Down SHALL move through the filtered entries, and Enter SHALL toggle the
highlighted one. Escape SHALL clear non-empty filter text, and close the picker when the text is
already empty. When the filter matches no namespace, the picker SHALL say so. The filter text
SHALL NOT survive closing the picker.

#### Scenario: Typing narrows the list

- **WHEN** the cluster has namespaces `default`, `kube-system`, `kube-public` and `payments`, and
  the user opens the picker and types `KUBE`
- **THEN** the picker lists "All namespaces", `kube-system` and `kube-public`, and not `default`
  or `payments`

#### Scenario: Picking several matches in a row

- **WHEN** the filter is `kube` and the user toggles `kube-system`, then `kube-public`
- **THEN** the picker stays open with the filter still `kube`
- **AND** the panel is scoped to both namespaces, with the button reading `2 namespaces`

#### Scenario: Keyboard toggle

- **WHEN** the picker is open, the filter is `pay`, and the user presses Down to highlight
  `payments`, then Enter
- **THEN** `payments` is toggled in the panel's scope

#### Scenario: Escape clears, then closes

- **WHEN** the filter holds `kube` and the user presses Escape
- **THEN** the filter is empty and every namespace is listed again
- **AND** pressing Escape again closes the picker

#### Scenario: No match

- **WHEN** the user types `zzz` and no namespace contains it
- **THEN** the picker lists "All namespaces" and a "No matching namespaces" message

#### Scenario: Filter resets on reopen

- **WHEN** the user types `kube`, closes the picker, and opens it again
- **THEN** the filter is empty and every namespace is listed
