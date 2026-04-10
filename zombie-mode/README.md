# Zombie Mode

Two composite actions for automated release management of low-maintenance Java/Maven repositories.

- **prepare-pr** — Compares e.g. `develop` vs `main`. If unreleased commits exist, opens a versioned release PR. If a zombie-mode PR is already open it adds a reminder comment instead.
- **merge-pr** — Merges open zombie-mode PRs older than a configurable age. Omit this action if you prefer manual merges.

Possible releases are published by your existing deployment workflow (e.g. `deployment/maven-deployment`) when the PR merges into `main`. Zombie-mode only manages the PR lifecycle.

---

## Release Pattern

Configure two separate workflow files: one that creates the PR runs e.g. 14 days before the release date and one that auto-merges on release day.

### Workflow 1 — Create release PR

Fires on the 17th of odd months (15 days before the 1st of each even month).

> **Note on cron math**: January has 31 days, so Jan 17 → Feb 1 = 15 days. September has 30 days, so Sep 17 → Oct 1 = 14 days. Accept ±1 day variance, or adjust the day per month.

```yaml
# .github/workflows/zombie_prepare_release.yml
name: Zombie Prepare Release

on:
  schedule:
    # 17th of odd months = ~15 days before 1st of even months (Feb, Apr, Jun, Aug, Oct, Dec)
    - cron: '0 0 17 1,3,5,7,9,11 *'
  workflow_dispatch:

jobs:
  prepare-release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: secure-software-engineering/actions/zombie-mode/prepare-pr@develop
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          base-branch: develop       # branch with unreleased changes
          target-branch: main        # branch the PR merges into
          version-bump: patch        # or wire a compat-check action (see below)
          lead-days: '15'
```

### Workflow 2 — Auto-merge on release day

Fires on the 1st of each even month. Uses `auto-merge-days: '0'` to merge any open zombie PR immediately.

```yaml
# .github/workflows/zombie_release.yml
name: Zombie Release

on:
  schedule:
    # 1st of even months at 09:00 UTC
    - cron: '0 9 1 2,4,6,8,10,12 *'
  workflow_dispatch:

jobs:
  auto-merge:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: secure-software-engineering/actions/zombie-mode/merge-pr@develop
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          age-days: '0'           # 0 = merge all open zombie-mode PRs immediately
          target-branch: main     # must match prepare-pr's target-branch
          merge-method: squash
          delete-branch: 'true'
```

When the merge completes, any workflow in your repo that triggers on `push` to `main` (e.g. your Maven deployment workflow) will fire automatically.

---

## Wiring an External Compatibility Check

Because GitHub Actions composite actions cannot call other actions dynamically, pipe the compat-check result through a `needs:` job:

```yaml
jobs:
  compat-check:
    runs-on: ubuntu-latest
    outputs:
      version-bump: ${{ steps.compat.outputs.version-bump }}
    steps:
      - uses: your-org/compat-check-action@v1
        id: compat
        # Expected output: version-bump = patch | minor | major

  prepare-release:
    needs: compat-check
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: secure-software-engineering/actions/zombie-mode/prepare-pr@develop
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          version-bump: ${{ needs.compat-check.outputs.version-bump || 'patch' }}
```

If the compat-check action is not configured, `version-bump` defaults to `patch`.

---

## Prerequisites

The target repository must have these labels (create once with `gh label create`):

```bash
gh label create zombie-mode    --color 0075ca --description "Automated zombie-mode release"
gh label create dependencies   --color 0075ca --description "Dependency updates"
gh label create auto-merge     --color e4e669 --description "Eligible for auto-merge"
```

---

## Input Reference

### prepare-pr

| Input | Default | Description |
|---|---|---|
| `token` | required | GitHub token |
| `base-branch` | `develop` | Branch with unreleased changes (PR source) |
| `target-branch` | `main` | Branch the release PR merges into |
| `version-bump` | `patch` | Semver component to increment: `patch` / `minor` / `major` |
| `auto-merge-days` | `` | Days shown in PR body as auto-merge deadline (informational, empty = disabled) |
| `lead-days` | `14` | Shown in PR body (informational only) |
| `version-file` | `pom.xml` | Version file to update (Maven only) |
| `java-version` | `17` | JDK version for Maven |
| `java-distribution` | `temurin` | JDK distribution for Maven |

**Outputs**: `release-created` · `new-version` · `pr-number` · `skipped-reason`

### merge-pr

| Input | Default | Description |
|---|---|---|
| `token` | required | GitHub token |
| `age-days` | required | Merge PRs older than N days; `0` merges all open PRs immediately |
| `target-branch` | `main` | Filter PRs by this target branch (must match prepare-pr's `target-branch`) |
| `merge-method` | `squash` | `merge` / `squash` / `rebase` |
| `delete-branch` | `true` | Delete release branch after merge |
| `label` | `zombie-mode` | Label identifying zombie-mode PRs |

**Outputs**: `merged-count` · `merged-prs`
