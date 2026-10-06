# upload-image

A Claude Code plugin that lets a session put screenshots and other images into GitHub issues, PRs and comments.

## Why

GitHub's own attachment upload can't be used from Claude cloud sessions, because the session proxy only sends JSON bodies to GitHub. So the image goes to an instance of the [uploads service](https://github.com/raunaqgupta/uploads), which stores it and returns a public URL, and the issue or PR shows it from that URL.

## What it does

It adds the `upload-image` skill, which tells Claude to:

1. Take a screenshot with Playwright, or use an existing image.
2. Upload it with `curl` to `$UPLOAD_URL?source=<owner>/<repo>`, using `$UPLOAD_KEY`.
3. Put the returned URL in the issue, PR or comment as an HTML `<img>` tag, through the GitHub MCP tools. A Markdown image (`![alt](url)`) loses its `!` when posted from a cloud session and shows as a link, so the skill avoids it, and falls back to a plain link if the tag is ever removed too.

## Requirements

- An instance of the uploads service, and a key for it that covers the repos you'll post images to.
- Two environment variables:
  - `UPLOAD_URL`: the instance's base URL, such as `https://cdn.example.com/github`.
  - `UPLOAD_KEY`: the key.
- If the environment's network access is limited, the instance's host in its allowed domains.
- Playwright and Chromium for screenshots. Claude cloud sessions have both.

## Install

```
claude plugin marketplace add raunaqgupta/ai-harness
claude plugin install upload-image@ai-harness
```

In a Claude cloud environment, put those two lines in the environment's setup script, and set `UPLOAD_URL` and `UPLOAD_KEY` as its environment variables.

## Notes

- Anyone with an image's URL can see it, even when the repo is private. The skill tells Claude not to upload anything sensitive.
- The skill tells Claude to stop and say what's missing if `UPLOAD_URL` or `UPLOAD_KEY` isn't set.
