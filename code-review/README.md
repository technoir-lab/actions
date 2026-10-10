# code-review

Reviews a pull request with [Claude Code](https://code.claude.com/docs/en/github-actions) when a maintainer comments `/review` on it. Claude runs Anthropic's [`code-review`](https://github.com/anthropics/claude-code/tree/main/plugins/code-review) plugin and posts its findings as inline comments, or as a single summary comment when it finds no issues.

## Setup

1. Install the [Claude GitHub App](https://github.com/apps/claude) on the repository or organisation.
2. Run `claude setup-token` and store the token as the `CLAUDE_CODE_OAUTH_TOKEN` Actions secret. Reviews use the usage limits of the Claude subscription that created the token.
3. Add a workflow that runs the action on pull request comments:

```yaml
name: Code Review

on:
  issue_comment:
    types: [ created ]

concurrency:
  group: ${{ github.workflow }}-${{ github.event.issue.number }}

jobs:
  review:
    name: Review
    if: github.event.issue.pull_request && github.event.issue.state == 'open' && startsWith(github.event.comment.body, '/review')
    runs-on: ubuntu-26.04

    permissions:
      contents: read
      pull-requests: read
      issues: write
      id-token: write

    steps:
      - name: Review
        uses: technoir-lab/actions/code-review@v1.1.1
        with:
          claude-code-oauth-token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
```

GitHub runs `issue_comment` workflows only from the default branch, so the workflow takes effect once it's merged. The `if` condition keeps runners from starting for other comments. The `issues: write` permission lets the action react to the `/review` comment; without it, the action skips the reaction.

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `claude-code-oauth-token` | | OAuth token for a Claude subscription, generated with `claude setup-token`. |
| `model` | `opus` | Model for the review session: an alias such as `opus` or `sonnet`, or a full model ID. |
| `required-role` | `maintain` | Minimum repository role required to request a review: `write`, `maintain` or `admin`. |

## Command

```
/review [low|medium|high|xhigh|max]
```

Post the command at the start of a pull request comment. The optional argument sets Claude's reasoning effort and defaults to `high`. The action reacts to an accepted request with 👀 before the review starts. Text on later lines of the comment is ignored. A comment whose first word isn't `/review`, such as `/reviewer`, doesn't start a review.

## Security

- **Maintainers only by default.** The commenter must have the role set by `required-role` or a higher one; a request from anyone else fails. Custom repository roles never qualify. `read` and `triage` aren't accepted: every GitHub user has `read` on a public repository, and neither role can push changes.
- **Pull requests from forks.** `issue_comment` workflows have access to secrets, so maintainers can request reviews of fork pull requests.
- **Default branch checkout.** The action checks out the default branch, never the pull request head. Claude Code loads settings and hooks from the working tree, so code from a fork must not run with access to secrets. Claude reads the pull request's changes through the GitHub API.

## Limitations

- The plugin skips draft and closed pull requests, pull requests it judges not to need a review, and pull requests that Claude has already commented on, so a second `/review` doesn't repeat a review.
- The plugin checks changes against `CLAUDE.md` files. Repositories that keep agent guidance in `AGENTS.md` can add a `CLAUDE.md` symlink to it.
- Anthropic validates the workflow that requests Claude's GitHub App token against the default branch. Calling this action from a reusable workflow in another repository can fail that check ([anthropics/claude-code-action#443](https://github.com/anthropics/claude-code-action/issues/443)), so add the workflow to each repository.
