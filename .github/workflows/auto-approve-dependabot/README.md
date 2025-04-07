# Auto Approve Dependabot PRs

This reusable GitHub Action approves PRs created by Dependabot.

## Usage

```yaml
uses: secure-software-engineering/actions/.github/workflows/auto-approve-dependabot/auto-approve-dependabot.yml@develop
with:
  token: ${{ secrets.SSE_BOT_PAT }}
```