# Actions for Zombie Mode Automation

## Zombie Auto-merge

The **zombie-mode** action automatically merges aged zombie release PRs after a specified timeout period. It identifies PRs with the zombie-mode label that are older than the configured age limit and automatically merges them with a comment explaining the auto-merge action.

A possible action **zombie_auto_merge.yml** can look like this:

```yaml
name: Zombie Auto-merge

on:
  schedule:
    # Run daily at 9 AM UTC to check for aged zombie PRs
    - cron: '0 9 * * *'
  workflow_dispatch:
    # Allow manual triggering

jobs:
  auto-merge-zombie-prs:
    name: Auto-merge aged zombie PRs
    runs-on: ubuntu-latest
    
    # These permissions are needed to merge PRs and manage branches
    permissions:
      contents: write
      pull-requests: write

    steps:
      - name: Auto-merge Zombie PRs
        uses: secure-software-engineering/actions/zombie-mode/auto-merge-zombie-prs@develop
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          age-days: '3'
          merge-method: 'squash'
          delete-branch: 'true'
          zombie-label: 'zombie-mode'
```

This action runs on a schedule (daily) or can be triggered manually. It identifies zombie release PRs that are older than the specified age limit and automatically merges them after adding a comment explaining the auto-merge action.

**Possible options:**
- **token**: The GitHub token for authentication (required)
- **age-days**: Number of days old a PR must be to auto-merge (optional, default: '3')
- **merge-method**: Merge method to use - 'merge', 'squash', or 'rebase' (optional, default: 'squash')
- **delete-branch**: Whether to delete the branch after merging (optional, default: 'true')
- **zombie-label**: Label to identify zombie mode PRs (optional, default: 'zombie-mode')


## Zombie Release

The **zombie-mode** action automatically creates release PRs when dependency updates accumulate. It detects dependency updates since the last release, calculates the next version, and creates a release PR with the zombie-mode label for automatic processing.

A possible action **zombie_release.yml** can look like this:

```yaml
name: Zombie Release

on:
  schedule:
    # Run weekly on Monday at 8:00 AM UTC to check for accumulated dependencies
    - cron: '0 8 * * 1'
  workflow_dispatch:
    # Allow manual triggering

jobs:
  create-zombie-release:
    name: Create zombie release PR
    runs-on: ubuntu-latest
    
    # These permissions are needed to create branches, PRs, and access git history
    permissions:
      contents: write
      pull-requests: write

    steps:
      - name: Create Zombie Release
        uses: secure-software-engineering/actions/zombie-mode@develop
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          base-branch: main
          auto-merge-days: '3'
          version-file: pom.xml
```
This action runs on a schedule or can be triggered manually. It checks for dependency updates since the last release and automatically creates a release PR when updates are found.

**Possible options**:
- **token**: The GitHub token for authentication (required)
- **base-branch**: Base branch to create release PR against (optional, default: 'main')
- **auto-merge-days**: Number of days before auto-merge for PR description (optional, default: '3')
- **version-file**: Version file to update - pom.xml, package.json, etc. (optional, default: 'pom.xml')