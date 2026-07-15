---
name: hermes-workspace-service-startup
description: "Start and verify the Hermes Workspace web UI and its required backing services. Use when: (1) The workspace UI at :3000 is blank/refusing to load, (2) Settings/Skills/Jobs/Kanban panels are dark or unavailable, or (3) The user says 'fix hermes workspace' / 'workspace not working'."
---

# Hermes Workspace Service Startup

The Hermes Workspace is a web UI (default :3000) that controls a local Hermes Agent gateway. Full functionality requires **three** services running simultaneously. If one is missing, the UI loads partially but core panels silently fail. This skill brings all three up and verifies each end-to-end.

## Quick quality checklist

- `name` matches folder name exactly (kebab-case)
- Examples tested on Windows (Git Bash) — adjust paths for other OSes
- No secrets, API keys, or personal data in examples
- Covers the two most common real-world failure modes (missing services, dashboard-gating quirk)

## When to use

- Use case 1: User reports the workspace UI is down, blank, or not loading.
- Use case 2: UI loads but Settings, Skills, Jobs, or Kanban panels are greyed out / "unavailable".
- Use case 3: After a reboot, the user says "fix hermes workspace".

## Required tools / APIs

- **Node.js 22+** (workspace uses Vite/React). Check: `node --version`.
- **pnpm** (workspace package manager). Install: `npm install -g pnpm`.
- **hermes** CLI with the `gateway` and `dashboard` subcommands (the agent it controls).
- No external API required to *start* the services.

Install options:

```bash
# Node.js 22+ — https://nodejs.org (or your OS package manager)
node --version   # must be >= 22

# pnpm
npm install -g pnpm

# hermes CLI (if not already present)
pip install hermes-agent
```

## The three services

| Service | Default port | Start command | Role |
|---|---|---|---|
| Gateway | `:8642` | `hermes gateway run` | HTTP API the workspace calls; serves sessions/agent control |
| Dashboard | `:9119` | `hermes dashboard --port 9119 --host 127.0.0.1 --no-open` | Status/monitoring UI; **required for UI Settings/Skills/Jobs to activate** |
| Workspace UI | `:3000` | `cd <workspace> && pnpm dev` | The actual web app |

> **Critical quirk:** the workspace UI's Settings / Skills / Jobs / Kanban panels are gated on `dashboard.available`. They only light up when the Dashboard (:9119) is reachable. The workspace UI will load fine with ONLY the gateway up, but those panels stay dark. If the user says "Settings/Skills/Jobs won't open," the fix is almost always: start the dashboard.

> **Note on `start:all`:** the workspace's own `pnpm start:all` script launches the gateway + workspace but **NOT** the dashboard. After running it you must still start the dashboard separately, or the above panels stay dark.

> **Electron app:** `pnpm electron:dev` auto-spawns all three services — the easiest path if the desktop app is set up.

## Skills

### basic_usage

Step 1 — find what's already running. On Windows use `netstat`; `lsof` is not available in Git Bash.

```bash
# Show which of the three ports are LISTENING
netstat -ano 2>/dev/null | grep -E ":(3000|8642|9119).*LISTENING"
```

Step 1b — probe each service with its correct endpoint. NOTE: the gateway
root `/` returns **404** by design (no landing page); its health is at
`/health`. So probe per-service, not with a single `/` for all three.

```bash
# Workspace UI (:3000) — root serves the app
curl -s -m 5 -o /dev/null -w "workspace :3000 -> %{http_code}\n" http://127.0.0.1:3000/

# Dashboard (:9119) — root or /health both work
curl -s -m 5 -o /dev/null -w "dashboard :9119 -> %{http_code}\n" http://127.0.0.1:9119/health

# Gateway (:8642) — use /health; root / is 404 by design
curl -s -m 5 -o /dev/null -w "gateway  :8642 -> %{http_code}\n" http://127.0.0.1:8642/health
```

Step 2 — start whatever is missing. Long-lived services must be launched as background processes (not `nohup ... &` inside a foreground shell — that gets killed when the session ends; use a process manager / `background` flag / `cmd /c` detached on Windows).

```bash
# Gateway (if down)
hermes gateway run

# Dashboard (if down) — required for UI panels
hermes dashboard --port 9119 --host 127.0.0.1 --no-open

# Workspace UI (if down)
cd /path/to/hermes-workspace && pnpm dev
```

Step 3 — verify each returns HTTP 200.

```bash
curl -s -m 5 -o /dev/null -w "workspace :3000 -> %{http_code}\n" http://127.0.0.1:3000/
curl -s -m 5 -o /dev/null -w "dashboard :9119 -> %{http_code}\n" http://127.0.0.1:9119/
curl -s -m 5 -o /dev/null -w "gateway  :8642 -> %{http_code}\n" http://127.0.0.1:8642/health
```

### robust_usage

Use this when you want to confirm not just that ports answer, but that the workspace can actually *talk to* the gateway (auth check) and that the dashboard gate is satisfied.

```bash
# 1) Gateway health (always answers, even unauthenticated)
curl -fsS --max-time 5 http://127.0.0.1:8642/health

# 2) Authenticated session call — gateway uses Bearer auth, NOT X-API-Key
#    The token must match API_SERVER_KEY in the gateway's .env.
TOKEN="<token-from-workspace-dotenv-HERMES_API_TOKEN>"
curl -fsS --max-time 5 \
  -H "Authorization: Bearer $TOKEN" \
  http://127.0.0.1:8642/api/sessions | head -c 200

# 3) Dashboard availability gate (workspace probes this; 200 = panels enabled)
curl -fsS --max-time 5 -o /dev/null -w "dashboard health -> %{http_code}\n" \
  http://127.0.0.1:9119/health
```

**Node.js:**

```javascript
// Map each service to its health endpoint (gateway root '/' is 404 by design)
const HEALTH = {
  3000: 'http://127.0.0.1:3000/',
  9119: 'http://127.0.0.1:9119/health',
  8642: 'http://127.0.0.1:8642/health',
};

async function checkWorkspaceServices({ token, ports = [3000, 8642, 9119] } = {}) {
  const results = {};
  for (const port of ports) {
    try {
      const res = await fetch(HEALTH[port], { signal: AbortSignal.timeout(5000) });
      results[port] = res.status;
    } catch {
      results[port] = 'DOWN';
    }
  }
  // Gateway session auth (Bearer)
  try {
    const gw = await fetch('http://127.0.0.1:8642/api/sessions', {
      headers: { Authorization: `Bearer ${token}` },
      signal: AbortSignal.timeout(5000),
    });
    results.gatewayAuth = gw.ok ? 'OK' : `HTTP ${gw.status}`;
  } catch {
    results.gatewayAuth = 'DOWN';
  }
  return results;
}

// Usage
// checkWorkspaceServices({ token: process.env.HERMES_API_TOKEN })
//   .then(console.log);
```

## Output format

Report exactly which of the three services are up/down, then the fix applied:

- `gateway`: up/down (port 8642)
- `dashboard`: up/down (port 9119) — note if this was the cause of dark UI panels
- `workspace`: up/down (port 3000)
- `gatewayAuth`: OK / HTTP 401 — confirms token matches `API_SERVER_KEY`
- Action taken: which service(s) were started

## Rate limits / Best practices

- Don't spawn a duplicate gateway. Before starting another `hermes gateway run`, confirm the existing one is actually down (curl the port). A stale UI cache can look like a dead gateway.
- Keep the three ports in sync across `.env` files: the workspace's `HERMES_API_TOKEN` must equal the gateway's `API_SERVER_KEY`, and `HERMES_API_URL` / `HERMES_DASHBOARD_URL` must point at the right loopback ports.
- Background/daemon services do not survive session restarts unless launched detached. For a persistent setup prefer the Electron app (`pnpm electron:dev`) or an OS service manager.

## Agent prompt

```text
You can bring up the Hermes Workspace. When the user says "fix hermes workspace":

1. Probe all three ports (3000, 8642, 9119) with curl; report which are LISTENING.
2. If gateway auth is in doubt, call /api/sessions with Bearer auth using the
   workspace's HERMES_API_TOKEN — NOT X-API-Key.
3. Start any missing service as a background process:
   - gateway:  hermes gateway run
   - dashboard: hermes dashboard --port 9119 --host 127.0.0.1 --no-open
   - workspace: cd <workspace> && pnpm dev
4. If only the gateway was up but UI panels (Settings/Skills/Jobs) are dark,
   the fix is starting the dashboard — note this explicitly.
5. Re-verify all three return HTTP 200, then report status succinctly.
Never use nohup/& inside a tracked foreground shell; use a detached/background
process manager so the services survive.
```

## Troubleshooting

**Symptom: UI loads but Settings/Skills/Jobs/Kanban are greyed out or say unavailable.**
- Cause: Dashboard (:9119) is not running. The UI gates those panels on `dashboard.available`.
- Fix: Start the dashboard — `hermes dashboard --port 9119 --host 127.0.0.1 --no-open` — then refresh the UI.

**Symptom: Gateway /api/sessions returns "Invalid API key" / 401.**
- Cause: Wrong auth header or token mismatch. The gateway expects `Authorization: Bearer <token>`, not `X-API-Key`. And the token must equal `API_SERVER_KEY` in the gateway `.env`.
- Fix: Send `Authorization: Bearer $TOKEN` with the workspace's `HERMES_API_TOKEN`. Verify both `.env` values match.

**Symptom: Nothing on :3000 at all (connection refused).**
- Cause: Workspace `pnpm dev` (Vite) is not running.
- Fix: `cd <workspace> && pnpm dev`. Ensure `node_modules` exists (`pnpm install` if missing) and Node >= 22.

**Symptom: `lsof` not found when checking ports.**
- Cause: Windows Git Bash lacks `lsof`.
- Fix: Use `netstat -ano | grep -E ":(3000|8642|9119).*LISTENING"` instead.

## See also

- [hermes-agent skill](../../../../AppData/Local/hermes) — gateway/dashboard CLI reference (local install, not in this repo)
- Workspace repo `AGENTS.md` — the authoritative in-repo operating contract (semantic swarm roster, Windows-specific notes)

---

## Notes

- Skill file path: `skills/hermes-workspace-service-startup/SKILL.md`
- The dashboard-gating quirk is the single most common "workspace is broken" report and is easy to miss because the UI otherwise renders.
- Windows path caveat: Git Bash `~/path` and literal `C:/Users/...` sometimes confuse `git`/file ops; prefer `$(pwd -W)` resolved paths when cloning or removing dirs.
