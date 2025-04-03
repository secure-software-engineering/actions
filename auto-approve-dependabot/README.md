# Auto Approve Dependabot PRs

This reusable GitHub Action approves PRs created by Dependabot.

## Usage

```yaml
uses: secure-software-engineering/actions/auto-approve-dependabot@develop
with:
  token: ${{ secrets.SSE_BOT_PAT }}
```