# Actions for Maven Release Management

Reusable **workflows** (not composite actions) that orchestrate the full release lifecycle for a
Maven-based project: open a release PR, keep it in sync with its title, and on merge tag +
deploy to Maven Central + advance the base branch to the next SNAPSHOT.

Because each of these needs multiple jobs, `needs:`-chained outputs and (for the deploy job) an
`environment:` protection rule, they are implemented as [reusable workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
under [`.github/workflows/`](../.github/workflows/) rather than as composite actions.

## Required repository secrets

The calling repository must define these and pass them through explicitly (or via `secrets: inherit`):

| Secret               | Used by                                | Purpose                                                            |
|-----------------------|-----------------------------------------|---------------------------------------------------------------------|
| a PAT (any name)      | prepare, title-sync, publish            | Push branches / open+merge PRs. Default `GITHUB_TOKEN` is usually blocked by branch protection or can't trigger downstream workflows. |
| GPG private key       | publish                                 | Sign deployed artifacts                                             |
| GPG passphrase        | publish                                 | Unlock the GPG private key                                          |
| Maven Central username| publish                                 | Publisher authentication                                            |
| Maven Central token   | publish                                 | Publisher authentication                                            |

## 1. `maven-release-prepare.yml`

Opens a release PR against `head_branch` (default `develop`) with the version bumped from the
latest `vX.Y.Z` tag according to `release_type`.

```yaml
name: Prepare Release
run-name: Release ${{ inputs.release_type }} triggered by ${{ github.actor }}

on:
  workflow_dispatch:
    inputs:
      release_type:
        description: 'Release type'
        required: true
        type: choice
        options: [major, minor, patch]

jobs:
  prepare-release:
    uses: secure-software-engineering/actions/.github/workflows/maven-release-prepare.yml@develop
    with:
      release_type: ${{ inputs.release_type }}
    secrets:
      release_pat: ${{ secrets.AUTO_MERGE_PAT }}
```

## 2. `maven-release-title-sync.yml`

Keeps the version on the release branch in sync whenever the PR title's `[major]`/`[minor]`/`[patch]`
tag is edited. Only acts on branches named `release/prep-*` opened by the prepare workflow above.

```yaml
name: Sync release version with PR title

on:
  pull_request:
    types: [edited, synchronize]

jobs:
  sync:
    uses: secure-software-engineering/actions/.github/workflows/maven-release-title-sync.yml@develop
    secrets:
      release_pat: ${{ secrets.AUTO_MERGE_PAT }}
```

## 3. `maven-release-publish.yml`

Triggered by the release PR closing (cleanup if abandoned) and by pushes to `head_branch` (tag,
deploy to Maven Central, open the auto-merging "next SNAPSHOT" PR).

```yaml
name: Publish Release

on:
  pull_request:
    branches: [develop]
    types: [closed]
  push:
    branches: [develop]

jobs:
  publish:
    uses: secure-software-engineering/actions/.github/workflows/maven-release-publish.yml@develop
    secrets:
      release_pat: ${{ secrets.AUTO_MERGE_PAT }}
      gpg_private_key: ${{ secrets.GPG_PRIVATE_KEY }}
      gpg_passphrase: ${{ secrets.GPG_PRIVATE_KEY_PASSPHRASE }}
      maven_username: ${{ secrets.MAVEN_USERNAME }}
      maven_central_token: ${{ secrets.MAVEN_CENTRAL_TOKEN }}
```

`publish` needs the `deployment` GitHub environment (or whatever you pass as `deployment_environment`)
to exist in the calling repo if you want required-reviewer protection on the Maven Central deploy step.

See each workflow's `on.workflow_call.inputs`/`secrets` block for the full list of overridable
knobs (Java version/distribution, Maven server id, extra deploy args, branch names, etc).
