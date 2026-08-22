---
name: social-post-publish-ego-browser
description: "Publish an approved text+image post to Facebook and X (Twitter) by driving the logged-in web UI through ego-browser. Use when: (1) posting to Facebook/X without API tokens, (2) publishing social content that reuses the user's existing login state, or (3) user explicitly asks to post via the browser."
---

# Social Post Publishing via ego-browser

Publish self-contained posts (text + image, no outbound links) to Facebook and X by driving the real web UI through ego-browser, which inherits the user's login state. Includes a mandatory human-approval gate before anything goes public.

## Approval gate (mandatory)

Never publish external content without the user approving the EXACT final text and image first. Draft, present, wait for explicit approval, then publish. Deleting/fixing your own erroneous posts is allowed; new external content is not.

Also prefer self-contained posts: platforms downrank outbound links, and internal workspace links (Notion, docs) should never be exposed.

## Required tools / APIs

- ego-browser CLI (`ego-browser nodejs <<'EOF' ... EOF`), inheriting the user's login state.
- One local image file (PNG/JPEG) to attach.

## Skills

### facebook_post

Open the composer, fill text, attach image via the hidden file input, publish via the Post button inside the "Create post" dialog.

```js
// ego-browser nodejs heredoc
const task = await useOrCreateTaskSpace('publish social post')
await openOrReuseTab('https://www.facebook.com', { wait: true, timeout: 25 })

// 1. Open composer: the inline trigger button contains "What's on your mind"
let snap = await snapshotText()
let lines = snap.split('\n')
let ref = null
for (let i = 0; i < lines.length; i++) {
  if (/What's on your mind/.test(lines[i])) {
    for (let j = i; j >= 0 && j > i - 4; j--) {
      const m = lines[j].match(/ref=(\d+)/)
      if (m) { ref = m[1]; break }
    }
    break
  }
}
await click('@' + ref)
await wait(4)

// 2. Fill the dialog field by CSS (refs in the dialog are unstable)
await fillInput('div[role="dialog"] input[placeholder*="on your mind"], div[role="dialog"] [contenteditable="true"]', POST_TEXT)

// 3. Attach image through the hidden file input
await uploadFile('div[role="dialog"] input[type="file"]', IMAGE_PATH)
await wait(5)

// 4. Dismiss hashtag-suggestion dropdown, then click Post inside the Create post dialog
await pressKey('Escape')
await js(String.raw`(() => {
  const dlg = [...document.querySelectorAll('div[role="dialog"]')].find(d => /Create post/.test(d.textContent))
  const btn = [...dlg.querySelectorAll('div[role="button"], button')].find(b => (b.textContent || '').trim() === 'Post')
  btn.click()
})()`)
await wait(5)

// 5. Verify: dialog closed and post visible on the profile
await gotoAndWait('https://www.facebook.com/<username>', { timeout: 20 })
cliLog('published: ' + (await snapshotText()).includes(POST_TEXT.slice(0, 40)))
```

### x_post

Fill the home composer textbox, attach the image, click the inline Post button.

```js
await openOrReuseTab('https://x.com', { wait: true, timeout: 25 })
await fillInput('div[role="textbox"][aria-label="Post text"]', POST_TEXT)   // <= 280 chars incl. hashtags
await uploadFile('input[type="file"][accept*="image"]', IMAGE_PATH)
await wait(4)
await js(String.raw`(() => document.querySelector('button[data-testid="tweetButtonInline"]').click())()`)
await wait(6)
cliLog('sent: ' + /Your post was sent/i.test(await snapshotText()))
```

### image_sourcing

Prefer, in order: (1) an existing brand asset, (2) the user's logged-in Gemini subscription via ego-browser (ask it to generate, then export the result image with a canvas `toDataURL` round-trip — `fetch(blob:)` is CSP-blocked on gemini.google.com), (3) a paid generation API with explicit consent.

```js
// export a generated image that only exists as a blob: URL
const dataUrl = await js(String.raw`(() => {
  const img = [...document.querySelectorAll('img')].find(i => i.naturalWidth > 300)
  const c = document.createElement('canvas')
  c.width = img.naturalWidth; c.height = img.naturalHeight
  c.getContext('2d').drawImage(img, 0, 0)
  return c.toDataURL('image/png')
})()`)
const fs = await import('node:fs')   // ESM runtime: dynamic import, not require()
fs.writeFileSync(OUT_PATH, Buffer.from(dataUrl.split(',')[1], 'base64'))
```

## Output format

Report per platform: published (true/false), verification method (profile/timeline check or "Your post was sent"), and the exact text that went out.

## Rate limits / Best practices

- One approval per exact content; re-approve after any edit.
- Keep posts self-contained: no outbound links, no internal workspace URLs.
- X inline composer button is `tweetButtonInline`; the modal composer uses `tweetButton` — check both.
- Facebook shows a hashtag-suggestion dropdown while typing `#tags`; press Escape before clicking Post.
- After publishing, verify on the profile/timeline, not just by the dialog closing.
- Close the ego task space when done (`completeTaskSpace(name, { keep: false })`).

## Agent prompt

```text
You have social publishing capability via ego-browser. When asked to publish a post:

1. Draft the exact final text (self-contained, no links) and select/produce the image.
2. Present both to the user and wait for explicit approval.
3. Publish via the facebook_post / x_post flows above, attaching the image.
4. Verify each publication on the profile/timeline and report the result.

Never publish without approval. Never expose internal workspace links.
```

## Troubleshooting

**Composer dialog opens but no field ref found:**
- Symptom: snapshotText shows the dialog but refs are missing/unstable.
- Solution: address the field by CSS selector (`input[placeholder*="on your mind"]` or `[contenteditable="true"]` inside `div[role="dialog"]`), not by ref.

**Post button click does nothing:**
- Symptom: click on a "Post" button from the wrong dialog (notifications etc.).
- Solution: scope the search to the dialog whose textContent contains "Create post" (Facebook) or use the exact data-testid (X).

**Generated image only available as blob: URL:**
- Symptom: `fetch(blob:...)` fails with TypeError: Failed to fetch (CSP).
- Solution: draw the `<img>` to a canvas and export with `toDataURL` (see image_sourcing).

## See also

- [../browser-automation-agent/SKILL.md](../browser-automation-agent/SKILL.md) — general browser automation patterns

---

## Notes

- ego-browser heredocs are ESM: use `await import('node:fs')`, never `require`.
- `wait()` values are seconds.
- Refs (`@N`) are only valid for the latest snapshotText call; prefer CSS/loc for elements you need across rounds.
