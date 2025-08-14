# GitHub Actions - Secure Software Engineering

Centralized & reusable GitHub Actions to simplify maintenance and maximize usability across all our repositories.
This collection provides our guidelines with standardized workflows for dependency management, documentation, and secure repository configuration to improve project maintenance.

## Quick Setup

### Repository Configuration

Branch Protection: Protect main/develop branches with required PR reviews
Dependabot: Add .github/dependabot.yml for weekly dependency updates
GitHub Pages: Enable for documentation deployment


## Basic Dependabot Config
```yaml
version: 2
updates:
   - package-ecosystem: "maven"
     directory: "/"
     schedule:
        interval: "weekly"
        day: "monday"
        time: "06:00"   # so that it is running before the workday starts
```

##  Available Actions

### Dependency Management
dependabot/ - Auto-approve Dependabot PRs
zombie-mode/ - Fully automated dependency management for unmaintained repositories

#### Documentation

- documentation/handle-pr-preview/ - Documentation Previews for PRs
- documentation/handle-deployment/ - Documentation Deployments
- javadoc/ - Javadoc publishing to GitHub Pages

#### Version Management
version/ - Automated version updates and releases for Maven projects

#### Usage
Reference actions using:
```yaml
uses: secure-software-engineering/actions/{action-path}@develop
```

## Zombie Mode
For repositories requiring zero-maintenance:

Auto-approves and merges Dependabot PRs
Creates weekly release PRs with accumulated changes
Auto-merges after 3-day review window

See individual action READMEs for detailed configuration.

*This actions repository is maintained by the Secure Software Engineering team. For questions, issues, or feature requests, please open an issue in this repository.*
