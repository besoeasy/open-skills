---
name: cinematic-landing-page
description: "Build a premium cinematic landing page with a full-screen video hero, glassmorphism text overlay, and psychology-backed conversion principles (halo effect, cognitive fluency, peak-end rule). Use when creating high-end marketing landing pages that need a wow-factor video background and professional, performance-focused design."
---

# Cinematic Landing Page

Build a high-converting cinematic landing page with an instant-loading full-screen video hero, a glassmorphism content overlay, and psychology-backed copy/layout. Everything is vanilla HTML/CSS/JS for perfect Core Web Vitals and GEOSEO — no React overhead.

## Quick quality checklist

- `name` matches folder name exactly (kebab-case)
- HTML uses `fetchpriority="high"` and `preload="auto"` on the video for instant loading
- CSS enforces massive whitespace, a single clear CTA, and smooth transitions on interactive elements
- JavaScript is minimal — only handling micro-interactions (scroll fades, parallax, click feedback)
- No API keys, tokens, or personal paths in examples
- Video model prompt is generic (no brand-specific or personal references)

## When to use

- Creating a premium marketing landing page with a cinematic video background
- Needing instant LCP and perfect Lighthouse scores (GEOSEO focus)
- Wanting psychology-backed design decisions (halo effect, cognitive fluency, peak-end rule)
- Building a single-file, self-contained HTML page that works offline

## Required tools / APIs

- No external API required
- Video generation: Sora, Kling, or any text-to-video model (for the hero background)
- Image generation: Midjourney or DALL-E 3 (for still variations)
- All code is vanilla HTML/CSS/JS — no build tools or dependencies

## Skills

### basic_usage

Build a cinematic landing page with a video hero and glassmorphism overlay.

```bash
# Generate the hero video with a cinematic prompt (example for Sora/Kling)
# Use the prompt template in Section 3 of the skill

# Serve the page locally for preview
python3 -m http.server 8000
# Open http://localhost:8000/cinematic-landing.html
```

**Node.js:**

```javascript
// No dependencies needed — vanilla HTML/CSS/JS
// The page is a single self-contained file
// Use fetchpriority="high" and preload="auto" on the <video> tag
// for instant loading (GEOSEO optimization)
```

### robust_usage

Production-oriented variant with tiered workflow and multiple prompt variations.

```bash
# Tier 1: Generate the hero video
# Use the cinematic video prompt template (8-second continuous tracking shot)

# Tier 2: Build the page from the code template
# Replace [BRAND], [SUBTITLE], [CTA LABEL] with actual values

# Tier 3: Generate 3-5 still variations before committing to a long video
# Use the Visual LLM prompt template (Midjourney/DALL-E 3)

# Tier 4: Hand off to a developer for CSS/interaction polish
```

**Node.js:**

```javascript
// The psychological principles JSON can be fed into AI coding agents
// (e.g., GitHub Copilot, Cursor, Claude Artifacts) to auto-generate
// CSS rules, copywriting, and UX flows
//
// Key rules the agent should extract:
// 1. HTML must load video instantly (fetchpriority="high", preload="auto")
// 2. CSS must enforce whitespace, single CTA, smooth transitions
// 3. JS must be minimal — only micro-interactions (scroll fades, parallax, click feedback)
```

## Output format

- A single self-contained HTML file with embedded CSS and JS
- No external dependencies (fonts loaded from Google Fonts CDN)
- Works completely offline after initial font load
- Perfect Lighthouse score (LCP < 2.5s, FID < 100ms, CLS = 0)

## Rate limits / Best practices

- Video generation: follow your video model's rate limits (typically 1-5 generations/min)
- Cache generated video assets locally — don't regenerate for each iteration
- Use exponential backoff if the video model API returns 429 errors
- Respect video model terms of service regarding generated content

## Agent prompt

```text
You have cinematic-landing-page capability. When a user asks to build a cinematic landing page:

1. Generate the hero video using the cinematic prompt template (8-second continuous tracking shot, macro scale, volumetric lighting)
2. Build the HTML page using the code template — full-screen video background with glassmorphism overlay
3. Apply psychological principles: halo effect (instant hero load), cognitive fluency (whitespace, minimal CTA), peak-end rule (micro-interactions on hover/click/scroll)
4. Return the complete HTML file ready to open in a browser
5. Confirm the page renders correctly (no console errors, video plays, CTA is interactive)

Always prefer vanilla HTML/CSS/JS over React for landing pages — React adds bundle overhead that hurts LCP.
```

## Troubleshooting

**Video doesn't autoplay:**
- Symptom: Browser blocks video autoplay
- Solution: Ensure `muted` and `playsinline` attributes are on the `<video>` tag. Most browsers require muted autoplay.

**Layout shift (CLS > 0):**
- Symptom: Content jumps when video loads
- Solution: Set explicit `width` and `height` on the video element, use `object-fit: cover`, and ensure the video container has fixed dimensions matching the viewport.

**CTA button not visible:**
- Symptom: Button blends into video background
- Solution: Increase `backdrop-filter: blur()` intensity on the overlay, or add a semi-transparent dark background behind the CTA.

**Page feels "cheap":**
- Symptom: Layout is cluttered, multiple CTAs competing for attention
- Solution: Apply cognitive fluency — remove competing elements, increase whitespace, keep a single primary CTA per viewport.

## See also

- `~/open-skills/skills/build-dashboard/SKILL.md` — For data-driven dashboard UIs
- `~/open-skills/skills/ui-ux-design-pro/SKILL.md` — For general UI/UX design principles
