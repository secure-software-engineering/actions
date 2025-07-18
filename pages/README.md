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
      - closed # Required, otherwise the preview is not deleted
      - synchronize
      - reopened
    # Only trigger on changes in documentation related files
    paths:
      - mkdocs.yml
      - docs/**
      - .github/workflows/doc_preview.yml

concurrency:
  group: gh-pages

jobs:
  deploy-preview:
    name: Preview documentation
    runs-on: ubuntu-latest
    # These permissions are needed to push and deploy the pages files
    permissions:
      contents: write
      pull-requests: write

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Create Documentation Preview
        uses: secure-software-engineering/actions/pages/doc-preview@develop
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