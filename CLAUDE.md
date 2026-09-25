# CLAUDE.md

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
