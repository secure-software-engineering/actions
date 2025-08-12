# Actions for Maven Central Deployment

## Deploy to Maven Central

The **maven-central-deploy** action deploys Maven artifacts to Maven Central with GPG signing and automatically creates a pull request to update the project version to the next SNAPSHOT version.

A possible action **deploy_maven_central.yml** can look like this:

```yaml
name: Deploy to Maven Central
run-name: Deploy run ${{ github.run_number }}

on:
  workflow_dispatch:

jobs:
  deployment:
    name: Deployment
    runs-on: ubuntu-latest
    environment: deployment
    steps:
      - name: Deploy to Maven Central
        uses: secure-software-engineering/actions/maven-central-deploy@develop
        with:
          maven_username: ${{ secrets.SONATYPE_USER }}
          maven_token: ${{ secrets.SONATYPE_PW }}
          gpg_private_key: ${{ secrets.GPG_PRIVATE_KEY }}
          gpg_passphrase: ${{ secrets.GPG_PRIVATE_KEY_PASSPHRASE }}
          token: ${{ secrets.GITHUB_TOKEN }}
```

**Possible options:**
- **maven_username**: Maven Central username (required)
- **maven_token**: Maven Central token (required)
- **gpg_private_key**: GPG private key for signing artifacts (required)
- **gpg_passphrase**: GPG private key passphrase (required)
- **token**: GitHub token for creating pull requests (required)
- **java_version**: Java version to use for building (optional, default: '17')
- **java_distribution**: Java distribution to use (optional, default: 'temurin')
- **target_branch**: Target branch for SNAPSHOT version PR (optional, default: 'develop')
- **release_branch**: Branch to release from (optional, default: 'master')
- **skip_tests**: Skip tests during deployment (optional, default: 'true')
- **snapshot_branch_name**: Base name for SNAPSHOT branch (optional, default: 'snapshot_version')