---
name: analyze-android-apk
description: "Decompile and analyze Android APKs (manifest, permissions, activities, SDKs, endpoints) using portable Java + jadx with no root required. Use when: (1) User shares an .apk or extracted APK folder and asks what it does, (2) Security/privacy review of an Android app, or (3) Inspecting app code, permissions, or network endpoints."
---

# Analyze Android APK

Decompile any Android APK into readable Java sources and a decoded manifest, then extract identity, permissions, components, bundled SDKs, and hardcoded endpoints. Works fully in user space — no root and no sudo needed.

## Quick quality checklist

- `name` matches folder name exactly (kebab-case)
- All examples are tested and runnable
- Uses free/public tools only
- No secrets, API keys, or personal data in examples

## When to use

- Use case 1: User provides an `.apk` file (or an already-unzipped APK directory) and wants to know what the app does.
- Use case 2: Privacy/security review: permissions, trackers, plaintext endpoints, self-update mechanisms.
- Use case 3: Locating specific logic inside an app (pairing protocol, update check, license validation).

## Required tools / APIs

No external API required. Two portable tools installed to `~/.local/opt` (no sudo):

- **Temurin JRE 21** — runtime for jadx
- **jadx** — DEX/APK decompiler producing Java sources + decoded resources

Install options:

```bash
# Portable install (works without sudo, Linux x86_64)
mkdir -p ~/.local/opt /tmp/apk-work
curl -sL -o /tmp/apk-work/jre.tar.gz \
  "https://api.adoptium.net/v3/binary/latest/21/ga/linux/x64/jre/hotspot/normal/eclipse"
tar -xzf /tmp/apk-work/jre.tar.gz -C ~/.local/opt

curl -sL -o /tmp/apk-work/jadx.zip \
  "https://github.com/skylot/jadx/releases/download/v1.5.4/jadx-1.5.4.zip"
mkdir -p ~/.local/opt/jadx && unzip -qo /tmp/apk-work/jadx.zip -d ~/.local/opt/jadx
chmod +x ~/.local/opt/jadx/bin/*

# System package alternative (requires sudo)
sudo apt-get install -y default-jre-headless   # then still install jadx from GitHub releases
```

## Skills

### basic_usage

Shortest reliable path: set env, decompile, read manifest.

```bash
export JAVA_HOME=$(echo ~/.local/opt/jdk-*-jre | tail -1)
export PATH="$JAVA_HOME/bin:$HOME/.local/opt/jadx/bin:$PATH"

jadx --version   # sanity check, expect 1.5.x

# Decompile (exit code 3 = finished with minor errors; normal for large apps)
jadx -j 4 -d /tmp/apk-work/out /path/to/app.apk > /tmp/apk-work/jadx.log 2>&1

cat /tmp/apk-work/out/resources/AndroidManifest.xml   # now readable XML
```

### robust_usage

Full analysis pass with error handling and structured extraction.

```bash
#!/usr/bin/env bash
set -euo pipefail
APK="$1"; OUT="${2:-/tmp/apk-work/out}"
[ -f "$APK" ] || { echo "Error: $APK not found" >&2; exit 1; }

export JAVA_HOME=$(echo ~/.local/opt/jdk-*-jre | tail -1)
export PATH="$JAVA_HOME/bin:$HOME/.local/opt/jadx/bin:$PATH"

if ! jadx -j 4 -d "$OUT" "$APK" >"$OUT.log" 2>&1; then
  echo "Note: jadx exited nonzero; check $OUT.log (minor errors are common)" >&2
fi

M="$OUT/resources/AndroidManifest.xml"
echo "== Identity =="
grep -oE 'package="[^"]+"|versionName="[^"]*"|versionCode="[^"]*"' "$M"
echo "== Permissions =="
grep -oE 'android:name="android.permission.[A-Z_]+"' "$M" | sort -u
echo "== Activities =="
grep -oE 'android:name="[^"]*Activity"' "$M" | sort -u
echo "== Hardcoded URLs =="
grep -rEoh 'https?://[a-zA-Z0-9./_?=&-]+' "$OUT/sources" 2>/dev/null | sort | uniq -c | sort -rn | head -20
```

Useful follow-up greps:

```bash
# Bundled third-party SDKs (trackers, ads, vendor libs)
ls "$OUT/sources/com"

# Find where a URL/string is used
grep -rl "example.endpoint.com" "$OUT/sources"

# App strings (app name, messages)
grep -oE '<string name="app_name">[^<]*' "$OUT/resources/res/values/strings.xml"
```

## Output format

Report these fields to the user:

- `package`: application ID (e.g., `com.vendor.app`)
- `version`: versionName + versionCode
- `purpose`: 1-3 sentence description of what the app does, inferred from activity/service names
- `permissions_notable`: dangerous/system-level permissions worth flagging
- `sdks`: third-party SDKs found under `sources/com/*` (analytics, ads, web kernels)
- `endpoints`: table of hardcoded URLs/IPs and likely purpose
- `risk_notes`: plaintext HTTP, hardcoded IPs, self-update/install permissions, telemetry SDKs
- Error shape: if decompile fails entirely, quote last lines of the jadx log + suggest `--no-res` retry

## Rate limits / Best practices

- Only download tools once; reuse `~/.local/opt` installs across sessions.
- Decompile to `/tmp`, copy only what is worth keeping to user-visible folders (sources can exceed 100 MB).
- Exit code 3 from jadx means partial errors — inspect the log before declaring failure.
- Prefer analyzing the original `.apk`; an extracted APK folder lacks central directory metadata but its `classes*.dex` can be passed to jadx individually.
- Respect copyright: analysis is fine; do not redistribute decompiled vendor code.

## Agent prompt

```text
You have analyze-android-apk capability. When a user asks what an APK does or to audit it:

1. Verify the input path exists (.apk file or extracted folder).
2. Ensure portable JRE + jadx are installed under ~/.local/opt; install if missing.
3. Run jadx decompilation to a temp output dir; tolerate exit code 3.
4. Extract identity, permissions, activities, SDK list, and hardcoded URLs using the greps in this skill.
5. Return the report in the defined Output format, flagging privacy/security risks.

Always prefer the portable no-sudo install over system packages.
```

## Troubleshooting

**`jadx: Permission denied`:**
- Symptom: launcher script not executable after unzip.
- Solution: `chmod +x ~/.local/opt/jadx/bin/*`

**`java: command not found` when running jadx:**
- Symptom: JAVA_HOME not exported in current shell.
- Solution: re-export `JAVA_HOME=$(echo ~/.local/opt/jdk-*-jre | tail -1)` and prepend `$JAVA_HOME/bin` to PATH.

**Manifest unreadable / binary garbage:**
- Symptom: reading `AndroidManifest.xml` from the raw APK zip shows binary AXML.
- Solution: use the jadx-decoded copy at `<out>/resources/AndroidManifest.xml`.

**Out of memory on huge APKs:**
- Symptom: jadx crashes or hangs near end of processing.
- Solution: add `JAVA_OPTS="-Xmx4g"` before jadx, or run with `--no-res` if resources are not needed.

## See also

- [../using-web-scraping/SKILL.md](../using-web-scraping/SKILL.md) — fetch vendor sites referenced by an app for extra context
- [../phone-specs-scraper/SKILL.md](../phone-specs-scraper/SKILL.md) — research the devices an app targets
