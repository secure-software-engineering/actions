# Actions for deploying documentations using mkdocs

## Documentation Preview on Pull Requests

The **handle-pr-preview** action creates a preview of a documentation in a pull request.
It creates a temporary documentation deployment accessible via a unique URL.
After the pull request is merged or closed, the action deletes the corresponding preview.

A possible workflow **doc_preview.yml** can look like this:
```yaml
name: Documentation Preview

on:
  # Only trigger on pull requests that target the main branch of the repository
  # and when there are changes to documentation related files
  pull_request:
    branch:
      - main
    types:
      - opened
      - closed                        # Required, otherwise the preview is not deleted
      - synchronize
      - reopened
    paths:                            # Only trigger on changes in documentation related files
      - mkdocs.yml
      - docs/**
      - .github/workflows/documentation-preview.yml     # same name as this file!

concurrency:
  group: gh-pages

jobs:
  deploy-preview:
    name: Preview documentation
    runs-on: ubuntu-latest
    # These permissions are needed to push and deploy the pages files and comment the link to the documentation
    permissions:
      contents: write
      pull-requests: write

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Create Documentation Preview
        uses: secure-software-engineering/actions/documentation/handle-pr-preview@develop
        with:
          # We create a preview with a title and name that depend on the current pull 
          # request (e.g. on pull request #100, we have the name 'pr-100' and title 'Preview for PR-100')
          preview-name: pr-${{ github.event.pull_request.number }}
          preview-title: Preview for PR-${{ github.event.pull_request.number }}

```
This workflow triggers when there are changes in documentation related files.
It creates a preview that is accessible via `https://<username>.github.io/<repository>/<preview-name>`.
Observe that the URL contains the **preview-name**.
Hence, you should make sure that the name relates to the current pull request, e.g. you can use the pull request's number or the source branch.

Possible options:
- **preview-name**: A name for the preview. This name must not contain any spaces and should relate the current pull request (required)
- **preview-title**: A title for the preview. This title does not need to relate to the current pull request or the preview name (required)
- **gh-pages-branch**: The branch name where the GitHub pages are deployed to (optional, default: gh-pages)
- **enable-comment**: If 'true', the action adds a comment to the pull request that links the most recent preview. The comment is recreated after each commit that triggered the action (optional, default: true)



# Deploy Snapshot to GitHub Pages

The **handle-deployment** action deploys documentation versions to GitHub Pages using mike. 
It provides functionalities to create temporary versions that are updated with each new deployment and to create persistent documentation versions.

Assume we have a Maven project with the following setup:
- We have a branch **develop** that consists of the current development state (denoted with *SNAPSHOT*)
- When we push to the **develop** branch, we want to create a temporary documentation version that consists of the current development state
- When we create a tag (for example for a new release), we want to create a stable/persistent version

A possible workflow **doc_snapshot_deployment.yml** for the temporary SNAPSHOT versions can look like this:
```yaml
name: Deploy Documentation Snapshot

on:
  push:
    branches:
      - develop
      
concurrency:
  group: gh-pages

jobs:
  deploy-snapshot:
    name: Deploy documentation snapshot
    runs-on: ubuntu-latest
    
    # These permissions are needed to push and deploy the pages files
    permissions:
      contents: write

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      # We assume Maven is used. For other build tools, adapt this step
      - name: Extract Maven Version
        id: version
        run: |
          VERSION=$(mvn help:evaluate -Dexpression=project.version -q -DforceStdout)
          echo "version=$VERSION" >> $GITHUB_OUTPUT

      - name: Deploy Snapshot Documentation
        uses: secure-software-engineering/actions/documentation/handle-deployment@develop
        with:
          name: ${{ steps.version.outputs.version }}
          title: ${{ steps.version.outputs.version }}
```

This workflow triggers when there is a push/merge to the **develop** branch.
It creates a temporary documentation version with the current project version from the pom files.

Additionally, a possible workflow **doc_stable_deployment.yml** may look like this:
```yaml
name: Deploy Stable Documentation

on:
  # Trigger on arbitrary created tags
  create:
    tags:
      - '*'

concurrency:
  group: gh-pages

jobs:
  deploy-doc-stable:
  name: Deploy Stable Documentation
  runs-on: ubuntu-latest

  # These permissions are needed to push and deploy the pages files
  permissions:
    contents: write

  steps:
    - name: Checkout Repository
      uses: actions/checkout@v4
      with:
        # Check out the current created tag
        ref: ${{ github.ref }}
        fetch-depth: 0

    - name: Deploy Stable Documentation
      uses: secure-software-engineering/actions/documentation/handle-deployment@fix/documentation-deployment
      with:
        # We use the tag as name and title (e.g. x.y.z) and make the deployment stable
        # s.t. future deployments do not override this version
        name: ${{ github.ref_name }}
        title: ${{ github.ref_name }}
        stable: true
```

This workflow triggers when a new arbitrary tag is created.
It creates a stable documentation version with the tag's name that is not overridden by future deployments.

Possible options:
- **name**: A name for the documentation. This name must not contain any spaces. It is part of the final URL (required)
- **title**: A title for the documentation (required)
- **gh-pages-branch**: The branch name where the GitHub pages are deployed to (optional, default: gh-pages)
- **stable**: If `true`, the action deploys a persistent version. Otherwise, future deployments override the current version (optional, default: `false`)
- **aliases**: Aliases for the deployed version. An alias allows the specification of multiple names that point to the same version. For example, we can have a version `x.y.z` and an alias `latest` that redirects to `x.y.z`. This input may also be a list, e.g. `latest recent` (optional, default: `latest`)
