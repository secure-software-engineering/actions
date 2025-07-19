# Actions for Javadoc Documentation

## Publish Javadoc

The **javadoc** action generates and publishes Javadoc documentation to GitHub Pages. It builds the Javadoc using Maven, publishes it to a specified directory, and maintains a "latest" symlink pointing to the most recent version from the specified branch.

A possible action **publish_javadoc.yml** can look like this:

```yaml
name: Publish Javadoc

on:
  push:
    branches:
      - main
      - develop
    # Only trigger when there are changes to Java files or documentation
    paths:
      - 'src/**/*.java'
      - 'pom.xml'
      - '.github/workflows/publish_javadoc.yml'

jobs:
  publish-javadoc:
    name: Generate and publish Javadoc
    runs-on: ubuntu-latest
    
    # These permissions are needed to publish to GitHub Pages
    permissions:
      contents: read
      pages: write
      id-token: write

    steps:
      - name: Publish Javadoc Documentation
        uses: secure-software-engineering/actions/javadoc@develop
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          title: ${{ github.ref_name }}
```

This action triggers when there are changes to Java source files or the Maven configuration. It generates Javadoc documentation and publishes it to `https://<username>.github.io/<repository>/apidocs/<title>/`. When pushing to the latest branch (default: `develop`), it also updates the "latest" symlink to point to the new documentation.

**Possible options:**
- **token**: The GitHub token for authentication (required)
- **title**: Title/version name for the documentation directory (required)
- **java_version**: Java version to use for building (optional, default: '17')
- **java_distribution**: Java distribution to use (optional, default: 'temurin')
- **latest_branch**: Branch that should update the "latest" symlink (optional, default: 'develop')
