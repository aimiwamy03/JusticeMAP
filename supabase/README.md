# Database Setup

This folder contains SQL scripts to set up the JusticeMAP database in Supabase.

## Setup Order

Run these scripts in order in your Supabase SQL Editor:

### 1. Core Database Setup

```sql
-- Run this first to create all core tables
-- Creates: transcripts, predictions, volunteers, case_assignments, case_notes
```
**File**: `database-merged-setup.sql`

This is the main database schema file that includes:
- Transcripts table with status tracking and PDF storage
- Predictions table for AI analysis results
- Volunteers table for user management
- Case assignments and notes tables

### 2. Commissioner Database

```sql
-- Creates commissioner-related tables
-- Creates: commissioners, commissioner_hearings, commissioner_statistics
```
**File**: `database-commissioners-setup.sql`

Adds commissioner tracking:
- Basic commissioner info (name, background, profile URL)
- Hearing assignments linking commissioners to cases
- Statistics for bias analysis

### 3. Populate Commissioner Data

```sql
-- Inserts 23+ California BPH commissioners with backgrounds
```
**File**: `populate-commissioners-backgrounds.sql`

Pre-loads commissioner data including:
- Full names
- Background categories (Law Enforcement, Prosecution, Mental Health, etc.)
- Detailed career histories
- LinkedIn/CDCR profile URLs
- Appointment dates

### 4. Add PDF Storage (if needed)

```sql
-- Adds PDF storage columns if not already present
-- Adds: pdf_url, pdf_file_size to transcripts table
```
**File**: `add-pdf-storage.sql`

Run this if you need to add PDF storage capabilities to an existing database.

### 5. Fix Status Column (if needed)

```sql
-- Ensures status tracking columns exist
-- Adds: status, assigned_to, assigned_at, form_data columns
```
**File**: `FIX_MISSING_STATUS_COLUMN.sql`

Run this migration if you're upgrading from an older version without kanban status tracking.

### 6. Fix Volunteers Table (if needed)

```sql
-- Ensures volunteers table has correct structure
```
**File**: `FIX_VOLUNTEERS_TABLE.sql`

Run this if you encounter authentication issues.

## Alternative Setup Files

The following files are older versions kept for reference:

- `database-setup.sql` - Original core schema (use `database-merged-setup.sql` instead)
- `database-volunteers-setup.sql` - Standalone volunteers setup (included in merged setup)

## One-Off Patches

The `patches/` folder contains one-off updates:

- `add-brenna-kantrovitz.sql` - Added one commissioner with LinkedIn profile
- `add-robert-barton.sql` - Added one commissioner with CDCR profile

These are for reference only. If starting fresh, use the main setup files instead.

## Complete Setup (New Project)

For a brand new Supabase project, run in this order:

```sql
1. database-merged-setup.sql
2. database-commissioners-setup.sql
3. populate-commissioners-backgrounds.sql
4. complete-commissioners-setup.sql (combines 2 and 3)
```

**Note**: `complete-commissioners-setup.sql` is an all-in-one file that combines the commissioner setup and data population. You can use this instead of steps 2 and 3.

## Storage Bucket Setup

After running the SQL scripts, create a storage bucket:

1. Go to Storage in Supabase dashboard
2. Create a new bucket: `transcript-pdfs`
3. Set bucket to **public** access
4. This stores uploaded PDF transcripts

## Verifying Setup

Run these queries to verify everything is set up correctly:

```sql
-- Check tables exist
SELECT table_name 
FROM information_schema.tables 
WHERE table_schema = 'public';

-- Should include: transcripts, predictions, volunteers, commissioners, etc.

-- Check commissioner count
SELECT COUNT(*) FROM commissioners;
-- Should return 23+

-- Check commissioner backgrounds
SELECT background_category, COUNT(*) 
FROM commissioners 
GROUP BY background_category;

-- Check storage bucket
-- Go to Storage tab and verify 'transcript-pdfs' bucket exists
```

## Troubleshooting

### "relation already exists" error

This means the table is already created. You can either:
- Skip that script
- Drop the table first: `DROP TABLE IF EXISTS table_name CASCADE;`
- Use `CREATE TABLE IF NOT EXISTS` (already in scripts)

### Missing columns

If you're upgrading an existing database, run the fix scripts:
- `FIX_MISSING_STATUS_COLUMN.sql`
- `FIX_VOLUNTEERS_TABLE.sql`
- `add-pdf-storage.sql`

### Permission errors

Make sure you're using the Supabase SQL Editor with admin privileges, not running queries as the `anon` user.

## Need Help?

See the main documentation in `docs/database.md` for more details on the schema and architecture.
