---
name: zeroclaw-telegram-channel
description: "Diagnose and repair the ZeroClaw Telegram channel so the agent answers you from Telegram. Use when (1) Telegram pairing/bind codes loop forever, (2) the gateway websocket is in a restart loop, (3) channel doctor is healthy but messages/send fail with 401, or (4) you need to bind your Telegram identity to a channel alias."
---

# ZeroClaw Telegram Channel

Gets the ZeroClaw agent reachable from Telegram: healthy channel, stable gateway, and your identity bound to the channel alias so the agent responds without endless `/bind` codes.

## When to use

- The agent keeps printing `Telegram pairing required. One-time bind code: NNNN` and never settles.
- `zeroclaw gateway` component shows `Address already in use (os error 98)` with a high restart count.
- `zeroclaw channel doctor` reports healthy, but `zeroclaw channel send` fails with `401 Unauthorized`.
- You just added a bot token and want to bind your own Telegram account (username or numeric user ID) to a channel alias.

## Required tools

- `zeroclaw` CLI (config-dir defaults to `~/.zeroclaw`)
- `systemctl --user` (daemon runs as a user systemd service)
- `ss` (socket inspection)

No external APIs required.

## Skills

### 1. Survey the current state

```bash
zeroclaw status                       # version, workspace, service, channels
zeroclaw channel doctor               # per-channel health
zeroclaw channel list                 # configured channels + aliases
cat ~/.zeroclaw/state/daemon_state.json | python3 -c \
  "import json,sys; d=json.load(sys.stdin); [print(k,'->',c.get('status'),(c.get('last_error') or '')[:60]) for k,c in d['components'].items()]"
ps aux | grep "zeroclaw daemon" | grep -v grep   # count daemons — must be exactly ONE
ss -tlnp | grep 42617                  # who owns the gateway port
```

### 2. Kill duplicate daemons (the #1 root cause)

Two daemons (e.g. a homebrew install and a cargo install) both polling the same bot token and fighting over the gateway port break pairing and Telegram getUpdates. Keep the daemon whose workspace matches the CLI, disable the other:

```bash
systemctl --user list-units | grep -i zeroclaw
systemctl --user stop homebrew.zeroclaw.service
systemctl --user disable homebrew.zeroclaw.service
systemctl --user restart zeroclaw.service
sleep 6
ps aux | grep "zeroclaw daemon" | grep -v grep   # expect exactly one
ss -tlnp | grep 42617                            # expected to be held by the surviving PID
```

### 3. Fix the gateway / wss port collision

If the `gateway` component binds `127.0.0.1:42617` while the `wss` (TLS websocket) component errors `binding WSS listener on 0.0.0.0:42617`, they collide on the same port. Move wss off the gateway port:

```bash
zeroclaw config set wss.port 42618
systemctl --user restart zeroclaw.service
sleep 6
cat ~/.zeroclaw/state/daemon_state.json | python3 -c \
  "import json,sys; d=json.load(sys.stdin); [print(k,'->',c.get('status')) for k,c in d['components'].items()]"
```

All components should report `ok` (including `gateway` and `wss`).

### 4. Bind your Telegram identity to the channel alias

`zeroclaw channel bind-telegram <identity> --alias <alias>` binds a Telegram username (no `@`) or numeric user ID into the channel allowlist. The alias must match `channels.telegram.<alias>` used by the agent — otherwise the agent keeps asking for approval.

```bash
# Find the alias the agent uses
grep -A3 "^\[agents\." ~/.zeroclaw/config.toml | grep channels

# Bind yourself (idempotent — re-running is safe)
zeroclaw channel bind-telegram 123456789 --alias telegram1
# → "✅ Telegram identity already bound to telegram.telegram1: 123456789"
```

### 5. Verify end-to-end

```bash
zeroclaw channel doctor
zeroclaw channel bind-telegram <your-id> --alias telegram1   # confirm bound
journalctl --user -u zeroclaw.service --no-pager | grep -iE "telegram|bind|pair" | tail
```

Then message the bot from Telegram; the agent should reply without a bind prompt.

## Troubleshooting

**Symptom: `zeroclaw channel send --channel-id telegram` fails with 401 Unauthorized**
- Cause: the `--channel-id telegram` CLI path resolves to the `channels.telegram.default` alias. If that alias holds a stale/revoked bot token, sends 401 even though your real channel (`telegram1`) is healthy.
- Fix: ignore it, or remove the dead `[channels.telegram.default]` section from `~/.zeroclaw/config.toml` and restart the daemon. Send through the real alias instead.

**Symptom: gateway component `Address already in use`, restart count in the hundreds**
- Cause: a second zeroclaw daemon owns the port.
- Fix: identify with `ps aux | grep "zeroclaw daemon"`, stop/disable the duplicate service, restart the survivor.

**Symptom: `Telegram pairing required. One-time bind code: NNNN` loops forever**
- Cause: your identity isn't in the channel allowlist (and/or two daemons are fighting the same bot token).
- Fix: run `zeroclaw channel bind-telegram <your-id> --alias telegram1`, ensure exactly one daemon, then restart it.

## Agent prompt

```text
You have ZeroClaw Telegram channel repair capability. When a user reports the ZeroClaw Telegram channel not working:

1. Run `zeroclaw channel doctor`, `zeroclaw channel list`, and `zeroclaw status` first.
2. Count running daemons with `ps aux | grep "zeroclaw daemon"`; if more than one, disable the duplicate systemd user service and restart the survivor.
3. Inspect `~/.zeroclaw/state/daemon_state.json`; if `gateway`/`wss` conflict on port 42617, move `wss.port` and restart.
4. Confirm the user's Telegram identity is bound: `zeroclaw channel bind-telegram <id> --alias telegram1`.
5. Verify all components are `ok` and report the final state.

Never modify or decrypt bot tokens. Never bind an identity the user hasn't confirmed as their own.
```

## See also

- [../using-telegram-bot/SKILL.md](../using-telegram-bot/SKILL.md) — Building standalone Telegram bots in Node.js with Telegraf (different use case: custom bots, not the ZeroClaw channel)
