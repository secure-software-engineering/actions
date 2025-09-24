# Actions for Dependabot Automation

### Automatically Approve and Merge PRs from Dependabot (Why)

This action automatically approves and merges pull requests created by Dependabot.
This is reduces maintainer interaction as it automates approving and merging dependency updates

### How/What

This action triggers when Dependabot creates or updates a pull request and automatically approves it.
The approval helps streamline the dependency update process while still allowing for manual review if needed.

### Concerns/Risks
Increases attack surface by possibly introducing compromised code from dependencies without further checks.
But If dependencies are updated "blindly" anyways this workflow is in no way different.

### Configuration
- enable `status checks` in `Settings / Branch protection rules` so only valid code is merged
- enable `Allow auto merge` in `Settings / General` to make automatic merging possible
- store a Personal Access Token in `dependabot secrets` e.g. as `AUTO_MERGE_PAT` to give write permissions
    - [Fine Grained Token](https://github.com/settings/personal-access-tokens/): Permissions
        - `Pull Requests` read & write
        - `Contents` read & write
- enable `Automatically delete head branches Loading` in `Settings / General` to cleanup old branches after merge

### Calling Workflow:

`.github/workflow/dependabot.yml`

```yaml
name: Handle Dependabot PRs

on:
  pull_request:
    types: [ opened, reopened, synchronize ]

permissions:
  contents: write
  pull-requests: write

jobs:
  ApproveAndMerge:
    name: Auto approve Dependabot PRs
    runs-on: ubuntu-latest
    # Only run for PRs created by Dependabot - extended verification is done in the reusable workflow
    if: github.actor == 'dependabot[bot]'
    # These permissions are needed to approve pull requests
    permissions:
      contents: read
      pull-requests: write
    steps:
      - name: Auto approve Dependabot PR
        uses: secure-software-engineering/actions/dependabot@develop
        with:
          token: ${{ secrets.AUTO_MERGE_PAT }}
```

### Options:

- **token**: The GitHub token for authentication with pull request approval permissions (required)
- **should-approve**: default: enabled - set false to disable
- **should-merge**: default: enabled - set false to disable