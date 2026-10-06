# ai-harness

A [Claude Code plugin marketplace](https://code.claude.com/docs/en/plugins) hosting a set of small, repo-agnostic plugins. Each plugin is self-contained under `plugins/<name>/`, with its own README.

## Install the marketplace

Add the marketplace once, then install the plugins you want from it (see each plugin's section below):

```
claude plugin marketplace add raunaqgupta/ai-harness
```

Or point at a local checkout:

```
claude plugin marketplace add /path/to/ai-harness
```

## ai-cost-calculator

Tracks token usage per commit on a branch and posts (and keeps updated) a PR comment with the approximate USD cost of the AI work behind it.

```
claude plugin install ai-cost-calculator@ai-harness
```

See the [ai-cost-calculator README](plugins/ai-cost-calculator/README.md).

## git-workflow

Applies a generic clarify → issue → branch → validate → code pipeline to every Claude Code session, regardless of which repo it's running in.

```
claude plugin install git-workflow@ai-harness
```

See the [git-workflow README](plugins/git-workflow/README.md).

## prompt-classifier

Classifies every prompt as a question, an issue, or a PR via TypeSafe, and nudges the session toward the matching lane of the `git-workflow` pipeline. Needs a TypeSafe API key, which Claude Code asks for when the plugin is enabled.

```
claude plugin install prompt-classifier@ai-harness
```

See the [prompt-classifier README](plugins/prompt-classifier/README.md).

## upload-image

Lets a session put screenshots and other images into GitHub issues and PRs, through an instance of the [uploads service](https://github.com/raunaqgupta/uploads). Needs the `UPLOAD_URL` and `UPLOAD_KEY` environment variables.

```
claude plugin install upload-image@ai-harness
```

See the [upload-image README](plugins/upload-image/README.md).
