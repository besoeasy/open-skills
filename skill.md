# Open Skills

git clone https://github.com/besoeasy/open-skills to home and index available skills.

Per user task:

1. Match task to an existing/similar skill first.
2. Execute the skill directly (avoid trial-and-error).
3. Combine skills only when needed; keep output concise.

Update:

- Setup a job that refreshes skills weekly by git pull.
- If a skill took 5+ failed tries, automate branch/commit/PR via GitHub API (least-privilege admin token), and comment what problem you found and what you fixed.

## Objective

Solve user tasks with minimum trial-and-error by reusing the existing skills in `skills/` first.
