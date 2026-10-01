# AGENTS.md

## Architecture

Fernrohr is a meta-repository: `App` is the `pilgrimagesoftware/Fernrohr-App` git submodule holding
the actual Rust/GPUI application code. This repo itself holds only the OpenSpec change proposals
and specs that drive that code's implementation.

See `docs/architecture.md` for the app's tech stack, architecture decisions (the tokio/GPUI bridge,
watch lifecycle, tunnels, Prometheus, persistence), and conventions. It mirrors
`openspec/config.yaml`'s `context:` block, which OpenSpec feeds into every proposal/apply
operation - keep both in sync when either changes.

## Workflow

Each OpenSpec change is implemented across two repos at once, on a matching branch name:
- This repo (`openspec/changes/<name>/tasks.md` only, tracking progress).
- The `App` submodule's own repo (the real code).

Work happens in matching worktrees under `worktrees/<branch-name>/` in each repo, not in the main
checkout - see `openspec/changes/bootstrap-fernrohr/` for the in-progress slice (app shell, cluster
connection, resource browser, pod logs, command system) as the reference example.

Branch names are `<issue-number>-<change-name>`, matching the change's GitHub Issue so GitHub
links the branch to it - e.g. `54-container-detail-expansion`. This follows the
`project-start-change` skill and deliberately diverges from `App/AGENTS.md`'s
`feature/description`. Small chores with no issue use a `0-` prefix (e.g. `0-bump-app`). The branch
still needs one commit per coherent step, not one per task.

Work one section at a time. Sections 1-11 of `cluster-picker-and-navigation` are done (8 and 9
share `8-resource-panel`, since every file section 9 touched was also touched by section 8); the
next branch continues from section 12.
