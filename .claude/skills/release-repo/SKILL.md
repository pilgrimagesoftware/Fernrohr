---
name: release-repo
description: Cut a release for a sweetrpg/* service or frontend repo - dispatch its "Prepare Release" workflow, auto-merge the resulting release PR, and watch the ArgoCD rollout to healthy. Use when the user asks to release, cut a release, ship, or deploy a specific sweetrpg repo.
metadata:
  type: engineering
---

# Release a sweetrpg repo

Runs `tooling/scripts/release-repo.sh <repo> [ref]` from the `platform` meta-repo, which does
the entire mechanical flow end-to-end: dispatches the repo's "Prepare Release" GitHub Actions
workflow, locates the `release/vX.Y.Z` PR it opens, arms it with `--auto --merge`, waits for the
merge, then polls the matching ArgoCD Application (`sweetrpg-<repo>`) until it reports the new
image tag and is Synced + Healthy.

```bash
tooling/scripts/release-repo.sh <repo-name> [ref]
```

`<repo-name>` accepts either form (`main-web` or `sweetrpg/main-web`). `[ref]` defaults to
`develop` - every sweetrpg repo's `Prepare Release` workflow is `workflow_dispatch`-only and
computes the next version from that branch's unreleased commits via `git-cliff`.

Run it with `run_in_background: true` and stream progress back to the user as its `==>` lines
land - the full round trip (checks, required review if any, tag, image build/push, ArgoCD
reconcile) commonly takes several minutes and the script's own polling loops account for that;
don't shorten its timeouts or re-invoke it if it looks slow.

## Preconditions

`gh`, `argocd`, and `jq` must be on `PATH` and already authenticated - the script checks for the
binaries but not auth. If either CLI 401s, ask the user to re-authenticate (`gh auth login`, or
the ArgoCD equivalent) rather than trying to embed credentials.

## When the script fails or times out

The script bails out at the first stalled step rather than guessing, since each of these has a
different real cause:

- **No `Prepare Release` run appears**: the workflow may not exist yet in that repo, or the repo
  name is wrong. Check `gh workflow list --repo sweetrpg/<repo>`.
- **Workflow run fails** (`gh run watch` exits non-zero): read the run's log
  (`gh run view <id> --repo sweetrpg/<repo> --log-failed`) - usually a `git-cliff` version-bump
  conflict or a failing pre-release check, not something to retry blindly.
- **No `release/*` PR found**: the workflow may have found no releasable commits since the last
  tag (nothing to release) - check the workflow's own summary/log before assuming a bug.
- **PR never merges**: auto-merge is armed but blocked - check required status checks
  (`gh pr checks <number> --repo sweetrpg/<repo>`) and required reviews on the PR itself.
- **ArgoCD never reports the new image tag**: the release workflow's own image-build/push or its
  `sweetrpg/kubernetes` tag-bump step may have failed, or ArgoCD hasn't picked up the change yet
  (no webhook configured for that repo - see `sweetrpg/platform`'s `docs/git-repos.md` on webhook
  coverage). Use the ArgoCD MCP tools (`get_application`, `get_application_events`) or
  `argocd app get sweetrpg-<repo>` directly to see what it's actually comparing against.
- **ArgoCD reports the tag but never goes Healthy**: the rollout itself is failing in-cluster -
  use the kubernetes MCP tools (`pods_list_in_namespace`, `pods_log`, `events_list`) on that
  repo's namespace to find the failing pod/container, the same way you'd debug any other bad
  deploy - don't just keep re-running the script.

Report back what step failed and why, not just "the script timed out."
