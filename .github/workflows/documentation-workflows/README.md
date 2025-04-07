# Workflows

## 1. `pages-pr-preview.yml` – Documentation PR Preview
Reusable GitHub Action to deploy a documentation preview when a pull request is opened.
### Usage
```yaml
jobs:
  preview-docs:
    uses: secure-software-engineering/actions/.github/workflows/documentation-workflows/pages-pr-preview.yml@develop
    with:
      enable_comment: true  # Optional: posts link in PR comment
      title_prefix: "PR Preview: "  # Optional: custom prefix for deployment title
```
## 2. `pages-branch-snapshot.yml` — Deploy a Snapshot for Branch
Reusable GitHub Action to deploy versioned MkDocs documentation from a branch.
### Usage
```yaml
jobs:
  deploy-snapshot:
    uses: secure-software-engineering/actions/.github/workflows/documentation-workflows/pages-branch-snapshot.yml@develop
    with:
      latest_branch: develop
      version: "maven" # Or any custom version
```