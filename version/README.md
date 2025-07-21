# Actions for Version Management

## Version Handling

The **version** action handles version updates and releases for merged pull requests in Maven-based projects. It automatically creates version update branches, opens pull requests for version changes, and creates tags when version PRs are merged.

A possible action **version_handling.yml** can look like this:

```yaml
name: Version Handling

on:
  pull_request:
    types: [closed]
    branches:
      - master
      - main

jobs:
  handle-version:
    name: Handle version updates
    runs-on: ubuntu-latest
    if: github.event.pull_request.merged == true
    
    # These permissions are needed to create branches, PRs, and tags
    permissions:
      contents: write
      pull-requests: write

    steps:
      - name: Handle Version Update
        uses: secure-software-engineering/actions/version@develop
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          target_branch: main
```

This action triggers when pull requests are merged into the main branch. It automatically creates version update branches and pull requests, and handles tagging when version PRs are merged.

**Possible options:**
- **token**: The GitHub token for authentication (required)
- **java_version**: Java version to use for Maven operations (optional, default: '17')
- **java_distribution**: Java distribution to use (optional, default: 'temurin')
- **target_branch**: Target branch for version updates (optional, default: 'master')
- **version**: Specific version to use, if not provided uses timestamp-based versioning (optional, default: '')
