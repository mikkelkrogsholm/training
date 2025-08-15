---
description: Check database health and migration status
allowed-tools: Bash, Read, Grep
---

## Context
- Database location: `/data/db.sqlite`
- Migrations directory: !`find . -type d -name "migrations" -o -name "drizzle" | head -5`
- Schema files: !`find . -path "*/db/*.ts" -o -path "*/schema/*.ts" | head -10`

## Your Task

Perform a comprehensive database health check:

1. **Database Status**:
   - Check if SQLite database exists
   - Verify WAL mode is enabled
   - Check database size and last modified time
   - List all tables in the database

2. **Schema Review**:
   - Check for core tables: users, settings, _migrations
   - Verify movement-related tables if Phase 3+ completed
   - Check for session tables if Phase 4+ completed
   - Verify indices on key columns (user_id, movement_id, date)

3. **Migration Status**:
   - List applied migrations from _migrations table
   - Check for pending migrations
   - Verify module migrations if modules exist
   - Check migration file naming consistency

4. **Data Integrity Checks**:
   - Verify canonical units (kg, meters, seconds) in relevant tables
   - Check for proper foreign key relationships
   - Verify bcrypt password hashes in users table
   - Check for orphaned records

5. **Performance Considerations**:
   - Table sizes and row counts
   - Index usage statistics if available
   - Identify potential N+1 query risks

Report findings with:
- ✅ Healthy aspects
- ⚠️ Warnings or optimizations needed
- ❌ Critical issues to fix
- 📊 Database statistics
- 🔧 Recommended actions