---
name: file-tracker
description: "Log all file changes (write, edit, delete) to a SQLite database for debugging and audit. Use when: (1) Tracking code changes, (2) Debugging issues, (3) Auditing file modifications, or (4) The user asks to track file changes."
---

# File Tracker

Log every file change (write, edit, delete) to a SQLite database for debugging and audit trail.

## Database

**Location:** `/root/.nanobot/workspace/chat_logs.db`

**Schema:**
```sql
CREATE TABLE file_changes (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  timestamp TEXT NOT NULL,
  channel TEXT,
  chat_id TEXT,
  action TEXT NOT NULL,  -- 'write', 'edit', 'delete'
  file_path TEXT NOT NULL,
  old_content TEXT,      -- for edits/deletes
  new_content TEXT       -- for writes/edits
);

CREATE INDEX idx_file_path ON file_changes(file_path);
CREATE INDEX idx_timestamp ON file_changes(timestamp);
```

## Usage

After each file operation, log the change:

```python
import sqlite3
from datetime import datetime

def log_file_change(channel: str, chat_id: str, action: str, file_path: str, old_content: str = None, new_content: str = None):
    conn = sqlite3.connect('/root/.nanobot/workspace/chat_logs.db')
    conn.execute(
        "INSERT INTO file_changes (timestamp, channel, chat_id, action, file_path, old_content, new_content) VALUES (?, ?, ?, ?, ?, ?, ?)",
        (datetime.utcnow().isoformat(), channel, chat_id, action, file_path, old_content, new_content)
    )
    conn.commit()
    conn.close()

# Log a write operation
log_file_change("telegram", "7676486207", "write", "/path/to/file.md", None, "# New content")

# Log an edit operation
log_file_change("telegram", "7676486207", "edit", "/path/to/file.md", "old text", "new text")

# Log a delete operation
log_file_change("telegram", "7676486207", "delete", "/path/to/file.md", "deleted content", None)
```

## When to Log

Log after every:
1. `write_file` operation - action: "write"
2. `edit_file` operation - action: "edit" (capture old and new content)
3. `delete_file` operation - action: "delete"

## Query Examples

```python
# Get recent changes to a file
def get_file_history(file_path: str, limit: int = 20):
    conn = sqlite3.connect('/root/.nanobot/workspace/chat_logs.db')
    cursor = conn.execute(
        "SELECT timestamp, action, old_content, new_content FROM file_changes WHERE file_path=? ORDER BY id DESC LIMIT ?",
        (file_path, limit)
    )
    results = cursor.fetchall()
    conn.close()
    return results

# Get all changes from a session/channel
def get_session_changes(channel: str, chat_id: str):
    conn = sqlite3.connect('/root/.nanobot/workspace/chat_logs.db')
    cursor = conn.execute(
        "SELECT timestamp, action, file_path, new_content FROM file_changes WHERE channel=? AND chat_id=? ORDER BY id DESC",
        (channel, chat_id)
    )
    results = cursor.fetchall()
    conn.close()
    return results

# Search for changes to files matching a pattern
def search_file_changes(pattern: str):
    conn = sqlite3.connect('/root/.nanobot/workspace/chat_logs.db')
    cursor = conn.execute(
        "SELECT timestamp, channel, chat_id, action, file_path FROM file_changes WHERE file_path LIKE ? ORDER BY id DESC",
        (f'%{pattern}%',)
    )
    results = cursor.fetchall()
    conn.close()
    return results
```
