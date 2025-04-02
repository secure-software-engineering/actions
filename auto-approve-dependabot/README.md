# Auto Approve Dependabot PRs

This reusable GitHub Action approves PRs created by Dependabot.

## Usage

```yaml
uses: sse/actions/auto-approve-dependabot@main
with:
  token: ${{ secrets.SSE_BOT_PAT }}