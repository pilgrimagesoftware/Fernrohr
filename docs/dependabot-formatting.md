# Dependabot PR formatting

Dependabot does not run project formatters. Its configuration controls update behavior and PR metadata, such as grouping and commit-message prefixes, not arbitrary post-update commands.

Use a separate GitHub Actions workflow to format Dependabot branches and commit the result:

```yaml
name: Format Dependabot PR

on:
  pull_request:
    types: [opened, synchronize]

permissions:
  contents: write
  pull-requests: write

jobs:
  format:
    if: github.actor == 'dependabot[bot]'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.head_ref }}

      - run: npm ci
      - run: npm run format

      - uses: stefanzweifel/git-auto-commit-action@v5
        with:
          commit_message: "style: format Dependabot update"
```

Replace the formatter commands for the project. Restrict the workflow to Dependabot PRs and avoid executing untrusted PR code from `pull_request_target`.

If dependency updates plus arbitrary post-update commands are needed, Renovate supports this through `postUpgradeTasks`.
