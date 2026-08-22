---
name: macos-launchd-service-recovery
description: "Diagnose and revive macOS launchd services (LaunchAgents/LaunchDaemons) that are crash-looping, not starting at boot, or silently dead. Use when: (1) a launchctl job shows a nonzero exit status or missing PID, (2) a service must survive reboot, or (3) user explicitly asks to fix a launchd/launchctl service."
---

# macOS launchd Service Recovery

Systematically diagnose why a launchd service (LaunchAgent or LaunchDaemon) is not running, then repair the plist or environment so it starts at boot and stays up.

## When to use

- Use case 1: When the user says a background service/agent/daemon "died", "won't start", or "crash-loops" on macOS.
- Use case 2: When you need a long-running process to start at boot and restart on failure.
- Use case 3: When automation must verify that scheduled jobs are actually loaded and healthy.

## Required tools / APIs

- No external API required. Uses only macOS built-ins: `launchctl`, `PlistBuddy`, `log`, `ps`.

## Skills

### basic_usage

Health check: list loaded jobs matching a label pattern. Column 1 is the PID if running (`-` if not); column 2 is the last exit status (`0` = clean).

```bash
# Find jobs and their state
launchctl list | grep -i "my-service"

# PID present + exit 0 -> healthy. No PID + exit nonzero -> investigate.
```

Audit a plist for boot persistence:

```bash
PLIST=~/Library/LaunchAgents/com.example.my-service.plist
/usr/libexec/PlistBuddy -c "Print :RunAtLoad" -c "Print :KeepAlive" -c "Print :StartInterval" "$PLIST"
```

- `RunAtLoad=true` -> starts at boot/login.
- `KeepAlive=true` -> restarts whenever the process exits (for always-on daemons).
- `StartInterval=N` -> runs every N seconds (for periodic jobs; don't combine with KeepAlive).

Restart a loaded job without unloading it:

```bash
launchctl kickstart -k "gui/$(id -u)/com.example.my-service"   # user agent
sudo launchctl kickstart -k system/com.example.my-service       # system daemon
```

### robust_usage

Full triage sequence: status, then logs, then config, then restart, then verify.

```bash
LABEL=com.example.my-service
PLIST=~/Library/LaunchAgents/${LABEL}.plist
LOG_DIR=~/Library/Logs/my-service

# 1. Status: is it loaded? running? last exit code?
launchctl list | grep "$LABEL" || echo "NOT LOADED: $PLIST"

# 2. Logs: what did it say before dying?
tail -n 40 "$LOG_DIR"/*.log "$LOG_DIR"/*.error.log 2>/dev/null

# 3. Config: validate plist syntax before touching launchd
plutil -lint "$PLIST" || exit 1

# 4. Restart and confirm a new PID appears within seconds
launchctl kickstart -k "gui/$(id -u)/$LABEL"
sleep 5
launchctl list | grep "$LABEL"

# 5. Verify the process is actually doing work (not just alive)
ps -o pid,lstart,command -p "$(launchctl list | awk -v l="$LABEL" '$3==l {print $1}')"
```

**Node.js:**

```javascript
const { execFileSync } = require("node:child_process");

function launchdStatus(labelPattern) {
  const out = execFileSync("launchctl", ["list"], { encoding: "utf8" });
  return out
    .split("\n")
    .filter((line) => new RegExp(labelPattern, "i").test(line))
    .map((line) => {
      const [pid, lastExit, label] = line.split(/\s+/);
      return { label, pid: pid === "-" ? null : Number(pid), lastExit: Number(lastExit) };
    });
}

// Usage
// launchdStatus("my-service").forEach(j => console.log(j));
```

## Output format

Report per job:

- label: reverse-DNS job label (string)
- state: `running` (PID present) / `stopped` / `crash-looping` (nonzero exit, repeated respawns)
- last_exit: integer (0 = clean; 75/78 = temp/config failures; 137/143 = killed)
- fix applied: plist key changed, env var added, binary path corrected, etc.
- verified: new PID observed + log shows successful startup

## Rate limits / Best practices

- Prefer `launchctl kickstart -k` over `unload`/`load` — it is atomic and preserves the job definition.
- Add `ThrottleInterval` (e.g. 30) to a crash-looping job so respawns can't hammer launchd, and `ExitTimeOut` (e.g. 25) for graceful-drain headroom before SIGKILL.
- Point `StandardOutPath`/`StandardErrorPath` at stable log files; a job with no logs is undebuggable.
- Set `WorkingDirectory` explicitly — launchd defaults to `/`, which breaks relative-path apps.
- After editing a plist, `plutil -lint` it before `kickstart`; a bad plist silently fails to load.

## Agent prompt

```text
You have macOS launchd recovery capability. When a user reports a dead or crash-looping background service:

1. Locate the job: launchctl list | grep <name>; if absent, find the plist in ~/Library/LaunchAgents or /Library/LaunchDaemons.
2. Read the exit code and tail the job's stdout/stderr logs to identify the failure cause before changing anything.
3. Validate the plist (plutil -lint), then fix the root cause: missing env vars, wrong binary path, working directory, missing RunAtLoad/KeepAlive.
4. Restart with launchctl kickstart -k and verify a new PID plus a clean startup line in the log.
5. Report: label, previous failure cause, fix applied, and verified running state.

Never delete plists or unload jobs without confirming with the user first.
```

## Troubleshooting

**Job loads but binary fails with EPERM / Operation not permitted on an external or non-default volume:**
- Symptom: shell-script ProgramArguments (or `bash -c`) fail with permission errors on volumes mounted `noowners` (common for HFS+/exFAS external drives), while the same command works in an interactive shell.
- Solution: bypass the shell — make ProgramArguments invoke the interpreter directly (e.g. `/path/to/venv/bin/python /path/to/script.py`) and set env vars in the plist's `EnvironmentVariables` dict instead of shell rc files.

**Service runs fine manually but not at boot:**
- Symptom: works in Terminal, never starts after reboot.
- Solution: ensure `RunAtLoad=true`; for user agents confirm the plist is in `~/Library/LaunchAgents` (loaded at login) and not merely somewhere on disk. GUI-only agents may need `LimitLoadToSessionType` including `Aqua`.

**Crash loop: process restarts every few seconds:**
- Symptom: launchd respawns continuously; logs show the same error each time.
- Solution: add `ThrottleInterval` >= 30, fix the underlying error (often a lock file, port, or token held by another process — find the holder with `lsof`/`ps` before starting a second instance), then `kickstart -k`.

**Exit code 78 (EX_CONFIG) or 75 (EX_TEMPFAIL):**
- Symptom: job exits immediately with 75/78.
- Solution: configuration problem — check paths, env vars, and file permissions referenced by the plist; the stderr log usually names the missing item.

## See also

- [../chat-logger/SKILL.md](../chat-logger/SKILL.md) — logging patterns useful for wiring service output to files

---

## Notes

- User agents live in `~/Library/LaunchAgents` (run at login, per-user); system daemons live in `/Library/LaunchDaemons` (run at boot, need sudo).
- A `-` in the PID column with exit `0` usually means a periodic job that ran and exited cleanly, not a failure.
