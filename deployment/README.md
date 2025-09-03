# Actions for Deployments to Maven Central

We provide a set of actions that allow the deployment of projects to Maven Central.
This includes the following actions:

- An action to deploy snapshot and release versions to Maven CentralAAAA
- An action to automatically update project versions

## Maven Deployment

The **maven-deployment** action deploys Maven artifacts to Maven Central.
It provides the following features:

- Setting up Java and Maven credentials
- Importing and using your GPG key
- Mode for snapshot and release version deployments
- Version checks to avoid accidental downgrades

#### Inputs

| Name                | Required | Default       | Description                                                                                      |
|---------------------|----------|---------------|--------------------------------------------------------------------------------------------------|
| `java-distribution` | Yes      | —             | Java distribution to use (e.g., `adopt`, `temurin`).                                             |
| `java-version`      | Yes      | —             | Java version to use (e.g., `11`, `17`).                                                          |
| `maven-username`    | Yes      | —             | Username for Maven Central or Sonatype (e.g. `deployment` for token based authentication).       |
| `maven-token`       | Yes      | —             | User token or password for Maven Central or Sonatype.                                            |
| `gpg-private-key`   | Yes      | —             | Armored GPG private key for signing artifacts.                                                   |
| `gpg-passphrase`    | Yes      | —             | Passphrase for the GPG private key.                                                              |
| `mvn-cli-args`      | No       | `-DskipTests` | Additional arguments passed to the Maven deploy command.                                         |
| `deploy-mode`       | No       | `release`     | Deployment mode: `release` or `snapshot`.                                                        |
| `version-check`     | No       | `false`       | If `true`, a check is performed to ensure the new version is higher than the last committed one. |

The action expects an armored GPG key. You can export your key with the following command: `gpg --armor --export-secret-keys <key_id> gpg_key.asc`.

You may also extend the Maven deploy command.
By default, the deployment skips all tests (`-DskipTests`).
However, in most cases, it is recommended to use a profile in the pom files that consists of required deployment plugins (e.g. the `maven-gpg-plugin`).
Hence, one can extend the Maven command with activating the profile, that is, using the additional command `-DskipTests -Pdeployment`, where `deployment` is the profile.

Setting the `deploy-mode` allows the configuration of the deployment mode.
If this input is `snapshot`, the project's version is expected to end with `-SNAPSHOT`.
If the input is `release`, the project's version must not end with `-SNAPSHOT`.
The action fails if these rules are violated.

#### Example Usage

Assuming we have stored the Maven credentials in the GitHub secrets `MAVEN_USERNAME` and `MAVEN_TOKEN`, and the GPG key and its passphrase in the secrets `GPG_PRIVATE_KEY` and `GPG_PASSPHRASE`, a possible workflow file for *snapshot* deployments may look like this:

```yaml
name: Deploy SNAPSHOT versions

on:
  push:
    branches:
      - develop

jobs:
  deploy-snapshot:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source code
        uses: actions/checkout@v4

      - name: Deploy SNAPSHOT version
        uses: secure-software-engineering/actions/deployment/maven-deployment@develop
        with:
          java-distribution: adopt
          java-version: 11
          maven-username: ${{ secrets.MAVEN_USERNAME }}
          maven-token: ${{ secrets.MAVEN_TOKEN }}
          gpg-private-key: ${{ secrets.GPG_PRIVATE_KEY }}
          gpg-passphrase: ${{ secrets.GPG_PASSPHRASE }}
          mvn-cli-args: '-DskipTests -Pdeployment'
          deploy-mode: snapshot
```

## Update Snapshot Versions

The **update-snapshot-version** action creates a pull requests from a new branch that patches the project version to the next snapshot version.
For example, if the current version is `1.2.3`, the version from the pull request will be `1.2.4-SNAPSHOT`

#### Inputs

| Name              | Required | Default            | Description                                                      |
|-------------------|----------|--------------------|------------------------------------------------------------------|
| `token`           | Yes      | —                  | GitHub token used to authenticate the GitHub actions bot.        |
| `source-branch`   | Yes      | —                  | Branch to create the new branch from (e.g., `main` or `master`). |
| `target-branch`   | Yes      | —                  | Branch where the PR should be merged into.                       |
| `snapshot-branch` | No       | `snapshot-version` | Name of the branch with the updated SNAPSHOT version.            |

#### Example Usage

Assume we have the following setup:
Our repository has a branch `develop` that holds the current development version (i.e. the snapshots) and a branch `master` that holds the released versions (i.e. the non-snapshot versions).
A possible workflow may look like this:

```yaml
name: SNAPSHOT Update

on:
  workflow_dispatch:

jobs:
  update-snapshot:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source code
        uses: actions/checkout@v4

      - name: Bump to next SNAPSHOT version
        uses: secure-software-engineering/actions/deployment/update-snapshot-version@develop
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          source-branch: master
          target-branch: develop
          snapshot-branch: snapshot-update
```

When triggered this workflow creates a new branch `snapshot-update` based on the `master` branch and opens a pull request targeting the `develop` branch.
The pull request consists of the updated snapshot version for the project.

## Combining the Actions

A common use case is the combination of deploying a released version and updating the version to the next snapshot version.
We can use the `maven-deployment` action and the `update-snapshot-version` action to create a workflow that deploys a release and updates the version in one step.
A possible workflow may look like this:

```yaml
name: Deploy and Update Version

on:
  push:
    tags:
      - '*'

jobs:
  deploy-and-update-version:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source code
        uses: actions/checkout@v4
        with:
          ref: master
          fetch-depth: 0
        
      - name: Deploy release version
        uses: secure-software-engineering/actions/deployment/maven-deployment@develop
        with:
          java-distribution: adopt
          java-version: 11
          maven-username: ${{ secrets.MAVEN_USERNAME }}
          maven-token: ${{ secrets.MAVEN_TOKEN }}
          gpg-private-key: ${{ secrets.GPG_PRIVATE_KEY }}
          gpg-passphrase: ${{ secrets.GPG_PASSPHRASE }}
          mvn-cli-args: '-DskipTests -Pdeployment'
          deploy-mode: release

      - name: Bump to next SNAPSHOT version
        uses: secure-software-engineering/actions/deployment/update-snapshot-version@develop
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          source-branch: master
          target-branch: develop
          snapshot-branch: snapshot-update
```

Using this workflow, we have the following setup:

- When creating a tag on the GitHub remote, the workflow triggers
- The workflow checks out the `master` branch that contains the project version that should be released
- The workflow deploys the project as release to Maven Central
- After the deployment, the workflow creates a pull request with the updated snapshot version

Using this workflow, the tags on the GitHub remote and the deployed versions stay in sync.
Additionally, the development versions are updated automatically.
You just need to make sure to update the snapshot versions on the `master` or `develop` branch to the next desired release version.
For example, this can be done manually or by using another workflow (e.g. a workflow that updates the version when merging the `develop` branch into the `master` branch).
