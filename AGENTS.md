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

Branch names are `<section-number>-<slug>`, matching the numbered section of `tasks.md` being
implemented - e.g. `9-resource-panels` for section 9. This deliberately diverges from
`App/AGENTS.md`'s `feature/description`, because the section number is what ties a branch back to
its slice of the change; the branch still needs one commit per coherent step, not one per task. A
section that cannot be split from its neighbour by file may share the branch, so the name tracks
where the branch *starts*, not an exclusive range.

Work one section at a time. Sections 1-9 of `cluster-picker-and-navigation` are done (8 and 9 share
`8-resource-panel`, since every file section 9 touched was also touched by section 8); the next
branch continues from section 10.
