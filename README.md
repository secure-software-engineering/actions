# actions
Secure Software Engineering groups GitHub Actions - centralized &amp; reusable

## 1. `pages/pr-preview-action.yml` – Documentation PR Preview
Reusable GitHub Action to deploy a documentation preview when a pull request is opened.
### Usage
```yaml
jobs:
  preview-docs:
    runs-on: ubuntu-latest
    steps:
      - uses: secure-software-engineering/actions/pages/pr-preview-action.yml@develop
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          event_action: ${{ github.event.action }}
          head_ref: ${{ github.head_ref }}
          repository_owner: ${{ github.repository_owner }}
          repository_name: ${{ github.event.repository.name }}
          enable_comment: true  # Optional: posts link in PR comment
          title_prefix: "PR Preview: "  # Optional: custom prefix for deployment title
```
## 2. `pages/branch-snapshot-action.yml` — Deploy a Snapshot for Branch
Reusable GitHub Action to deploy versioned MkDocs documentation from a branch.
### Usage
```yaml
jobs:
  deploy-snapshot:
    runs-on: ubuntu-latest
    steps:
      - uses: secure-software-engineering/actions/pages/branch-snapshot-action.yml@develop
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          latest_branch: develop
          version: "maven" # Or any custom version
```
## 3. `dependabot/auto-approve-action.yml` - Auto Approve Dependabot PRs

This reusable GitHub Action approves PRs created by Dependabot.

### Usage

```yaml
jobs:
  auto-approve:
    runs-on: ubuntu-latest
    steps:
      - uses: secure-software-engineering/actions/dependabot/auto-approve-action.yml@develop
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
```

## 4. `javadoc/publish-action.yml` - Publish Javadoc to GitHub Pages

Reusable GitHub Action to generate and publish Javadoc to GitHub Pages.

### Usage
```yaml
jobs:
  publish-javadoc:
    runs-on: ubuntu-latest
    steps:
      - uses: secure-software-engineering/actions/javadoc/publish-action.yml@develop
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          java_version: '17'  # Optional, default: '17'
          java_distribution: 'temurin'  # Optional, default: 'temurin'
          latest_branch: 'develop'  # Optional, default: 'develop'
          title: ''  # Required: title for documentation

```

## 5. `version/version.yml` - Version Handling

Handles version updates and releases for merged pull requests (Maven-based projects).

### Usage
```yaml
name: Version Handling

concurrency:
  group: version-handling
  cancel-in-progress: false

on:
  pull_request:
    types: [closed]
    branches: [master]

jobs:
  handle-version:
    if: ${{ github.event.pull_request.merged == true }}
    runs-on: ubuntu-latest
    steps:
      - uses: secure-software-engineering/actions/version/version.yml@develop
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          java_version: '17'  # Optional, default: '17'
          java_distribution: 'adopt'  # Optional, default: 'adopt'
          target_branch: 'master'  # Optional, default: 'master'
          version: ''  # Optional: specify version, otherwise uses timestamp
```