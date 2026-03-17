# Notes Database Schema

## Overview
PostgreSQL database schema for the fullstack notes application. Supports users, notes, tags, and many-to-many relationships between notes and tags.

## Connection Information
- **Database**: myapp
- **User**: appuser
- **Port**: 5000
- **Connection String**: See `db_connection.txt`

## Tables

### 1. users
Stores user account information.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | SERIAL | PRIMARY KEY | Unique user identifier |
| username | VARCHAR(255) | UNIQUE, NOT NULL | User's username |
| email | VARCHAR(255) | UNIQUE, NOT NULL | User's email address |
| password_hash | VARCHAR(255) | NOT NULL | Bcrypt hashed password |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Account creation timestamp |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Last update timestamp |

### 2. notes
Stores user notes with metadata.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | SERIAL | PRIMARY KEY | Unique note identifier |
| user_id | INTEGER | NOT NULL, FK → users(id) ON DELETE CASCADE | Owner of the note |
| title | VARCHAR(500) | NOT NULL | Note title |
| content | TEXT | | Note content/body |
| is_pinned | BOOLEAN | DEFAULT FALSE | Pin status for quick access |
| is_favorite | BOOLEAN | DEFAULT FALSE | Favorite status |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Note creation timestamp |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Last update timestamp |

### 3. tags
Stores user-defined tags for organizing notes.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | SERIAL | PRIMARY KEY | Unique tag identifier |
| user_id | INTEGER | NOT NULL, FK → users(id) ON DELETE CASCADE | Owner of the tag |
| name | VARCHAR(100) | NOT NULL | Tag name |
| color | VARCHAR(7) | DEFAULT '#3b82f6' | Hex color code for tag |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Tag creation timestamp |
| | | UNIQUE(user_id, name) | Ensures unique tag names per user |

### 4. note_tags
Junction table for many-to-many relationship between notes and tags.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | SERIAL | PRIMARY KEY | Unique relationship identifier |
| note_id | INTEGER | NOT NULL, FK → notes(id) ON DELETE CASCADE | Reference to note |
| tag_id | INTEGER | NOT NULL, FK → tags(id) ON DELETE CASCADE | Reference to tag |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Relationship creation timestamp |
| | | UNIQUE(note_id, tag_id) | Prevents duplicate tag assignments |

## Indexes

### Performance Indexes
- `idx_notes_user_id` - B-tree index on notes(user_id) for user-specific queries
- `idx_notes_created_at` - B-tree index on notes(created_at DESC) for sorting
- `idx_tags_user_id` - B-tree index on tags(user_id) for user-specific tag queries
- `idx_note_tags_note_id` - B-tree index on note_tags(note_id) for note-to-tags lookups
- `idx_note_tags_tag_id` - B-tree index on note_tags(tag_id) for tag-to-notes lookups

### Partial Indexes (for filtered queries)
- `idx_notes_is_pinned` - Partial index WHERE is_pinned = TRUE
- `idx_notes_is_favorite` - Partial index WHERE is_favorite = TRUE

### Full-Text Search Indexes
- `idx_notes_title_search` - GIN index on to_tsvector('english', title)
- `idx_notes_content_search` - GIN index on to_tsvector('english', content)

## Seed Data

### Demo User
- **Username**: demo_user
- **Email**: demo@example.com
- **Password**: demo123 (hashed with bcrypt)

### Sample Tags (4 tags)
1. Work (#3b82f6 - blue)
2. Personal (#06b6d4 - cyan)
3. Ideas (#10b981 - green)
4. Important (#ef4444 - red)

### Sample Notes (6 notes)
1. Welcome to Notes App (pinned, favorite)
2. Project Ideas (pinned, tagged: Ideas)
3. Meeting Notes - Q1 Planning (tagged: Work, Important)
4. Reading List (favorite, tagged: Personal)
5. Shopping List (tagged: Personal)
6. Quick Reminder (tagged: Important)

## Query Examples

### Get all notes for a user with their tags
```sql
SELECT 
    n.id, n.title, n.content, n.is_pinned, n.is_favorite, n.created_at,
    ARRAY_AGG(t.name) as tags
FROM notes n
LEFT JOIN note_tags nt ON n.id = nt.note_id
LEFT JOIN tags t ON nt.tag_id = t.id
WHERE n.user_id = 1
GROUP BY n.id
ORDER BY n.is_pinned DESC, n.created_at DESC;
```

### Full-text search across title and content
```sql
SELECT id, title, content
FROM notes
WHERE user_id = 1
  AND (
    to_tsvector('english', title) @@ plainto_tsquery('english', 'search term')
    OR to_tsvector('english', content) @@ plainto_tsquery('english', 'search term')
  );
```

### Get notes by tag
```sql
SELECT n.*
FROM notes n
JOIN note_tags nt ON n.id = nt.note_id
JOIN tags t ON nt.tag_id = t.id
WHERE n.user_id = 1 AND t.name = 'Work';
```

## Schema Verification

To verify the schema:
```bash
psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "\dt"
psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "\d users"
psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "\d notes"
psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "\d tags"
psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "\d note_tags"
```

## Maintenance

### Backup
```bash
./backup_db.sh
```

### Restore
```bash
./restore_db.sh
```
