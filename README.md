# GitHub Actions - Secure Software Engineering (Reusable Workflows)

> **DEPRECATED BRANCH**
>
> Reusable workflow implementations for legacy compatibility. **Composite actions are preferred** - use develop branch for new projects.

## Available Workflows

### Dependency Management

#### Auto-Approve Dependabot PRs
*Automatically approves pull requests created by Dependabot*
```yaml
jobs:
  auto-approve:
    uses: secure-software-engineering/actions/.github/workflows/dependabot.yml@reusable-workflows
    with:
      token: ${{ secrets.GITHUB_TOKEN }}
```

#### Auto-Merge Zombie PRs
*Automatically merges aged zombie release PRs after specified timeout period*
```yaml
jobs:
  auto-merge-aged:
    uses: secure-software-engineering/actions/.github/workflows/auto-merge-zombie-prs.yml@reusable-workflows
    with:
      token: ${{ secrets.GITHUB_TOKEN }}
      age-days: '3'
      merge-method: 'squash'
      delete-branch: 'true'
      label: 'zombie-mode'
```

#### Zombie Release Creation
*Automatically creates release PRs when dependency updates accumulate*
```yaml
jobs:
  create-release:
    uses: secure-software-engineering/actions/.github/workflows/zombie-mode.yml@reusable-workflows
    with:
      token: ${{ secrets.GITHUB_TOKEN }}
      base-branch: 'main'
      auto-merge-days: '3'
      version-file: 'pom.xml'
```

### Documentation

#### PR Documentation Preview
*Creates and deploys a preview of the documentation for pull requests*
```yaml
jobs:
  preview-docs:
    uses: secure-software-engineering/actions/.github/workflows/documentation-handle-pr-preview.yml@reusable-workflows
    with:
      token: ${{ secrets.GITHUB_TOKEN }}
      preview-name: "pr-${{ github.event.number }}"
      preview-title: "PR Preview"
      gh-pages-branch: 'gh-pages'
      enable-comment: 'true'
```

#### Deploy Documentation
*Deploys documentation snapshot to GitHub Pages using mike*
```yaml
jobs:
  deploy-docs:
    uses: secure-software-engineering/actions/.github/workflows/documentation-pin-version.yml@reusable-workflows
    with:
      token: ${{ secrets.GITHUB_TOKEN }}
      latest_branch: 'develop'
      version: 'maven'
```

#### Publish Javadoc
*Generates and publishes Javadoc to GitHub Pages*
```yaml
jobs:
  publish-javadoc:
    uses: secure-software-engineering/actions/.github/workflows/javadoc.yml@reusable-workflows
    with:
      token: ${{ secrets.GITHUB_TOKEN }}
      path: ${{ github.ref_name }}
      java_version: '17'
      java_distribution: 'temurin'
      latest_branch: 'develop'
```

### Version Management

#### Version Handling
*Handles version updates and releases for merged pull requests (Maven-based projects)*
```yaml
jobs:
  handle-version:
    uses: secure-software-engineering/actions/.github/workflows/version.yml@reusable-workflows
    with:
      token: ${{ secrets.GITHUB_TOKEN }}
      java_version: '17'
      java_distribution: 'temurin' 
      target_branch: 'main'
      version: ''
```

## Key Differences from Composite Actions

- Direct GitHub context access (`github.ref_name`, `github.repository_owner`, etc.)
- Cross-repository usage with `uses: owner/repo/.github/workflows/file.yml@ref` syntax

---

*Use develop branch with composite actions for better performance and simpler setup.*