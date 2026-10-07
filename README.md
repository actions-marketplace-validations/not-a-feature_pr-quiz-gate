![PR Quiz Gate: Understand the change before merging](assets/banner.svg)

# PR Quiz Gate

A GitHub Action that asks PR authors to explain their changes before merging.
Copilot or Claude writes the quiz and grades written answers. Multiple-choice
answers are checked directly.

## Setup

1. Add a `COPILOT_GITHUB_TOKEN` repository secret using a fine-grained personal
   token with **Account permissions → Copilot Requests → Read** and active Copilot
   access. See the [token instructions](https://github.github.com/gh-aw/reference/auth/#copilot_github_token).
2. Copy [the Copilot workflow](examples/quiz-gate-copilot.yml) to
   `.github/workflows/quiz-gate.yml` on your default branch. Set the Action reference
   to `not-a-feature/pr-quiz-gate@COMMIT_SHA` with a tested SHA, and set `mode: required`.
   Keep the example's events, permissions, and concurrency settings.
3. In your branch rules, require the **`quiz-gate`** status from GitHub Actions and
   up-to-date checks. Choose the status rather than the workflow job, and leave
   bypass actors empty to enforce the rule for administrators too.

Prefer Claude? Use [the Claude workflow](examples/quiz-gate.yml) with an
`ANTHROPIC_API_KEY` secret.

Or preview a setup PR with the installer:

```sh
uv run python install.py OWNER/REPO --action not-a-feature/pr-quiz-gate@COMMIT_SHA
```

Add `--apply` to create it. You'll need an authenticated `gh` CLI and the provider
token in your environment. Set branch rules separately.

## Taking the quiz

The bot posts a quiz with code links and an answer form. Copy the form into your
own PR comment, answer every question, and check **Submit answers** when ready.

You can also submit numbered answers:

```text
/quiz answer CURRENT_QUIZ_ID
1: B
2: Explain the implementation and what you intend.
```

Only the PR author can answer. By default, passing takes a majority of correct
answers, and any unresolved decisions still block merging even with a passing
score. Results explain what needs work. New commits need a new quiz.
To submit another attempt, post a new comment rather than editing a graded one.

| Command | What it does |
| --- | --- |
| `/quiz` | Start a missing or outdated quiz |
| `/quiz retry` | Get fresh questions after failing |
| `/quiz override REASON` | Override as a maintainer or admin, when enabled |

## Configuration

The default mode is `advisory`. Use `required` to block merging. Defaults allow
up to 3 questions and 3 attempts per PR revision, with maintainer overrides enabled.
See [action.yml](action.yml) for models, score thresholds, and all other inputs.
Set a stable `state-key` to keep quizzes valid when rotating the provider token.

## Limitations

The provider receives changed-file contents, diffs, and human PR comments, but
cannot run code or use tools. Answer keys are encrypted in bot comments and tied
to the PR revision and author.

Binary files and changes over the context budget cause an error. Unchanged
dependencies, review threads, and local agent sessions are excluded. Grading can
be wrong, and authors can use AI to answer. Merge queues are unsupported.
Bot-authored PRs need an override.

## Development

```sh
uv sync --locked
uv run python -m unittest discover -s tests -v
```

Tests mock the providers and GitHub APIs. See [integration testing](INTEGRATION_TESTING.md)
to test a branch or commit without a Marketplace release. Choose a commit with
checkbox support, since older tags keep their original behavior.
