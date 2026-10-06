---
name: upload-image
description: Put an image into a GitHub issue, PR or comment from a cloud session, such as a screenshot of a page or app, or an existing PNG, JPEG, GIF or WebP file. Use whenever an issue or PR should show an image. GitHub's own attachment upload doesn't work from cloud sessions, so images go to the uploads service, and the issue or PR links them by URL.
---

# Upload an image to a GitHub issue or PR

GitHub's attachment upload can't be used from cloud sessions: the session proxy only sends JSON bodies to GitHub. Don't try it. Store the image with an instance of the uploads service (https://github.com/raunaqgupta/uploads) instead, and put its URL in the issue or PR.

The environment provides two variables:

- `UPLOAD_URL`: the instance's base URL, such as `https://cdn.example.com/github`.
- `UPLOAD_KEY`: the key for uploading to it.

If either isn't set, or the instance can't be reached, tell the user what's missing rather than trying another host.

## 1. Get the image

Save it in your scratchpad directory. In cloud sessions Chromium and Playwright are already installed; don't run `playwright install`.

- **A page as it loads:**

  ```sh
  playwright screenshot --full-page --viewport-size=1280,800 --wait-for-timeout=1000 <url> <scratchpad>/shot.png
  ```

- **A page that needs steps first** (signing in, clicking, filling a form): write a CommonJS script and run it with `NODE_PATH=$(npm root -g) node <script>.cjs`. Playwright is installed globally, and ES modules ignore `NODE_PATH`, so use `require`:

  ```js
  const { chromium } = require("playwright");
  (async () => {
    const browser = await chromium.launch();
    const page = await browser.newPage({ viewport: { width: 1280, height: 800 } });
    await page.goto("http://localhost:3000/");
    // ...steps...
    await page.screenshot({ path: "<scratchpad>/shot.png", fullPage: true });
    await browser.close();
  })();
  ```

Look at the image (Read it) before uploading, to check it shows what you mean and nothing private.

## 2. Upload it

`source` is the repo the issue or PR is in, as `<owner>/<repo>`. Set `Content-Type` to the file's real type (`image/png`, `image/jpeg`, `image/gif` or `image/webp`).

```sh
curl -sS -X POST "$UPLOAD_URL?source=<owner>/<repo>" \
  -H "Authorization: Bearer $UPLOAD_KEY" \
  -H "Content-Type: image/png" \
  --data-binary @<scratchpad>/shot.png \
  -w '\n%{http_code}\n'
```

It returns `201 {"url": "<UPLOAD_URL>/<owner>/<repo>/<date>-<uuid>.png"}`.

- Never print `UPLOAD_KEY`, write it to a file, or put it in a command's text; use the variable.
- Limits: 10 MB, and PNG, JPEG, GIF or WebP only, checked against the file's first bytes. No SVG.
- Errors are JSON `{"error": "..."}`: `400` bad source, `401` missing or unknown key, `403` the key doesn't cover the source, `413` too big, `415` wrong or mismatched type.

## 3. Put it in the issue or PR

Write the URL as a Markdown image in the body or comment, with alt text that says what it shows:

```markdown
![The settings page after saving, with the error banner](<url from step 2>)
```

Post it with the GitHub MCP tools as usual: `issue_write` or `update_pull_request` for a body, `add_issue_comment` for a comment, `create_pull_request` for a new PR.

## Things to know

- **Anyone with the URL can see the image**, even when the repo is private. Don't upload anything with secrets, tokens, personal data or other content the user wouldn't want public. If in doubt, ask first.
- **Deleting:** `curl -sS -X DELETE -H "Authorization: Bearer $UPLOAD_KEY" <url>` returns `204`. GitHub's image proxy may keep showing a deleted image for a while.
