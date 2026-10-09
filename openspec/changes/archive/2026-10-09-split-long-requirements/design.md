# Design

## Context

`openspec validate --strict` rejects a requirement description over 500 characters. Only ADDED
requirements are checked when a change is written, so MODIFIED requirements have grown past the
limit over time. See [proposal.md](proposal.md).

## Goals / Non-Goals

**Goals:**

- Every main spec passes `openspec validate --specs --strict`.
- Each resulting requirement states one behavior.

**Non-Goals:**

- Changing, adding, or removing any behavior, or rewording requirements that are already under
  the limit.

## Decisions

### D1. The OpenSpec split procedure

For each over-long requirement, the delta has two parts:

- **MODIFIED:** the requirement with its exact header and every one of its scenarios unchanged.
  Its description is cut down to the one behavior that is its core, keeping the original meaning.
- **ADDED:** each behavior removed from that description becomes a new `### Requirement:` with
  normative wording (SHALL), a description of 500 characters or fewer, and at least one
  `#### Scenario:` in WHEN/THEN form. A scenario may restate behavior an existing scenario already
  covers, but never introduces new behavior.

Requirements that are already under the limit are not touched.

### D2. Work split

The specs are divided between two builders by requirement count. Each builder writes only the
delta files for its own specs, `specs/<capability>/spec.md`, on its own sub-branch. The files do
not overlap, so the sub-branches merge without conflicts.

## Risks / Trade-offs

- [A split subtly changes meaning] -> Every behavior of the original description must appear in
  exactly one resulting requirement. Reviewers compare the original text with the union of the
  split texts.
