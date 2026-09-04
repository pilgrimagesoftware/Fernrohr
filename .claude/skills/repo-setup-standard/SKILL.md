---
name: repo-setup-standard
description: Scaffold a new or under-scaffolded repo with CI, dependabot, branch protection, community docs, and AGENTS.md/CLAUDE.md robot setup. Use whenever the user asks to "set up the repo", "add repo scaffolding", "bootstrap CI", or start a brand-new repo before implementation begins. Note: this skill was designed for multi-repo org setups; adapt the org-specific parts to your project.
metadata:
  type: engineering
---

# Repo setup standard

Bring a service repo up to a working baseline: CI, dependabot, branch protection,
community docs, and robot guidance. This is infrastructure work, separate from implementing
whatever the repo actually does - do this first, commit it on its own, then move to feature
work.

For polyglot orgs (Python, TypeScript, and others across different services), reusable
workflows may exist in a shared `github-actions` repo for the release family per language
(`<lang>-prepare-release.yaml`/`<lang>-release.yaml`/`<lang>-tag-release.yaml`). **Don't assume a
fixed stack or copy another repo's CI file blind.** This skill is discovery-first: figure out
what the target repo actually needs, then find a live sibling repo with a working example of
that pattern.

## Steps

### 1. Detect the target repo's stack

Look for the obvious signals before asking: `pyproject.toml`/`requirements.txt` (Python),
`package.json` (TypeScript/JS), `go.mod` (Go), `Cargo.toml` (Rust), `Package.swift` (Swift). If
none exist yet (a genuinely brand-new, empty repo), ask the user what the repo will be.

### 2. Find a live sibling repo with a working example

Don't hardcode a "source of truth" repo - query the org for repos and check which ones actually
have populated `.github/workflows/`:

```bash
gh repo list <org> --limit 100 --json name,url
gh api repos/<org>/<candidate>/contents/.github/workflows --jq '.[].name'
```

Prefer a sibling with the same detected stack (step 1) and workflows that ran successfully
recently (`gh run list --repo <org>/<candidate> --limit 5`) over one that merely has files
present but stale/broken. If more than one plausible sibling exists, ask the user which to model
this repo on.

Read the target files directly out of the chosen sibling before writing anything - this skill
describes what to create and why, not literal bytes to copy, since the sibling's own files may
have moved on since this was written.

### 3. What to add

1. **CI workflow(s)** (`.github/workflows/`) - matching the detected stack's build/lint/test
   steps from the chosen sibling. Adapt package/module names; don't copy them verbatim.

   Before wiring any caller workflow (`uses: <org>/github-actions/.github/workflows/<x>.yaml@master`),
   read that reusable workflow's own `permissions:` and `secrets:` blocks - don't assume the
   caller's `permissions:` can just be "whatever looks reasonable" or that `secrets: inherit`
   covers everything:
   - **Permissions are a ceiling, not a suggestion.** A reusable workflow's top-level
     `permissions:` block states what it needs; if the caller's own `permissions:` block doesn't
     grant at least that much, the *entire run* fails at dispatch time with `startup_failure` and
     zero jobs created - not a runtime permission error, so there's nothing to debug in job logs.
     E.g. `rust-release.yaml` declares `id-token: write` (for crates.io trusted publishing) even
     when a caller sets `publish-to-crates-io: false` and never uses it - the caller still has to
     grant it. Confirmed the hard way on `main-web`'s first release.
   - **A release/merge-back step that needs to push past branch protection needs a GitHub App
     token, not the default `GITHUB_TOKEN`.** Any reusable workflow step that pushes a commit
     directly to a ruleset-protected branch (not through a PR) - e.g. merging a release ref back
     into `develop` - will get rejected (`GH013: Repository rule violations`) unless it merges via
     the GitHub API as the `SRPG_CI_APP_ID`/`SRPG_CI_PRIVATE_KEY` app (a configured ruleset bypass
     actor, see step 5) rather than `git push` with `secrets.GITHUB_TOKEN`. Check whether the
     reusable workflow's `secrets:` block lists these - if it does, `secrets: inherit` on the
     caller is enough (the org-level secrets already exist), but the *reusable workflow itself*
     needs to actually merge server-side (`gh api repos/OWNER/REPO/merges`), not shell out to
     `git merge && git push` - a workflow written before this pattern was established (as
     `rust-release.yaml` originally was) will fail on literally the first real release into a
     protected branch. Fix the reusable workflow, don't route around it in the caller.
2. **Dependabot** (`.github/dependabot.yaml`) - ecosystem(s) matching the detected stack, weekly,
   targeting `develop` per the platform's git-flow standard (see `docs/git-flow.md`). If the
   repo doesn't have a `develop` branch yet, create it as part of this scaffolding rather than
   targeting `master`.
3. **Community docs** - `CODE_OF_CONDUCT.md` (generic, copy from the sibling verbatim),
   `CONTRIBUTING.md`, `SECURITY.md`, `README.md`. Copy each from the sibling and adapt the
   repo-specific parts: name, GitHub URL, what the repo actually does, its dependency
   relationships within the platform. Seed `CHANGELOG.md` with just an `[Unreleased]` section
   noting the scaffolding work, if the repo uses a changelog.
4. **Robot guidance** - write `AGENTS.md` covering: About This Project, any submodule/dependency
   relationships, Committing Code (Conventional Commits), Branches and Workflow, and a Running
   Checks Locally section with the actual detected-stack commands. Then symlink `CLAUDE.md` to
   it - a real symlink, not a copy: `ln -s AGENTS.md CLAUDE.md`.

   Include a **Platform Conventions and Decisions** section pointing at the platform docs and
   ADR index (step 12 has the exact block). Accepted `PADR-*` records are binding constraints -
   the block says so, and says a task that needs to contradict one stops and proposes a
   superseding ADR rather than working around it.
5. **Branch protection** - check what the chosen sibling actually has configured before assuming
   a shape:

   ```bash
   gh api repos/<org>/<sibling>/rulesets
   gh api repos/<org>/<sibling>/branches/<default-branch>/protection
   ```

   Whichever mechanism the sibling uses (rulesets vs. classic protection), replicate that same
   mechanism on the target repo rather than picking whichever this skill happens to default to.
   Required status check names must match the target repo's own CI job names (from step 3.1),
   not copied verbatim from the sibling if the job IDs differ.

   Two fixes to make regardless of what the sibling's ruleset literally contains - the sibling
   itself may still carry these from before they were caught:
   - **`allowed_merge_methods` must never include `squash`.** Squash merges are disabled
     org-wide (`docs/git-flow.md`) - a squash-merged multi-commit PR reliably strips the
     Conventional Commits prefix `git-cliff` needs, and can make a release PR silently reuse an
     already-tagged version. Use `["merge", "rebase"]` only.
   - **Never add a `copilot_code_review` ruleset rule.** A rule requiring
     that review never completes review, silently blocking every PR forever (needs `--admin` to
     merge anything, permanently). If a sibling's ruleset has this rule, strip it before copying
     the pattern, and strip it from the sibling too while you're there.
6. **Repository setting: allow auto-merge** - enable it at the repo level:

   ```bash
   gh repo edit <org>/<repo> --enable-auto-merge
   ```

   This is a plain repository setting, not a ruleset rule, and is off by default on new repos.
   Without it, `gh pr merge --auto` fails with `GraphQL: Auto merge is not allowed for this
   repository` even when the PR is otherwise mergeable. Verify with
   `gh api repos/<org>/<repo> --jq '.allow_auto_merge'`.
6b. **Repository setting: disable "Automatically delete head branches"** - explicitly, even
   though this skill never turns it on:

   ```bash
   gh repo edit <org>/<repo> --delete-branch-on-merge=false
   ```

   Found live (`true`) on several repos never scaffolded through this exact sequence - GitHub
   defaults it to `true` for a repo created through some paths, and a `master`+`develop` git-flow
   makes that dangerous in a way a typical single-default-branch repo isn't: any PR whose *head*
   ref is `master` or `develop` itself (a manual merge-forward PR, a botched hotfix, anything that
   isn't the release automation's own server-side API merge) gets that branch deleted the moment
   the PR merges. Verify with `gh api repos/<org>/<repo> --jq '.delete_branch_on_merge'` and
   disable unconditionally - never assume it's already off because this skill doesn't set it.
7. **ArgoCD webhook** - if this repo has (or will have) a `kubernetes/overlays/` ArgoCD deploys
   from, add a GitHub webhook so pushes sync instantly instead of waiting out ArgoCD's default
   poll cycle:

   ```bash
   gh api repos/<org>/<repo>/hooks -X POST \
     -f name=web \
     -f "config[url]=https://argocd.example.com/api/webhook" \
     -f "config[content_type]=json" \
     -F active=true \
     -f "events[]=push"
   ```

8. **Debug workflow** (`.github/workflows/debug.yml`) - every repo carries one of these;
   never delete it while scaffolding or rewriting CI, even if it looks stale or references
   something being retired (e.g. an old private registry). Fix the stale parts in place instead:
   find a sibling's current `debug.yml`, keep its context-dump steps (GitHub/job/steps/runner/
   strategy/matrix - these never go stale, they just echo whatever `toJson()` returns) verbatim,
   and only rewrite the parts that reference this repo specifically (the `docker/metadata-action`
   step's `images:` value, e.g. `ghcr.io/<org>/<repo>` instead of a stale
   `registry.example.com/<repo>`).
9. **Code coverage** - every repo's CI (and PR) workflow gets a coverage step, a warn-only
   threshold check, and a GitHub Pages-hosted badge, using the language-appropriate tool. This
   was rolled out platform-wide via `openspec/changes/add-code-coverage`; treat that change's
   `tasks.md` as the detailed precedent log and this as the condensed convention:

   - **Tool per language**: Go - `go test -coverprofile` + `go tool cover -func`. Rust -
     `cargo tarpaulin --engine llvm --out Html --out Xml` (the `llvm` engine avoids `ptrace`
     flakiness on GitHub-hosted runners; plain `--engine ptrace` is not worth the instability).
     Python - `pytest-cov` (reuse an already-provisioned `tox` venv via `tox exec -e <env> --
     coverage ...` rather than installing coverage separately, if the repo uses tox). Swift -
     `swift test --enable-code-coverage` + `llvm-cov export -format=lcov` / `llvm-cov show
     -format=html` (not `genhtml`/`lcov` - neither is preinstalled on the GitHub-hosted runner
     image).
   - **Step summary**: write a `## <Language> coverage` section to `$GITHUB_STEP_SUMMARY`
     wrapped in a fenced code block, from whatever plain-text report the tool produces.
   - **Artifact**: `actions/upload-artifact@v4` uploading the HTML+raw report directory.
   - **Threshold check**: a separate step, `continue-on-error: true` (warn-only - never blocks a
     merge), placed *after* the summary/artifact steps so a below-threshold result never
     prevents those from being produced.
   - **Badge**: write a `coverage-badge.json` (shields.io endpoint schema:
     `{"schemaVersion":1,"label":"coverage","message":"<pct>%","color":"<color>"}`, color
     `brightgreen`/`>=80`, `yellow`/`>=50`, `red` otherwise) into the coverage output directory,
     then publish that directory to `gh-pages` via `peaceiris/actions-gh-pages@v4`. README badge
      markdown: `[![Coverage](https://img.shields.io/endpoint?url=https://<org>.github.io/<repo>/<path>/coverage-badge.json)](https://<org>.github.io/<repo>/<path>/)`.
     Two badge-hosting designs were tried and abandoned before landing on GitHub Pages: a direct
     push to the source branch (rejected by PR-only branch-protection rulesets) and a bot-opened
     PR with auto-merge (GITHUB_TOKEN-authored pushes/PRs never trigger further Actions runs, so
     the required check on that PR never appeared).

   **Gotchas specific to this step** (all found the hard way during the platform rollout - verify
   each, don't assume they're already handled):
   - **GitHub Pages must be explicitly enabled**: a `gh-pages` branch with the right content
     existing is *not* the same as Pages serving it. `gh api repos/<org>/<repo>/pages --jq
     '.status'` first; if 404, `gh api -X POST repos/<org>/<repo>/pages -f
     "source[branch]=gh-pages" -f "source[path]=/"`.
   - **If another job already publishes to `gh-pages` root** (e.g. a `docs` job publishing
     rustdoc/sphinx), the coverage publish step needs `destination_dir: coverage` (or similar)
     to avoid colliding, *and* the other job's own publish step needs `keep_files: true` -
     without it, a root-level `peaceiris` publish with no `destination_dir` wipes the *whole*
     branch by default, deleting whatever the coverage step just published moments earlier if
     the jobs run in the same workflow (`needs: [tests]`).
   - **Push-trigger path filters must include `.github/workflows/**`** - otherwise a
     coverage-only or CI-fix-only commit merged to the default branch triggers *zero* CI runs
     there, so the change ships completely unverified. This was the single most common bug
     found in this rollout - always check `gh run list --branch <default-branch>` actually shows
     a run for the merge commit, not just that the PR's own checks passed.
   - **A `VAR=$(pipeline)` assignment aborts the whole step under `set -e` if the pipeline
     produces no output** (e.g. a crate/package with 0 coverable lines, where the coverage tool
     reports `NaN` and every downstream `grep` fails to match) - even though the equivalent bare
     pipe with an always-succeeding trailing command (`| tail -1`) would not. Append `|| true` to
     the extraction pipeline and default the captured value (`VAR=${VAR:-0}`).
   - **A CI run reporting "success" is not proof the badge is live** - `continue-on-error` steps,
     silently-skipped downstream jobs, and Pages not being enabled can all leave a badge 404 even
     when every visible check passed. Always `curl` the actual published `coverage-badge.json`
     URL before calling a repo done.
10. **Multi-arch Docker builds** - every repo's `docker-build.yml` builds and pushes both
    `linux/amd64` and `linux/arm64`, not just the runner's native `amd64`:

    ```yaml
    - name: Set up QEMU
      uses: docker/setup-qemu-action@v4
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v4
    - name: Build and push
      uses: docker/build-push-action@v7
      with:
        push: true
        platforms: linux/amd64,linux/arm64
    ```

    `admin-api` is the reference implementation. The QEMU/Buildx setup steps alone are not
    sufficient and are easy to mistake for "done" - `docker/build-push-action` still builds
    `amd64`-only unless `platforms:` is explicitly passed; `main-web` and `assets-web` both
    shipped this way (QEMU/Buildx present, `platforms:` missing) until caught and fixed. Verify
    with a manual `workflow_dispatch` run and check the pushed manifest is multi-arch (`docker
    buildx imagetools inspect ghcr.io/<org>/<repo>:latest` lists both platforms), not just
    that the workflow went green - a green run proves the build succeeded for whatever platforms
    were actually requested, not that both were.

11. **Release workflow family** - every repo gets `prepare-release.yaml`/`release.yaml`/
    `tag-release.yaml`, each a thin caller into a shared `github-actions` repo's reusable
    `<lang>-prepare-release.yaml`/`<lang>-release.yaml`/`<lang>-tag-release.yaml` workflows
    (`go`/`rust`/`swift`/`python` families exist as of this writing - check
    `gh api repos/<org>/github-actions/contents/.github/workflows --jq '.[].name'` for the
    current set before assuming a language isn't covered). Don't hand-roll release automation
    per repo - `assets-web` is the reference consumer for the Python family; check its own
    `.github/workflows/{prepare-release,release,tag-release}.yaml` for the exact caller shape
    before copying, since reusable-workflow inputs drift.

    ```yaml
    # prepare-release.yaml
    on:
      workflow_dispatch:
    permissions:
      contents: write
      pull-requests: write
    jobs:
      prepare:
        uses: <org>/github-actions/.github/workflows/<lang>-prepare-release.yaml@master
        with:
          version-file: <path to the file holding the version string, e.g. src/<pkg>/__init__.py>
        secrets: inherit
    ```

    - `prepare-release.yaml` computes the next version from Conventional Commits (`git-cliff`),
      bumps the version file, updates `CHANGELOG.md`, and opens a `release/<version>` PR into
      `master` - dispatched manually, not on a schedule.
    - `release.yaml` triggers on `v*` tag push (or manual dispatch with an existing tag) and
      builds/tests/publishes a GitHub Release, then merges `master` back into `develop`. If the
      repo's dependency manager isn't the reusable workflow's default (e.g. Python's default
      assumes `pip install -r requirements/tests.txt -e .`, not uv), override `install-command`/
      `test-command` explicitly - see `assets-web`'s `release.yaml` for the uv override.
    - `tag-release.yaml` triggers on a `release/*` PR closing (merged) into `master` and tags the
      release, which the `release.yaml` tag-push trigger then picks up.
    - Both `prepare-release.yaml` and `tag-release.yaml` need the `SRPG_CI_APP_ID`/
      `SRPG_CI_PRIVATE_KEY` secrets (see step 3.1's permissions/secrets note) - `secrets: inherit`
      is enough since these are already org-level secrets, but confirm the reusable workflow
      itself uses the GitHub API for the protected-branch merge/push, not a bare `git push`.
    - Superseding an older ad hoc release mechanism (e.g. a `relekang/python-semantic-release`
      step auto-tagging every push to `develop` with no review or changelog) is in scope for this
      step - remove it once the reusable-workflow family replaces it, don't run both.

12. **Architecture Decision Records** - every repo carries a `docs/adr/` directory and the `/adr`
    command, so a service-local decision (`ADR-NNNN`) has somewhere to land and platform-wide
    ones (`PADR-NNNN`) are discoverable. Source everything from the platform repo:

    - `docs/adr/0000-template.md` - copy verbatim from `platform/docs/adr/0000-template.md`.
    - `docs/adr/README.md` - a short index. Copy `platform/docs/adr/README.md` and trim it to
      this repo: keep the "what an ADR is / rules / statuses / frontmatter / writing one"
      sections, replace the index table with just the template row, and state that this repo's
      records are the service-local `ADR-NNNN` tier while `PADR-NNNN` lives in `platform`.
    - `.claude/commands/adr.md` - copy verbatim from `platform/.claude/commands/adr.md`. It
      auto-detects the service-local tier when run outside the platform repo.
    - `AGENTS.md` gets this block. Adjust the relative prefix on the first bullet to the repo's
      actual depth inside `platform`: `../../` for a repo under `services/`, `frontends/`,
      `foundational/`, `support/`, `templates/`, or `tooling/`; `../` for `design/` or
      `kubernetes/`. The absolute GitHub URL on the second bullet is the fallback for a repo
      cloned on its own or consumed only as a Go/other module, where the relative path does not
      resolve:

      ```markdown
      ## Platform Conventions and Decisions

      This repo is a submodule of the platform repo. Platform-wide conventions and Architecture
      Decision Records live there:

      - checked out inside the platform tree: `../../docs/README.md` (convention index) and
        `../../docs/adr/README.md` (platform ADRs, `PADR-*`)
      - standalone or module-only checkout:
        <https://github.com/<org>/platform/tree/master/docs> and
        <https://github.com/<org>/platform/tree/master/docs/adr>

      Accepted `PADR-*` records are binding constraints. Read the ADR index before proposing a
      structural change; if a task needs to contradict an accepted record, stop and say so -
      propose a superseding ADR rather than working around it. This repo's own service-local
      decisions are `ADR-*` in `docs/adr/` here. `/adr "<title>"` scaffolds one.
      ```

    A repo that already has `docs/adr/` (it adopted ADRs early) just needs the `AGENTS.md` block
    and the `/adr` command checked for currency against `platform`'s copies.

### 4. Create a `release` label

Every repo should have a `release` label, colored green, for tagging release PRs/issues:

```bash
gh label create release --repo <org>/<repo> --color 00FF00 --description "Release" --force
```

`--force` makes this idempotent if the label already exists under a different color/description.

### 5. Look up the repo's real GitHub Projects v2 board

```bash
gh project list --owner <org> --format json
```

Confirm which project the repo actually belongs to (ask the user if more than one is plausible)
before wiring any project-number references into CI or docs.

## Gotchas

- **Branch protection checks that "never ran"**: GitHub lets you register a required status
  check context before it has ever reported - expected on a freshly scaffolded repo, not an
  error. The check starts enforcing on the first PR that runs CI.
- **Empty-diff PRs get rejected**: if you're also opening a PR right after scaffolding and the
  branch has no other commits, `gh pr create` fails with "No commits between X and Y". An empty
  commit (`git commit --allow-empty`) unblocks it if a PR is genuinely needed before real work
  lands; don't reach for this if you can just wait for the first real commit.
- **Don't hardcode project numbers, package names, or CI job names** by copying them from
  another repo's workflow file - these are exactly the values that must be repo-specific.
- **`git add`**: stage the new files explicitly by path, not `git add -A`/`.` - an explicit list
  is one line of extra typing and never accidentally sweeps in something unrelated sitting in
  the worktree.
- **Auto-merge is a separate on/off repo setting, not part of any ruleset.** A repo can have
  fully correct rulesets and still reject `gh pr merge --auto` until `allow_auto_merge` is
  turned on (step 3.6). Don't assume "branch protection is set up" implies auto-merge works.
- **Rulesets vs. classic branch protection - check a live example before guessing the shape.**
  Don't assume either is "the standard" without querying an actual sibling repo first
  (step 3.5) - a repo you check might have neither configured, which is not the same as the org
  having no standard elsewhere.
- **If GitHub API calls fail with a TLS/certificate error**, that's a sandboxed-network
  restriction, not a real GitHub outage - retry the `gh`/`gh api` call outside the sandbox.
- **Never delete `debug.yml` while cleaning up stale CI** - a workflow rewrite (e.g. Python to
  Rust) tends to sweep up every `.github/workflows/*` file that references the old stack, and
  `debug.yml` looks like one of them (it usually has an old-registry Docker meta step too) but
  its context-dump steps are stack-agnostic and worth keeping. Fix the stale reference (step 8),
  don't remove the file.
- **No ArgoCD webhook means every deploy sits for up to ~3 minutes** (`timeout.reconciliation`
  120s + up to 60s jitter in `argocd-cm`) after a push or a Docker build's automated image-tag
  bump before ArgoCD's Application object actually updates - confirmed the hard way across three
  sequential hotfix releases in one incident before the webhook (step 3.7) was added. Don't
  mistake this lag for the deploy automation itself failing; check whether the git commit landed
  before assuming the pipeline is broken.
