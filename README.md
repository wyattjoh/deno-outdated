# deno-outdated

A GitHub Action that checks for outdated Deno dependencies, updates them, and opens a pull request.

Fills the gap left by Dependabot and Renovate, neither of which support Deno's `deno.json` imports or `deno.lock` today. Uses Deno's built-in [`deno outdated --update`](https://docs.deno.com/runtime/reference/cli/outdated/) under the hood.

## Usage

Add a scheduled workflow to your repository:

```yaml
name: Update Deno Dependencies

on:
  schedule:
    - cron: "0 9 * * 1" # Weekly on Monday at 9am UTC
  workflow_dispatch: {}

jobs:
  update:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: actions/checkout@v4

      - uses: denoland/setup-deno@v2
        with:
          deno-version: v2.x

      - uses: wyattjoh/deno-outdated@v1
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
```

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `token` | Yes | | GitHub token for creating pull requests |
| `latest` | No | `false` | Update beyond semver ranges to latest versions |
| `base-branch` | No | `main` | Base branch for the pull request |
| `labels` | No | | Comma-separated labels to add to the pull request |
| `branch` | No | `deps/deno-outdated` | Branch name for the update PR |
| `commit-message` | No | `deps: update deno dependencies` | Git commit message |

## Outputs

| Output | Description |
|---|---|
| `updated` | Whether any dependencies were updated (`true`/`false`) |
| `summary` | Markdown summary of outdated dependencies |
| `pull-request-number` | Number of the created/updated pull request (empty if none) |

## Behavior

- Runs `deno outdated` to detect outdated dependencies
- Runs `deno outdated --update` to update `deno.json` and regenerate `deno.lock`
- Creates a pull request with the changes (or updates an existing one)
- Idempotent: skips PR creation if no dependencies changed
- Supports both semver-compatible updates and `--latest` for major bumps

## Examples

### Update to latest versions (including major bumps)

```yaml
- uses: wyattjoh/deno-outdated@v1
  with:
    token: ${{ secrets.GITHUB_TOKEN }}
    latest: "true"
```

### Custom branch and labels

```yaml
- uses: wyattjoh/deno-outdated@v1
  with:
    token: ${{ secrets.GITHUB_TOKEN }}
    branch: deps/weekly-update
    labels: "dependencies,automated"
```
