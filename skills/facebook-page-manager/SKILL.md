---
name: facebook-page-manager
description: "Build and deploy an autonomous Facebook page agent in Node.js (Graph API posting, scheduling, comment replies, analytics, monetization tracking) and take it from dry-run to live production. Use when the user wants to automate, grow, or monetize a Facebook page with minimal human involvement."
---

# Facebook Page Manager Agent

Builds a standalone Node.js agent that manages a Facebook page: AI-drafted content,
scheduled posting cadence, comment auto-replies, analytics, and monetization-gate
tracking. Covers the credential setup, the Graph API pitfalls that block "going
live", and deploying it as a long-running service so it runs without human help.

## When to use

- User asks to "build a Facebook agent / automate a Facebook page / grow a page"
- User wants scheduled posting, comment auto-replies, or monetization tracking
- User asks to make the agent "run on its own / continue without our help"
- Taking an existing dry-run Facebook agent into production

## Required tools / APIs

- **Facebook Graph API** (free): page posting, scheduling, comments, insights. Requires a Facebook Developer app + token.
- **System User token** (free, recommended for autonomy): never expires. Created in Business Manager.
- **Local LLM via ollama** (free, optional): content drafting. Any OpenAI-compatible endpoint works.
- **node-cron / telegraf** (free): scheduler + optional Telegram control.

```bash
npm install node-cron telegraf dotenv
```

## Credential setup (the 90% blocker)

For an agent that runs unattended, use a **System User token** because it never
expires (regular user tokens die in ~60 days; Explorer tokens die in hours).

1. Create a developer app at https://developers.facebook.com → My Apps → Create App → Business. Add the **Facebook Login** product. Keep it in **Development mode**.
2. Create a system user: https://business.facebook.com/settings/system-users → Create → Generate New Token → select your app.
3. Select **all** these permissions (each is needed):
   - `pages_show_list` — list pages
   - `pages_read_engagement` — read posts
   - `pages_manage_posts` — post / schedule
   - `pages_manage_engagement` — comment replies
   - `pages_read_user_content` — read comment text
   - `read_insights` — analytics metrics
4. Assign the page to the system user (Business Settings → System Users → user → Assign assets → the page).
5. Verify with the debug endpoint:
   ```bash
   curl -s "https://graph.facebook.com/v21.0/debug_token?input_token=TOKEN&access_token=TOKEN"
   ```
   Confirm `type: SYSTEM_USER`, `expires_at: 0`, and all 6 scopes present.

## Graph API client (Node.js)

```javascript
const BASE = "https://graph.facebook.com/v21.0";
let userToken = process.env.FB_USER_TOKEN;   // system user token
let pageToken = null;

async function pageTokenFor(pageId) {
  if (pageToken) return pageToken;
  const res = await fetch(`${BASE}/me/accounts?fields=id,access_token&access_token=${userToken}`);
  const data = await res.json();
  pageToken = data.data.find((p) => String(p.id) === String(pageId))?.access_token;
  if (!pageToken) throw new Error("page not linked to this token");
  return pageToken;
}

async function graph(path, params = {}, method = "GET", token = userToken) {
  const url = new URL(`${BASE}/${path}`);
  if (method === "GET") Object.entries(params).forEach(([k, v]) => url.searchParams.set(k, v));
  url.searchParams.set("access_token", token);
  const opts = { method };
  if (method !== "GET") { opts.headers = { "Content-Type": "application/json" }; opts.body = JSON.stringify({ ...params, access_token: token }); }
  const res = await fetch(url, opts);
  const data = await res.json();
  if (data.error) throw new Error(`Graph API ${data.error.code}: ${data.error.message}`);
  return data;
}

// read posts (use /posts, NOT /feed — see gotchas)
const feed = await graph(`${pageId}/posts`, { limit: 25, fields: "id,message,created_time,comments.limit(10){id,message,created_time}" }, "GET", await pageTokenFor(pageId));
// schedule a post (unix SECONDS, published:false)
await graph(`${pageId}/feed`, { message, published: false, scheduled_publish_time: Math.floor(Date.now() / 1000) + 8 * 3600 }, "POST", await pageTokenFor(pageId));
// reply to a comment
await graph(`${commentId}/replies`, { message }, "POST", await pageTokenFor(pageId));
// insights
await graph(`${pageId}/insights`, { metric: "page_fans,page_impressions,page_engaged_users,page_video_views", period: "day" }, "GET", await pageTokenFor(pageId));
// current scheduled posts (ground truth for reconciliation)
const sched = await graph(`${pageId}/scheduled_posts`, { fields: "id,created_time" }, "GET", await pageTokenFor(pageId));
```

## Production gotchas (learned the hard way)

- **`/feed` reads are broken for many accounts** — `GET /{page}/feed` returns `error 10 requires pages_read_engagement` even with the permission granted. Use `GET /{page}/posts` instead. (POSTing to `/feed` is fine; only *reading* `/feed` is broken.)
- **The new Pages experience requires a PAGE token** for `/posts`, `/scheduled_posts`, etc. A system-user or user token fails with `error 190 subcode 2069032`. Derive the page token from `/me/accounts` (page tokens don't expire for pages you administer) and use it for all page-scoped calls.
- **Missing `read_insights` masks as `error 100 "The value must be a valid insights metric"`** for any metric — it's a permission problem, not a metric problem.
- **Reading comment text needs `pages_read_user_content`** on top of `pages_read_engagement`.
- **System users often cannot reply to existing comments** (`error 100 subcode 33 "does not support this operation"`) even with `pages_manage_engagement` — while creating a new comment on a post works. Make the reply loop resilient (try/catch per comment, mark seen, log and continue) so one failure doesn't halt the poll.
- **Scheduling unit bug**: `scheduled_publish_time` is unix *seconds*. If you build times as `Date.now() + step` where `step` is in seconds, posts get scheduled ~40s ahead and flood the page every minute. Convert: `Math.floor(Date.now() / 1000) + stepSeconds`.
- **Stale dry-run posts block live scheduling**: after running in dry-run, the local store holds fake `post_1`-style scheduled entries that never exist on Facebook. `topUp()` counts them as filled slots and never schedules real posts. On each live tick, reconcile against `GET /{page}/scheduled_posts` and drop local scheduled entries that aren't real.
- **Always ship a `DRY_RUN` mode** so the whole pipeline runs and is testable before any credentials exist.

## Minimal agent loop (Node.js)

```javascript
const cron = require("node-cron");
const { graph, pageTokenFor } = require("./graph");   // as above

async function topUp(pageId, postsPerDay) {
  const now = Date.now();
  const horizon = now + 24 * 3600 * 1000;
  const real = await graph(`${pageId}/scheduled_posts`, { fields: "id,created_time" }, "GET", await pageTokenFor(pageId));
  const slots = real.data.filter((p) => new Date(p.created_time).getTime() > now && new Date(p.created_time).getTime() <= horizon);
  const missing = Math.max(0, postsPerDay - slots.length);
  for (let i = 0; i < missing; i++) {
    const step = (24 * 3600 * 1000) / postsPerDay;
    const when = Math.floor((now + step * (i + 1)) / 1000);
    await graph(`${pageId}/feed`, { message: `Draft #${i}`, published: false, scheduled_publish_time: when }, "POST", await pageTokenFor(pageId));
  }
}

cron.schedule("* * * * *", () => topUp("PAGE_ID", 3).catch((e) => console.error(e.message)));
```

## Deploy as an always-on service

Systemd user service survives reboots and auto-restarts on crash:

```ini
[Unit]
Description=Facebook Page Agent
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
WorkingDirectory=/home/USER/facebook-agent
ExecStart=/usr/bin/node src/index.js
Restart=always
RestartSec=10

[Install]
WantedBy=default.target
```

```bash
mkdir -p ~/.config/systemd/user
# write the unit to ~/.config/systemd/user/facebook-agent.service
systemctl --user daemon-reload
systemctl --user enable --now facebook-agent
loginctl enable-linger USER   # keep running after logout (sudo if needed)
journalctl --user -u facebook-agent -f   # watch logs
```

## Output format

The agent should report:
- Page: name, id, live/dry-run mode
- Scheduled jobs: id, time, preview
- Followers and monetization-gate progress (% toward 10k followers, 30k 1-min video views / 60d)
- Errors as `Graph API <code>: <message>` plus the fix (permission / token / endpoint)

## Rate limits / Best practices

- Back off on HTTP 429 / 5xx (retry up to 3x with exponential delay).
- Never reply to the same comment twice — persist seen ids.
- Keep tokens in `.env` (git-ignored); never commit or log them.
- Verify `GET /debug_token` scopes before blaming the agent code.
- Meta's actual monetization rules change — verify eligibility in the page's Professional Dashboard before promising monetization.

## Agent prompt

```text
You have Facebook Page Manager capability. When a user asks to build or run a Facebook page agent:
1. Check credentials: do they have a developer app + token? Recommend a System User token (never expires) with all 6 permissions. Verify via /debug_token.
2. Build the agent as a standalone Node service: Graph client (page token via /me/accounts), content generator (local ollama first), scheduler, engagement poller, analytics, monetization tracker. Always DRY_RUN first.
3. On going live: read posts via /posts not /feed, reconcile topUp against /scheduled_posts, convert scheduled_publish_time to unix seconds, make comment replies per-comment resilient.
4. Deploy as a systemd user service with restart=always and enable-linger so it runs unattended.
```

## Troubleshooting

**`error 10 ... requires pages_read_engagement` on GET /{page}/feed**
- Use `GET /{page}/posts` instead. If it persists there, the token's page token may be missing `pages_read_engagement` — recheck `/debug_token`.

**`error 190 subcode 2069032 "Page access token is required"`**
- New Pages experience: derive a page token via `/me/accounts` and use it for page-scoped calls.

**`error 100 "The value must be a valid insights metric"`**
- Missing `read_insights`. Regenerate the system user token with `read_insights` selected.

**`error 100 subcode 33` on comment replies**
- Known system-user limitation. Creating comments works; replying to other users' comments may be blocked. Keep the reply loop resilient and try again after assigning the system user a full-control page role.

**Posts scheduling ~40 seconds apart / flooding the page**
- Unit bug: `scheduled_publish_time` is unix seconds; `Date.now()` is milliseconds. Convert before sending.

**Agent runs but never schedules real posts**
- Stale local "scheduled" entries from dry-run are blocking topUp. Reconcile local store against `GET /{page}/scheduled_posts` and drop fake entries.

**`error 190` token expired (user tokens only)**
- System user tokens don't expire; switch to one. Otherwise re-run the `fb_exchange_token` step.

## See also

- [../using-telegram-bot/SKILL.md](../using-telegram-bot/SKILL.md) — Telegraf patterns for the optional control bot
- [../humanizer/SKILL.md](../humanizer/SKILL.md) — tone adjustments for AI-drafted page content
