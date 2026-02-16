---
name: skill-name-kebab-case
description: "One-line description of what this skill does and why it matters. Use when: (1) First use case, (2) Second use case, (3) Third use case, or (4) User explicitly asks."
---

# Skill Title (Descriptive)

Brief overview of what this skill enables. Explain the problem it solves in 1-2 sentences.

## When to use

- Use case 1: When the user asks to...
- Use case 2: When you need to...
- Use case 3: For automation pipelines that...

## Required tools / APIs

- No external API required (if applicable)
- Tool 1: Brief description and install command
- Tool 2: Brief description and install command

Install options:

```bash
# Ubuntu/Debian
sudo apt-get install -y package-name

# macOS
brew install package-name

# Node.js
npm install package-name
```

## Skills

### skill_name_bash

Brief description of this specific skill implementation.

```bash
# Example 1: Basic usage
curl -s "https://api.example.com/endpoint?param=value" | jq '.field'

# Example 2: With options
command --option1 value1 --option2 value2 "input data"

# Example 3: Save to file
curl -s "https://api.example.com/data" > output.json
```

**Node.js:**

```javascript
async function skillName(param) {
  const res = await fetch(`https://api.example.com/endpoint?param=${param}`);
  const data = await res.json();
  return data.result;
}

// Usage
// skillName('value').then(console.log);
```

### skill_name_advanced

More complex variation with error handling, retries, etc.

```bash
# Bash with error handling
if ! response=$(curl -fsS --max-time 10 "https://api.example.com" 2>&1); then
  echo "Error: $response" >&2
  exit 1
fi
echo "$response"
```

**Node.js:**

```javascript
async function skillNameAdvanced(options = {}) {
  const { param1, timeout = 10000 } = options;
  
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), timeout);
  
  try {
    const res = await fetch(`https://api.example.com/endpoint`, {
      signal: controller.signal
    });
    clearTimeout(timeoutId);
    
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return await res.json();
  } catch (err) {
    clearTimeout(timeoutId);
    throw err;
  }
}
```

## Rate limits / Best practices

- Implement delays between requests (e.g., 1-2 seconds)
- Cache results for at least 30 seconds to avoid redundant queries
- Use exponential backoff on rate-limit errors (HTTP 429)
- Respect API terms of service

## Agent prompt

```text
You have [skill name] capability. When a user asks to [task description]:

1. Check if the input is valid by [validation steps]
2. Use the [specific API/tool] from the examples above
3. Handle errors gracefully with fallback to [alternative]
4. Return results in [format] with [specific fields]

Always prefer [free alternative] over [paid alternative] to save API costs.
```

## Troubleshooting

**Error scenario 1:**
- Symptom: What the user sees
- Solution: How to fix it

**Error scenario 2:**
- Symptom: What the user sees
- Solution: How to fix it

## See also

- [../related-skill-1/SKILL.md](../related-skill-1/SKILL.md) — Brief description of related skill
- [../related-skill-2/SKILL.md](../related-skill-2/SKILL.md) — Brief description of related skill

---

## Skill Writing Guidelines

### File Structure
- Each skill must be in its own folder: `skills/skill-name-kebab-case/`
- The skill content must be in a file named `SKILL.md` (not skill-name.md)
- Additional resources (images, config files) can be placed in the same folder

### Frontmatter (required)
- `name`: Kebab-case skill identifier (matches folder name)
- `description`: One-line summary for discovery. Use quotes if the description contains colons or special YAML characters. Include "Use when:" pattern with numbered use cases for clarity.
  - Example: `"Log all file changes to SQLite. Use when: (1) Tracking code changes, (2) Debugging issues, or (3) Auditing modifications."`

### Structure
1. Start with clear H1 title
2. Brief description paragraph
3. "When to use" bullet list for agent context
4. "Required tools / APIs" with install commands
5. ## Skills section with working code examples
6. Include both Bash and Node.js examples
7. Add rate limits / best practices
8. Agent prompt for LLM consumption
9. Troubleshooting section (optional but recommended)
10. See also links to related skills (use `../skill-name/SKILL.md` format)

### Code Examples
- Test all commands before committing
- Include comments explaining non-obvious steps
- Show multiple variations (basic, advanced, with error handling)
- Prefer free/public APIs over paid services
- Include timeout and error handling in Node.js examples

### Privacy & Cost
- Prioritize tools that don't require API keys
- Use local/offline solutions when possible
- Mention data privacy implications if applicable
- Highlight cost savings compared to paid alternatives

### YAML Frontmatter Notes
- Always quote description values that contain colons (`:`) to avoid YAML parsing errors
- Example: `description: "Use when: (1) Task A, (2) Task B"` (correct)
- Example: `description: Use when: (1) Task A` (incorrect - will cause YAML error)
- Test your YAML frontmatter before committing
