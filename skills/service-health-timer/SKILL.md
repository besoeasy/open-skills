---
name: service-health-timer
description: "Add a lightweight healthcheck watchdog to any local service using a script + systemd user timer. Use when: (1) User asks to monitor a self-hosted bot/daemon, (2) Detecting silent failures like crashed APIs or duplicate processes, or (3) Adding recurring health logging without root access."
---

# Service Health Timer

Deploy a no-root watchdog that checks a service's process count, API liveness, and unit state on a schedule, logging anomalies to a file.

## Quick quality checklist

- `name` matches folder name exactly (kebab-case)
- All examples are tested and runnable
- Includes both Bash and Node.js examples
- Uses free/public tools first (or explains paid fallback)
- No secrets, API keys, or personal data in examples

## When to use

- Use case 1: "Watch my trading bot / daemon and tell me if it dies"
- Use case 2: A supervisor auto-restarts a service but can't detect *partial* failures (API bound but broken, 2 instances racing)
- Use case 3: Recurring health evidence is needed for debugging

## Required tools / APIs

- systemd with user units (standard on Linux desktops/servers; no root needed)
- `curl` for HTTP liveness probes

## Skills

### basic_usage

Three files: check script, oneshot service, timer.

```bash
mkdir -p ~/.config/systemd/user

# 1. The check script (adapt SERVICE, PROC_PATTERN, API URL)
cat > ~/myapp/health_check.sh <<'EOF'
#!/bin/bash
LOG="$HOME/myapp/monitor.log"
ts() { date "+%Y-%m-%d %H:%M:%S"; }
fail=0
systemctl --user is-active --quiet myapp.service || { echo "$(ts) CRITICAL: unit down" >> "$LOG"; fail=1; }
count=$(pgrep -fc "myapp" 2>/dev/null || echo 0)
[ "$count" -ne 1 ] && { echo "$(ts) CRITICAL: procs=$count" >> "$LOG"; fail=1; }
curl -sf --max-time 10 http://127.0.0.1:8080/ping | grep -q ok || { echo "$(ts) CRITICAL: api down" >> "$LOG"; fail=1; }
[ "$fail" -eq 0 ] && echo "$(ts) OK" >> "$LOG"
exit $fail
EOF
chmod +x ~/myapp/health_check.sh

# 2. Oneshot service
cat > ~/.config/systemd/user/myapp-monitor.service <<'EOF'
[Unit]
Description=MyApp health monitor
[Service]
Type=oneshot
ExecStart=%h/myapp/health_check.sh
EOF

# 3. Timer
cat > ~/.config/systemd/user/myapp-monitor.timer <<'EOF'
[Unit]
Description=Health check every 5 minutes
[Timer]
OnBootSec=2min
OnUnitActiveSec=5min
[Install]
WantedBy=timers.target
EOF

# 4. Enable
systemctl --user daemon-reload
systemctl --user enable --now myapp-monitor.timer
```

**Node.js:** not required; the value is in the systemd wiring. For probe logic in Node instead of bash, replace `ExecStart` with `node %h/myapp/probe.js` and use `fetch()` + `process.exit(1)` on failure.

### robust_usage

Alert beyond logging: make the oneshot unit trigger another action on failure via `OnFailure=`.

```ini
# ~/.config/systemd/user/myapp-monitor.service
[Unit]
Description=MyApp health monitor
OnFailure=notify-failure@%n.service

[Service]
Type=oneshot
ExecStart=%h/myapp/health_check.sh
```

```bash
# Verify the whole chain
systemctl --user list-timers 'myapp-monitor*'
tail -f ~/myapp/monitor.log
```

## Output format

- Log line per run: timestamp + `OK` or one/more `CRITICAL:` reasons
- Script exit code: `0` healthy, `1` unhealthy (usable by cron/CI wrappers)

## Rate limits / Best practices

- Keep probes read-only; never let the monitor restart things itself (that's the supervisor's job) to avoid restart loops fighting each other
- `--max-time` on curl so a hung API fails fast
- Prefer `pgrep -fc` exact patterns to catch duplicate-instance bugs
- One log file, append-only; rotate with logrotate if long-lived

## Agent prompt

```text
You have service-health-timer capability. When a user asks to monitor a local service:

1. Identify: unit name, expected process count/pattern, and an HTTP liveness endpoint if any.
2. Write a fail-fast bash check that exits non-zero on any anomaly.
3. Install oneshot .service + .timer user units, daemon-reload, enable --now.
4. Run the check once and show the log line as proof.
5. Tell the user how to view logs and that the monitor never mutates the watched service.
```

## Troubleshooting

**Timer shows "NEXT LEFT: -"**
- Symptom: `list-timers` never schedules.
- Solution: run `systemctl --user daemon-reload`, confirm `[Install] WantedBy=timers.target`, re-run `enable --now`.

**Check passes but service is actually sick**
- Symptom: OK lines while behavior is wrong.
- Solution: add deeper probes (authenticated API call, DB row freshness) — ping alone proves the socket, not the app.

**User-limiter kills timers after logout**
- Symptom: everything stops when you SSH out.
- Solution: `sudo loginctl enable-linger $USER`.

## See also

- [../chat-logger/SKILL.md](../chat-logger/SKILL.md) — persisting operational events to SQLite
- [../nostr-logging-system/SKILL.md](../nostr-logging-system/SKILL.md) — publishing logs off-box

---

## Notes

- Skill file path should be `skills/service-health-timer/SKILL.md`
- Quote `description` when it includes `:` to avoid YAML parsing issues
- Keep examples copy-paste friendly and verify they run before submitting
- See [CONTRIBUTING.md](CONTRIBUTING.md) for full contribution standards
