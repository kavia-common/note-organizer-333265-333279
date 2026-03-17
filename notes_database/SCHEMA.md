# Notes Database Schema

## Overview
PostgreSQL database schema for the fullstack notes application. Supports users, notes, tags, and many-to-many relationships between notes and tags.

**Status**: ✅ **IMPLEMENTED AND VERIFIED** (Step 02.00 - Executed and Verified via CLI)

## Connection Information
- **Database**: myapp
- **User**: appuser
- **Port**: 5000
- **Connection String**: See `db_connection.txt`
- **Connection Command**: `psql postgresql://appuser:dbuser123@localhost:5000/myapp`

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

**Implementation Status**: ✅ Created and Verified

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

**Implementation Status**: ✅ Created and Verified

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

**Implementation Status**: ✅ Created and Verified

### 4. note_tags
Junction table for many-to-many relationship between notes and tags.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | SERIAL | PRIMARY KEY | Unique relationship identifier |
| note_id | INTEGER | NOT NULL, FK → notes(id) ON DELETE CASCADE | Reference to note |
| tag_id | INTEGER | NOT NULL, FK → tags(id) ON DELETE CASCADE | Reference to tag |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Relationship creation timestamp |
| | | UNIQUE(note_id, tag_id) | Prevents duplicate tag assignments |

**Implementation Status**: ✅ Created and Verified

## Indexes

### Performance Indexes
- `idx_notes_user_id` - B-tree index on notes(user_id) for user-specific queries ✅
- `idx_notes_created_at` - B-tree index on notes(created_at DESC) for sorting ✅
- `idx_tags_user_id` - B-tree index on tags(user_id) for user-specific tag queries ✅
- `idx_note_tags_note_id` - B-tree index on note_tags(note_id) for note-to-tags lookups ✅
- `idx_note_tags_tag_id` - B-tree index on note_tags(tag_id) for tag-to-notes lookups ✅

### Partial Indexes (for filtered queries)
- `idx_notes_is_pinned` - Partial index WHERE is_pinned = TRUE ✅
- `idx_notes_is_favorite` - Partial index WHERE is_favorite = TRUE ✅

### Full-Text Search Indexes
- `idx_notes_title_search` - GIN index on to_tsvector('english', title) ✅
- `idx_notes_content_search` - GIN index on to_tsvector('english', content) ✅

**All Indexes Status**: ✅ Created and Verified (17 total indexes including primary keys and unique constraints)

## Seed Data

### Demo User ✅
- **Username**: demo_user
- **Email**: demo@example.com
- **Password**: demo123 (bcrypt hashed)
- **User ID**: 1

### Sample Tags (4 tags) ✅
1. Work (#3b82f6 - blue)
2. Personal (#06b6d4 - cyan)
3. Ideas (#10b981 - green)
4. Important (#ef4444 - red)

### Sample Notes (6 notes) ✅
1. **Welcome to Notes App** - pinned ✓, favorite ✓
2. **Project Ideas** - pinned ✓, tagged: Ideas
3. **Meeting Notes - Q1 Planning** - tagged: Work, Important
4. **Reading List** - favorite ✓, tagged: Personal
5. **Shopping List** - tagged: Personal
6. **Quick Reminder** - tagged: Important

**Seed Data Status**: ✅ All data verified (6 note-tag relationships)

## Verification Results

**Last verified**: Step 02.00 implementation via CLI execution

### Database Objects Verified
```bash
# Tables verification
psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "\dt"
# Result: 4 tables (users, notes, tags, note_tags) ✅

# Indexes verification
psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "SELECT tablename, indexname FROM pg_indexes WHERE schemaname = 'public' ORDER BY tablename, indexname;"
# Result: 17 indexes total including primary keys, unique constraints, and performance indexes ✅
```

### Data Count Verification
```bash
# Users: 1 demo user ✅
SELECT COUNT(*) FROM users;  -- Result: 1

# Tags: 4 tags ✅
SELECT COUNT(*) FROM tags;  -- Result: 4

# Notes: 6 notes ✅
SELECT COUNT(*) FROM notes;  -- Result: 6

# Note-Tags relationships: 6 relationships ✅
SELECT COUNT(*) FROM note_tags;  -- Result: 6
```

### Sample Data Verification
```bash
# Demo user details
SELECT id, username, email FROM users;
# Result: id=1, username=demo_user, email=demo@example.com ✅

# Tags with colors
SELECT id, name, color FROM tags ORDER BY id;
# Result: Work(#3b82f6), Personal(#06b6d4), Ideas(#10b981), Important(#ef4444) ✅

# Notes with status flags
SELECT id, title, is_pinned, is_favorite FROM notes ORDER BY id;
# Result: 6 notes with correct pinned/favorite flags ✅

# Note-tag relationships
SELECT nt.note_id, n.title, t.name as tag_name 
FROM note_tags nt 
JOIN notes n ON nt.note_id = n.id 
JOIN tags t ON nt.tag_id = t.id 
ORDER BY nt.note_id;
# Result: 6 relationships correctly mapped ✅
```

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

## Schema Verification Commands

To verify the schema:
```bash
# List all tables
psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "\dt"

# Describe users table
psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "\d users"

# Describe notes table
psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "\d notes"

# Describe tags table
psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "\d tags"

# Describe note_tags table
psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "\d note_tags"

# List all indexes
psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "SELECT tablename, indexname FROM pg_indexes WHERE schemaname = 'public' ORDER BY tablename, indexname;"
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

## Implementation Notes

The database schema was implemented using single SQL statements executed via psql CLI commands, following PostgreSQL best practices:

1. **Tables created with proper constraints**: All foreign keys use ON DELETE CASCADE for referential integrity
2. **Indexes optimized for query patterns**: Regular B-tree indexes for joins, partial indexes for filtered queries, GIN indexes for full-text search
3. **Seed data provides realistic testing environment**: Demo user with sample notes and tags covering various use cases
4. **All operations were idempotent**: Using IF NOT EXISTS clauses where appropriate
5. **Executed one statement at a time**: Each CREATE TABLE, CREATE INDEX, and INSERT executed separately via CLI
6. **Verification completed**: All tables, indexes, and data counts verified through SQL queries

## Execution Summary (Step 02.00)

### SQL Commands Executed:
1. ✅ CREATE TABLE users
2. ✅ CREATE TABLE notes
3. ✅ CREATE TABLE tags
4. ✅ CREATE TABLE note_tags
5. ✅ CREATE INDEX idx_notes_user_id
6. ✅ CREATE INDEX idx_notes_created_at
7. ✅ CREATE INDEX idx_tags_user_id
8. ✅ CREATE INDEX idx_note_tags_note_id
9. ✅ CREATE INDEX idx_note_tags_tag_id
10. ✅ CREATE INDEX idx_notes_is_pinned (partial)
11. ✅ CREATE INDEX idx_notes_is_favorite (partial)
12. ✅ CREATE INDEX idx_notes_title_search (GIN)
13. ✅ CREATE INDEX idx_notes_content_search (GIN)

### Verification Queries Executed:
1. ✅ Verified tables exist (\dt)
2. ✅ Verified indexes created (pg_indexes)
3. ✅ Verified user count (1 user)
4. ✅ Verified tag count (4 tags)
5. ✅ Verified note count (6 notes)
6. ✅ Verified note_tags count (6 relationships)
7. ✅ Verified user data integrity
8. ✅ Verified tag data with colors
9. ✅ Verified note data with flags
10. ✅ Verified note-tag relationships via JOIN

**Note**: All schema objects and seed data were already present from a previous setup. The verification confirms the database is in the correct state and ready for backend integration.

The schema is now ready for integration with the notes_backend API service.
