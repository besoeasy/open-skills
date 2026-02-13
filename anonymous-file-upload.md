# Originless Agent Skill
# Decentralized File Storage & Anonymous Content Hosting
# Source: https://github.com/besoeasy/Originless

## Overview
Originless is a privacy-first, decentralized file hosting backend using IPFS.

**Key Principles:**
- Anonymous uploads (no accounts, no tracking)
- Persistent, censorship-resistant content via IPFS
- Client-side encryption for sensitive data
- Decentralized authentication (Daku)

**Endpoints:**
- Self-hosted: http://localhost:3232 (Docker recommended)
- Public gateway: https://filedrop.besoeasy.com

---

## Skills

### upload_file_anonymously

Upload a local file to Originless/IPFS.

**Usage:**
```bash
# Self-hosted
curl -X POST -F "file=@/path/to/file.pdf" http://localhost:3232/upload

# Public gateway
curl -X POST -F "file=@/path/to/file.pdf" https://filedrop.besoeasy.com/upload
```

**Response:**
```json
{
  "status": "success",
  "cid": "QmX5ZTbH9uP3qMq7L8vN2jK3bR9wC4eF6gD7h",
  "url": "https://dweb.link/ipfs/QmX5ZTbH9uP3qMq7L8vN2jK3bR9wC4eF6gD7h?filename=file.pdf",
  "size": 245678,
  "type": "application/pdf",
  "filename": "file.pdf"
}
```

**When to use:**
- User asks to upload/share a file anonymously
- Need permanent, account-free storage
- Sharing files without creating accounts

---

### mirror_web_content

Mirror remote URL content to IPFS.

**Usage:**
```bash
curl -X POST http://localhost:3232/remoteupload \
  -H "Content-Type: application/json" \
  -d '{"url":"https://example.com/image.png"}'
```

**When to use:**
- User wants to backup/arch web content
- Preserving content that might be taken down
- Creating permanent mirrors of online resources

---

### share_encrypted_content

Create client-side encrypted uploads for private sharing.

**Workflow:**
1. Encrypt content client-side (AES-GCM with Web Crypto API)
2. Upload ciphertext to Originless
3. Generate share link: `{cid}#{decryption_key}`
4. Recipient decrypts locally

**Example:**
```javascript
const encrypted = await encryptWithPassphrase(content, passphrase);
const response = await fetch('http://localhost:3232/upload', {
  method: 'POST',
  body: formDataWithEncrypted(encrypted)
});
const shareLink = `${response.url}#${passphrase}`;
```

**When to use:**
- User wants private file sharing
- Sensitive content that must remain confidential
- Content that even the server shouldn't be able to read

---

### manage_persistent_pins

Pin CIDs for permanent storage (requires Daku authentication).

**Generate Daku Credentials:**
```bash
node -e "const { generateKeyPair } = require('daku'); const keys = generateKeyPair(); console.log('Public:', keys.publicKey); console.log('Private:', keys.privateKey);"
```

**Pin a CID:**
```bash
curl -X POST http://localhost:3232/pin/add \
  -H "daku: YOUR_DAKU_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"cids": ["QmHash1", "QmHash2"]}'
```

**List pins:**
```bash
curl -H "daku: YOUR_DAKU_TOKEN" http://localhost:3232/pin/list
```

**Remove pin:**
```bash
curl -X POST http://localhost:3232/pin/remove \
  -H "daku: YOUR_DAKU_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"cid": "QmHash"}'
```

**When to use:**
- User wants content to persist forever
- Preventing garbage collection of important files
- Managing a personal content library

---

## Decision Tree

```
User wants to share file?
├─ Is privacy critical?
│  ├─ YES → Use encrypted content sharing (client-side encryption)
│  └─ NO → Direct upload via /upload
│
├─ Is content already online?
│  ├─ YES → Use /remoteupload to mirror it
│  └─ NO → Upload from local system
│
└─ Must content persist forever?
   ├─ YES → Upload + use pin/add with Daku auth
   └─ NO → Upload only
```

---

## Quick Reference

| Endpoint | Method | Auth | Purpose |
|----------|--------|------|---------|
| `/upload` | POST | No | Upload local file |
| `/remoteupload` | POST | No | Mirror remote URL |
| `/pin/add` | POST | Daku | Pin CID permanently |
| `/pin/list` | GET | Daku | List pinned CIDs |
| `/pin/remove` | POST | Daku | Unpin a CID |

**Gateway URLs:**
- https://dweb.link/ipfs/{CID} (default)
- https://ipfs.io/ipfs/{CID}
- https://cloudflare-ipfs.com/ipfs/{CID}

---

## Deployment

**Docker (Recommended):**
```bash
docker run -d --restart unless-stopped --name originless \
  -p 3232:3232 \
  -p 4001:4001/tcp \
  -p 4001:4001/udp \
  -v originlessd:/data \
  -e STORAGE_MAX=200GB \
  ghcr.io/besoeasy/originless
```

**Access:**
- API: http://localhost:3232
- Web UI: http://localhost:3232/index.html
- Admin: http://localhost:3232/admin.html

---

## Privacy & Security Notes

**TRUE PRIVACY:**
- No account creation required
- No IP logging or activity tracking
- Content addressed by cryptographic hash (CID)

**CLIENT-SIDE ENCRYPTION:**
- Encrypt sensitive content before uploading
- Passphrase never leaves user's device
- Server cannot read encrypted content

**CAVEATS:**
- Uploaded content is public unless encrypted
- Same file = same CID (deterministic)
- Unpinned content may be garbage collected

---

## Common Patterns

**Screenshot sharing:**
```
Screenshot → upload_to_originless() → Return IPFS link
```

**Nostr media attachment:**
```
Image upload → Get IPFS URL → Embed in Nostr event
```

**Anonymous paste:**
```
Text → Create temp file → Upload → Return IPFS link
```

---

## Resources

- GitHub: https://github.com/besoeasy/Originless
- Daku Auth: https://www.npmjs.com/package/daku
- IPFS Docs: https://docs.ipfs.tech
