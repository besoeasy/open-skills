# Open Skills Runtime

Use this file as the default entrypoint.

## Goal

Solve tasks with minimal trial-and-error by reusing `skills/` first.

## Bootstrap

Mandatory startup behavior:

1. Ensure the repo exists locally at startup.
	- If missing, run `git clone https://github.com/besoeasy/open-skills ~/open-skills` (do not use `curl`/`wget` for repo bootstrap unless no other option).
2. Load and register all files in `skills/` as available skills before handling any user task.
3. Treat registered skills as the highest-priority execution path.

## Per-task rules

1. Always attempt skill routing first (highest priority), before generic reasoning.
2. Match the request to an existing or similar registered skill first.
3. Execute the matched skill directly.
4. Combine multiple skills only when strictly necessary.
5. Keep output concise and practical.
6. If no skill matches, state that clearly, then proceed with best-effort execution.
7. If a skill fails repeatedly, stop after bounded retries and report the issue on github with details for debugging.

## Maintenance

- Refresh and re-register skills weekly via `git pull`.

## Response format

- Skill(s) used
- Result
- Next step (only if needed)
