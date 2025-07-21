# GitHub Actions - Secure Software Engineering

Centralized & reusable GitHub Actions to simplify maintenance and maximize usability across all our repositories.
This collection provides our guidelines with standardized workflows for dependency management, documentation, and secure repository configuration to improve project maintenance.

## Repository Setup Guidelines

Follow this checklist to set up a secure, maintainable repository that can be easily transferred to new maintainers:

### Essential Security & Maintenance Configuration

1. **Branch Protection Rules**
   - Protect `main`/`master` and `develop` branches
   - Require pull requests with mandatory status checks
   - Require reviews from code owners
   - Enable merge queues for busy repositories
   - Disallow administrators from bypassing protections

2. **Dependency Management**
   - Configure Dependabot for automatic security updates
   - Set up weekly dependency scans
   - Use environment secrets for tokens and deployment credentials

3. **Documentation & Pages**
   - Enable GitHub Pages for automatic documentation deployment
   - Set up branch-based documentation snapshots
   - Configure Javadoc publishing for Java projects

4. **Access Control**
   - Use environment secrets (not repository secrets) for sensitive data
   - Add bot accounts to privileged users for automated workflows
   - Configure CODEOWNERS file for code review assignments

### Quick Dependabot Configuration

Add `.github/dependabot.yml` to your repository:

```yaml
version: 2
updates:
   - package-ecosystem: "maven"
     directory: "/"
     schedule:
        interval: "weekly"
        day: "monday"
        time: "05:00"   # so that it is running before the workday starts
     reviewers:         # optional
        - "your-team"
     assignees:         # optional
        - "your-team"
```

---

##  Available Actions

### Dependency Management

#### Standard Dependabot Auto-Approval
**Purpose**: Streamline dependency updates with manual oversight
**Use Case**: Active repositories with regular maintenance

```yaml
jobs:
  auto-approve:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
    steps:
      - uses: secure-software-engineering/actions/dependabot/auto-approve-action@develop
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
```

#### Zombie Mode - Automated Dependency Management
**Purpose**: Keep unmaintained repositories secure with zero-touch dependency updates
**Use Case**: Legacy repositories, archived projects, low-maintenance codebases

Zombie mode creates a complete automation pipeline:
1. **Auto-approve & merge** Dependabot PRs immediately
2. **Weekly scans** detect accumulated dependency updates
3. **Creates release PRs** aggregating all changes since last release
4. **3-day review window** allows manual intervention
5. **Auto-merge** triggers your existing release workflows

##### Setup Zombie Mode

Add these two scheduling workflows to orchestrate your existing actions:

```yaml
name: Zombie Release Management
on:
  schedule:
    - cron: '0 9 * * MON'  # Weekly on Monday at 9 AM
  workflow_dispatch:  # Allow manual trigger

jobs:
  create-release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: secure-software-engineering/actions/zombie-mode/zombie-mode@develop
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          base-branch: 'main'
          auto-merge-days: '3'
          version-file: 'pom.xml'  # For Maven projects

  auto-merge-aged:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
      contents: write
    steps:
      - uses: secure-software-engineering/actions/zombie-mode/auto-merge-zombie-prs@develop
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          age-days: '3'
          merge-method: 'squash'
          delete-branch: 'true'
```

For Dependabot automation, use your existing actions:
```yaml
   name: Dependabot Management
   on:
      pull_request:
      types: [opened]

   jobs:
      auto-approve:
         runs-on: ubuntu-latest
         permissions:
            pull-requests: write
         steps:
            - uses: secure-software-engineering/actions/dependabot/auto-approve-action@develop
              with:
               token: ${{ secrets.GITHUB_TOKEN }}
```

##### Manual Control Options

- **Immediate merge**: Manually merge the zombie release PR to trigger release immediately
- **Cancel release**: Close the zombie release PR to skip this release cycle
- **Review changes**: Check PR description for complete list of dependency updates
- **Emergency stop**: Remove `zombie-mode` label to exclude PR from auto-merge

##### Requirements for Zombie Mode

- Maven-based project with `pom.xml` (or specify custom `version-file`)
- Dependabot configured and enabled
- Semantic versioning format (`v1.2.3`)
- Repository write permissions for GitHub Actions

---

### Documentation & Pages

#### PR Documentation Preview
**Purpose**: Deploy documentation previews for pull requests to review changes before merge

```yaml
jobs:
  preview-docs:
    runs-on: ubuntu-latest
    permissions:
      pages: write
      pull-requests: write
    steps:
      - uses: secure-software-engineering/actions/pages/documentation/handle-pr-preview@develop
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          event_action: ${{ github.event.action }}
          head_ref: ${{ github.head_ref }}
          repository_owner: ${{ github.repository_owner }}
          repository_name: ${{ github.event.repository.name }}
          enable_comment: true  # Posts preview link in PR comments
          title_prefix: "PR Preview: "  # Custom deployment title prefix
```

**How it works**: When a PR is opened, creates a temporary documentation deployment accessible via unique URL. Automatically cleans up when PR is closed.

#### Branch Documentation Snapshots
**Purpose**: Deploy versioned documentation for different branches (develop, releases)

```yaml
# Trigger on push to main branches
on:
  push:
    branches: [main, develop, 'release/*']

jobs:
  deploy-snapshot:
    runs-on: ubuntu-latest
    permissions:
      pages: write
      contents: read
    steps:
      - uses: secure-software-engineering/actions/documentation/pin-version@develop
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          latest_branch: develop  # Branch treated as "latest" version
          version: "maven"  # Extract version from pom.xml, or specify manually
```

**Behavior**:
- **Latest branch** (e.g., `develop`): Deployed as `/latest/` and root `/`
- **Release branches**: Deployed as `/v1.2.3/` based on version
- **Other branches**: Deployed as `/branch-name/`

#### Javadoc Publishing
**Purpose**: Generate and publish Javadoc documentation to GitHub Pages

```yaml
jobs:
  publish-javadoc:
    runs-on: ubuntu-latest
    permissions:
      pages: write
      contents: read
    steps:
      - uses: secure-software-engineering/actions/javadoc/publish-action@develop
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          java_version: '17'  # Java version for building
          java_distribution: 'temurin'  # JDK distribution
          latest_branch: 'develop'  # Branch for latest docs
          path: ${{ github.ref_name }}
```

**Output**:
- **Latest branch**: Published as main Javadoc site
- **Release tags**: Archived as versioned API documentation
- Includes search functionality and cross-references

---

### Version & Release Management

#### Automated Version Handling
**Purpose**: Handle version updates and releases for merged pull requests (Maven projects)

```yaml
name: Version Handling

# Prevent concurrent version updates
concurrency:
  group: version-handling
  cancel-in-progress: false

on:
  pull_request:
    types: [closed]
    branches: [main]

jobs:
  handle-version:
    if: ${{ github.event.pull_request.merged == true }}
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: secure-software-engineering/actions/version/version@develop
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          java_version: '17'
          java_distribution: 'temurin'
          target_branch: 'main'  # Target branch for releases
          version: ''  # Leave empty to auto-generate timestamp version
```

**Process**:
1. **PR merged to main** → Triggers version workflow
2. **Version calculation** → Increments based on PR labels or uses timestamp
3. **Maven version update** → Updates `pom.xml` with new version
4. **Git tag creation** → Creates release tag
5. **Cleanup** → Prepares repository for next development cycle

**Finalization Steps**:
- Review generated release tag
- Manually trigger deployment workflows if needed
- Clean up any temporary branches

---

## Advanced Configuration

### Environment Secrets Setup

For repositories requiring additional permissions:

1. **Create bot account** or request access to shared service account
2. **Generate Personal Access Token** with required scopes:
   - `repo` - Full repository access
   - `write:packages` - Package publishing
   - `pages:write` - GitHub Pages deployment
3. **Add as environment secret** (not repository secret) named `BOT_TOKEN`
4. **Update workflows** to use `${{ secrets.BOT_TOKEN }}` instead of `GITHUB_TOKEN`

### Protected Branch Configuration

```bash
# Example branch protection API call
curl -X PUT \
  -H "Authorization: token $GITHUB_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/OWNER/REPO/branches/main/protection \
  -d '{
    "required_status_checks": {
      "strict": true,
      "contexts": ["ci/build", "ci/test"]
    },
    "enforce_admins": true,
    "required_pull_request_reviews": {
      "required_approving_review_count": 1,
      "dismiss_stale_reviews": true
    },
    "restrictions": null
  }'
```

### Troubleshooting

**Common Issues**:

1. **Permission denied errors**: Check if `GITHUB_TOKEN` has sufficient permissions, use bot token if needed
2. **Zombie mode not triggering**: Verify Dependabot is enabled and creating PRs
3. **Documentation not deploying**: Ensure GitHub Pages is enabled and source is set to "GitHub Actions"
4. **Version workflow failures**: Check Maven configuration and ensure `pom.xml` is in repository root

**Debug Steps**:
1. Check workflow run logs in Actions tab
2. Verify repository settings and permissions
3. Test with minimal configuration first
4. Review branch protection rules for conflicts

---

##  Migration Guide

### From Manual to Automated

1. **Start with standard actions** for active repositories
2. **Gradually enable zombie mode** for stable, low-maintenance projects
3. **Test in fork first** to verify compatibility with your project structure
4. **Monitor first few runs** to ensure expected behavior

### Repository Transfer Checklist

When transferring repository to new maintainer:

- [ ] Document custom configuration in repository README
- [ ] Transfer environment secrets and bot account access
- [ ] Update CODEOWNERS file with new maintainer
- [ ] Verify branch protection rules are properly configured
- [ ] Test all automated workflows with new maintainer permissions
- [ ] Provide access to this actions repository documentation

---

*This actions repository is maintained by the Secure Software Engineering team. For questions, issues, or feature requests, please open an issue in this repository.*
