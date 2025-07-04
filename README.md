# actions
Secure Software Engineering groups GitHub Actions - centralized &amp; reusabl to simplifying maintenance and maximize usability.


## How to configure a Repository
TODO

- use environment secrets e.g. for passwords, deployment tokens etc
- protect develop, master: use PRs, merge queues, setup mandatory checks, disallow admins to circumvent these protections 
- gh-pages for ducomentation
- dependabot to get latest dependency updates
- ...

## Auto Approve Dependabot PRs
This reusable GitHub Action approves PRs created by Dependabot.

### Goal 
Up to date dependencies with the latest security fixes. Reduce repetetive clicks from the maintainer.

### Usage
1. [configure dependabot](https://docs.github.com/en/code-security/getting-started/dependabot-quickstart-guide) to run at least once a week.
   ```
   TODO quick dependabot example
   ```
3. define the workflow e.g. with the example below
4. if your target branch is protected you need to define an Security Access Token - as you don't want your personal Token used in a shared project - add our bot account [tbc!] to your privileged users and request an access token from that account from the person that is currently responsible for the acocunt. Define the secret as an environment secret!

### Example Configuration
```yaml
jobs:
  auto-approve:
    runs-on: ubuntu-latest
    steps:
      - uses: secure-software-engineering/actions/dependabot/auto-approve-action.yml@develop
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
```


## GH-Pages Documentation with PR Preview
### Usage
 deploy a documentation preview when a pull request is opened.

### Example Configuration
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


## Deploy Documentation Snapshots for the develop and release Branch
Reusable GitHub Action to deploy versioned MkDocs documentation from a branch.

### Usage
TODO: describe behaviour for "latest_branch" and every other branch that it is called on.


### Example Configuration
```yaml
TODO: full example with different branches e.g. [master,develop] as trigger

jobs:
  deploy-snapshot:
    runs-on: ubuntu-latest
    steps:
      - uses: secure-software-engineering/actions/pages/branch-snapshot-action.yml@develop
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          latest_branch: develop # 
          version: "maven" # maven to get it from the pom - if a numeric version is given it will take the parameter
```

## Publish Javadoc to GitHub Pages
Reusable GitHub Action to generate and publish Javadoc to GitHub Pages.

### Usage
TODO: latest branch vs. snapshotting for releases

### Example Configuration
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

## Version Handling
Handles version updates and releases for merged pull requests (Maven-based projects).

### Usage
TODO: how to trigger, what to do to finalize it, cleanup, 

### Example Configuration
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
