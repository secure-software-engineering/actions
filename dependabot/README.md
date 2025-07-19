# Actions for Dependabot Automation

## Auto Approve Dependabot PRs

The **auto-approve-action** automatically approves pull requests created by Dependabot. This is useful for automatically approving dependency updates, especially for patch and minor version updates that are generally safe to merge.

A possible action **dependabot_auto_approve.yml** can look like this:

```yaml
name: Auto Approve Dependabot PRs

on:
  pull_request_target:
    types: [opened, reopened, synchronize]

jobs:
  auto-approve:
    name: Auto approve Dependabot PRs
    runs-on: ubuntu-latest
    # Only run for Dependabot PRs
    if: github.actor == 'dependabot[bot]'
    
    # These permissions are needed to approve pull requests
    permissions:
      contents: read
      pull-requests: write

    steps:
      - name: Auto approve Dependabot PR
        uses: secure-software-engineering/actions/dependabot@develop
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
```

This action triggers when Dependabot creates or updates a pull request and automatically approves it. The approval helps streamline the dependency update process while still allowing for manual review if needed.

**Possible options:**
- **token**: The GitHub token for authentication with pull request approval permissions (required)