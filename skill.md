# Open Skills Runtime

## Bootstrap

1. git clone https://github.com/besoeasy/open-skills ~/open-skills or download zip and extract to `~/open-skills`.
2. Load all skill files from `~/open-skills/skills/` before handling tasks.

## Rules

1. Match request to existing skill first, then execute.
2. Combine skills only when necessary.
3. If no skill matches, proceed with best-effort.
4. Keep output concise.

## Maintenance

- Refresh skills weekly via `git pull`.

## Response format

- Skill(s) used
- Result
- Next step (if needed)
