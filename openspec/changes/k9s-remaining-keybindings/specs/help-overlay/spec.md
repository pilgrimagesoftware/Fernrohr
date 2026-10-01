# Spec Delta

## Purpose

Gives the user a single-keystroke, context-aware list of every command currently available and its
key binding, so unfamiliar or forgotten bindings can be discovered without leaving the keyboard.

## ADDED Requirements

### Requirement: Context-aware key hints overlay

The application SHALL provide a "show key hints" command that opens an overlay listing every
command currently available in the focused context, each with its title and current key binding,
sourced from the command registry.

#### Scenario: Open help from a resource panel

- **WHEN** the user invokes show key hints while a Pods panel is focused
- **THEN** the overlay lists every command available in that context, including Pods-panel-scoped
  commands, each showing its current binding

#### Scenario: Unbound commands are still listed

- **WHEN** a command available in the current context has no key binding
- **THEN** it still appears in the overlay, shown as having no binding

#### Scenario: Closing the overlay

- **WHEN** the overlay is open and the user dismisses it
- **THEN** the overlay closes and keyboard focus returns to where it was before the overlay opened
