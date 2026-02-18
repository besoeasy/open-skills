---
name: song-website
description: Generate a minimal animated song website with album art, lyrics, and Wikipedia links, then upload to originless.
---

# Song Website Skill

Purpose: Create a simple, animated HTML song card/website featuring album art, lyrics (user-provided or fetched legally), and related Wikipedia links, with a white background and lots of animations. Upload the generated index.html to originless for hosting.

What it does:
- Fetches song metadata (album art, artist info) from public APIs like iTunes or MusicBrainz.
- Generates an HTML page with:
  - White background.
  - Animations (fade-ins, bounces, rotations on hover).
  - Album art image.
  - Lyrics section (user-provided; do not fetch copyrighted lyrics).
  - Footer with Wikipedia links (artist, song, album).
- Uploads the HTML to originless (https://originless.io or similar service) for free hosting.

Files:
- song_website_generator.js — Node.js script to generate and upload the HTML.

Prerequisites:
- Node.js 18+ (native fetch).
- curl for Bash examples.
- No API keys needed (uses free endpoints).

Bash example (fetches album art via iTunes API, generates HTML, uploads to originless):

```bash
SONG="Armed and Dangerous"
ARTIST="King Von"

# Fetch album art URL
ART_URL=$(curl -s "https://itunes.apple.com/search?term=${ARTIST} ${SONG}&limit=1" | jq -r '.results[0].artworkUrl100 // empty')

# Generate HTML (simplified; add animations in full script)
HTML="<html><head><style>body{background:white;}img{animation:spin 2s infinite;}@keyframes spin{from{transform:rotate(0deg);}to{transform:rotate(360deg);}}</style></head><body><h1>${SONG}</h1><img src='${ART_URL}'><p>Lyrics here...</p><a href='https://en.wikipedia.org/wiki/King_Von'>Wikipedia</a></body></html>"

# Upload to originless (use their API or manual upload; example uses curl to their endpoint)
curl -X POST -H "Content-Type: text/html" --data-binary "$HTML" https://api.originless.io/upload
```

Node.js example (recommended; generates full animated HTML with lyrics placeholder, uploads to originless):

```javascript
async function generateSongWebsite(song, artist, lyrics = "Lyrics go here...") {
  // Fetch album art
  const search = await fetch(`https://itunes.apple.com/search?term=${encodeURIComponent(artist + ' ' + song)}&limit=1`);
  const data = await search.json();
  const artUrl = data.results[0]?.artworkUrl100?.replace('100x100', '300x300') || 'https://via.placeholder.com/300';

  // Generate HTML with animations
  const html = `
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>${song} - ${artist}</title>
  <style>
    body { background: white; font-family: Arial, sans-serif; padding: 20px; }
    .card { max-width: 600px; margin: 0 auto; padding: 20px; border-radius: 10px; box-shadow: 0 0 10px rgba(0,0,0,0.1); animation: fadeIn 1s; }
    @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
    img { width: 300px; height: 300px; border-radius: 10px; animation: bounce 3s infinite; }
    @keyframes bounce { 0%, 20%, 50%, 80%, 100% { transform: translateY(0); } 40% { transform: translateY(-10px); } 60% { transform: translateY(-5px); } }
    h1 { animation: slideIn 1s; }
    @keyframes slideIn { from { transform: translateX(-100%); } to { transform: translateX(0); } }
    p { animation: fadeIn 2s; }
    a { color: blue; animation: pulse 2s infinite; }
    @keyframes pulse { 0% { transform: scale(1); } 50% { transform: scale(1.1); } 100% { transform: scale(1); } }
  </style>
</head>
<body>
  <div class="card">
    <h1>${song} by ${artist}</h1>
    <img src="${artUrl}" alt="Album Art">
    <h2>Lyrics</h2>
    <p>${lyrics.replace(/\n/g, '<br>')}</p>
    <h2>Related Wikipedia</h2>
    <a href="https://en.wikipedia.org/wiki/${encodeURIComponent(song)}">Song Page</a> |
    <a href="https://en.wikipedia.org/wiki/${encodeURIComponent(artist)}">Artist Page</a> |
    <a href="https://en.wikipedia.org/wiki/${encodeURIComponent(song)}_(song)">Album/Song Wiki</a>
  </div>
</body>
</html>
  `;

  // Upload to originless (example endpoint; adjust for their API)
  const uploadRes = await fetch('https://api.originless.io/upload', {
    method: 'POST',
    headers: { 'Content-Type': 'text/html' },
    body: html
  });
  const uploadData = await uploadRes.json();
  return { html, uploadUrl: uploadData.url };
}

// Usage
// generateSongWebsite('Armed and Dangerous', 'King Von').then(console.log);
```

Notes:
- Album art fetched from iTunes (free, no key).
- Lyrics: User-provided; do not fetch copyrighted lyrics.
- Animations: CSS keyframes for fade-in, bounce, slide, pulse.
- Upload: Assumes originless.io API; check their docs for exact endpoint.
- Legal: Only use for non-commercial, fair-use purposes.

See also:
- SKILL_TEMPLATE.md

