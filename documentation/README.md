# Actions for deploying documentations using mkdocs

## Documentation Preview on Pull Requests

The **doc-preview** action creates a preview of a documentation in a pull request. It creates a temporary documentation deployment accessible via a unique URL. After the pull request is merged or closed, the action deletes the corresponding preview.

A possible action **doc_preview.yml** can look like this:
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
          token: ${{ secrets.GITHUB_TOKEN }}
          # We create a preview with a title and name that depend on the current pull 
          # request (e.g. on pull request #100, we have the name and title 'pr-100')
          preview-name: pr-${{ github.event.pull_request.number }}
          preview-title: Preview for pr-${{ github.event.pull_request.number }}

```
This action triggers when there are changes in documentation related files. It creates a preview that is accessible via `https://<username>.github.io/<repository>/<preview-name>`. Observe that the URL contains the **preview-name**. Hence, you should make sure that the name relates to the current pull request, e.g. you can use the pull request's number or the source branch.

Possible options:
- **token**: The GitHub token for authentication (required)
- **preview-name**: A name for the preview. This name must not contain any spaces and should relate the current pull request (required)
- **preview-title**: A title for the preview. This title does not need to relate to the current pull request or the preview name (required)
- **gh-pages-branch**: The branch name where the GitHub pages are deployed to (optional, default: gh-pages)
- **enable-comment**: If 'true', the action adds a comment to the pull request that links the most recent preview. The comment is recreated after each commit that triggered the action (optional, default: true)



# Deploy Snapshot to GitHub Pages

The **branch-snapshot** deploys documentation snapshots to GitHub Pages using mike. It automatically determines version information from Maven projects and creates branch-specific documentation deployments.

A possible action **deploy_snapshot.yml** can look like this:
```yaml
name: Deploy Documentation Snapshot

on:
  push:
    branches:
      - main
      - develop
      - 'release/**'
    # Only trigger when there are changes to documentation related files
    paths:
      - mkdocs.yml
      - docs/**
      - pom.xml
      - build.gradle
      - '.github/workflows/documentation-version.yml'

jobs:
  deploy-snapshot:
    name: Deploy documentation snapshot
    runs-on: ubuntu-latest
    
    # These permissions are needed to push and deploy the pages files
    permissions:
      contents: write
      pages: write
      id-token: write

    steps:
      - name: Deploy Documentation Snapshot
        uses: secure-software-engineering/actions/documentation/pin-version@develop
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
```

This action triggers when there are changes to documentation files or build configurations. It deploys documentation snapshots for each branch, making it easy to preview documentation changes across different development branches.

Possible options:
- **token**: The GitHub token for authentication (required)
- **latest_branch**: Branch that should get the "latest" alias (optional, default: 'develop')
- **version**: Version source - "maven", "gradle", or custom string (optional, default: 'maven')