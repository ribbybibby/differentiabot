# Chainctl Diff for Digestabot

This action is designed to add `chainctl image diff` results to pull requests
created by
[`chainguard-dev/digestabot`](https://github.com/chainguard-dev/digestabot).

This allows you to understand the implications of an update to your Chainguard
images in terms of the vulnerabilities and packages that are being
added/removed.

## Usage

Configure an identity in your Chainguard organization for use by your workflow.

```
chainctl iam identity create github github-repo-name \
    --parent=your.org \
    --github-repo=my-org/repo-name \
    --role=registry.puller
```

Then, you can configure the workflow.

```yaml
name: Image digest update

on:
  workflow_dispatch:
  schedule:
    # At the end of every day
    - cron: "0 0 * * *"

jobs:
  image-update:
    name: Image digest update
    runs-on: ubuntu-latest

    permissions:
      contents: write # to push the updates
      pull-requests: write # to open Pull requests
      id-token: write # used to sign the commits using gitsign

    steps:
    - uses: actions/checkout@v4

    - uses: chainguard-dev/setup-chainctl@[VERSION]
      with:
        # the ID of the identity
        identity: "..."

    - uses: chainguard-dev/digestabot@v1
      id: digestabot
      with:
        token: ${{ secrets.GITHUB_TOKEN }}

    - uses: chainguard-dev/digestabot-diff@v1
      with:
        token: ${{ secrets.GITHUB_TOKEN }}
        json: ${{ steps.digestabot.outputs.json }}
        pull_request_number: ${{ steps.digestabot.outputs.pull_request_number }}
```
