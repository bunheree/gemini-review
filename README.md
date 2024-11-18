# Review Pull Request with Gemini API

<p>
<a href="https://bunhere.com"><img src="./images/reviewpr.png" width="24" alt="WTM"></a>
<a href="#"><img src="https://img.shields.io/badge/Review PR-v1.0.1-blue" alt="Latest Stable Version"></a>
<a href="#"><img src="https://img.shields.io/badge/license-MIT-yellow" alt="License"></a>
</p>

This action reviews code in pull requests using Gemini.

## Inputs

- `model`: The model to use for the code review. Default: `gemini-1.5-pro-latest`.
- `pull_request_diff`: The diff of the pull request.
- `pull_request_chunk_size`: (Optional) The chunk size for review. Default: `3500`.
- `extra_prompt`: (Optional) Additional context for the review.
- `log_level`: (Optional) Log level for the action. Default: `DEBUG`.

## Outputs

- `review_comments`: Review comments from Gemini.

![Review - comment PR](/images/comment-PR.png)

## Example Usage

### 1. Create `GEMINI_API_KEY` and `GIT_TOKEN_KEY`

After having the 2 above keys, at the github repository, go to 

> **Settings** > **Secrets and Variales** > **Actions** > ***Add 2 new secret keys***.

## Example Usage

```yaml
name: "Review the code with Gemini"

on:
  pull_request:

jobs:
  review:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write

    steps:
      - uses: actions/checkout@v3

      - name: "Get diff of the pull request"
        id: get_diff
        shell: bash
        env:
          PULL_REQUEST_HEAD_REF: "${{ github.event.pull_request.head.ref }}"
          PULL_REQUEST_BASE_REF: "${{ github.event.pull_request.base.ref }}"
        run: |-
          git fetch origin "${{ env.PULL_REQUEST_HEAD_REF }}"
          git fetch origin "${{ env.PULL_REQUEST_BASE_REF }}"
          git checkout "${{ env.PULL_REQUEST_HEAD_REF }}"
          git diff "origin/${{ env.PULL_REQUEST_BASE_REF }}" > "diff.txt"
          {
            echo "pull_request_diff<<EOF";
            cat "diff.txt";
            echo 'EOF';
          } >> $GITHUB_OUTPUT

      - name: Review the code with Gemini
        uses: bunheree/gemini-review@v1.0.1
        with:
          model: "gemini-1.5-pro-latest"
          pull_request_diff: ${{ steps.get_diff.outputs.pull_request_diff }}
          log_level: "DEBUG"
        env:
          GEMINI_API_KEY: ${{ secrets.GEMINI_API_KEY }}
          GITHUB_TOKEN: ${{ secrets.GIT_TOKEN_KEY }}
```