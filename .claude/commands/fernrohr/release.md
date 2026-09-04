---
description: Release a sweetrpg/* repo - dispatch its Prepare Release workflow, auto-merge the release PR, and watch the ArgoCD rollout to healthy
argument-hint: "<repo-name> [ref]"
---

Invoke the `release-repo` skill for the repo named below. If no repo name was given, ask which
`sweetrpg/*` repo to release before proceeding - don't guess.

`$ARGUMENTS`
