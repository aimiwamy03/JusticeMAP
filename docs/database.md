# Database Guide

This guide covers the database schema, setup process, and important tables in JusticeMAP.

## Database Schema Overview

JusticeMAP uses PostgreSQL (via Supabase) with the following core tables:

### Core Tables

- `transcripts` - Parole hearing transcripts with metadata and PDF storage
- `predictions` - AI analysis results (innocence claims, bias language, scores)
- `volunteers` - User accounts with role-based access
- `case_assignments` - Tracks which volunteers are assigned to which cases
- `case_notes` - Volunteer annotations on cases
- `commissioners` - Database of 23+ California BPH commissioners
- `commissioner_hearings` - Links commissioners to specific hearings
- `commissioner_statistics` - Performance metrics for commissioners

## Setting Up the Database

Follow these steps in order using the Supabase SQL Editor:

### 1. Core Schema Setup

Run `database-merged-setup.sql` to create:

- Transcripts table with columns for:
  - Basic info: `id`, `filename`, `content`, `date_uploaded`
  - Metadata: `hearing_date`, `inmate_name`, `cdcr_number`, `facility`
  - Status tracking: `status` (assigned/in_review/completed/flagged/unassigned)
  - Assignment: `assigned_to` (volunteer ID), `assigned_at` (timestamp)
  - Form data: `form_data` (JSON for generated forms)
  - PDF storage: `pdf_url`, `pdf_file_size`
  
- Predictions table for storing AI analysis:
  - Explicit innocence claims
  - Implicit signals
  - Contextual signals
  - Bias language patterns
  - Overall innocence score (0-100)
  - Citations with line and page numbers

- Volunteers table:
  - Authentication fields
  - Role management (volunteer/admin)
  - Activity tracking

- Case assignments and notes tables

### 2. Commissioner Database

Run `database-commissioners-setup.sql` to create:

```sql
commissioners (
  id uuid,
  name text,
  background_category text,
  background_details text,
  profile_url text,
  photo_url text,
  is_active boolean,
  appointed_date date,
  created_at timestamp,
  updated_at timestamp
)
```

**Background Categories:**
- Law Enforcement & Corrections
- Legal, Judicial & Prosecution
- Legal, Judicial & Mixed Legal
- Mental Health & Social Services
- Parole Board Administration

### 3. Populate Commissioner Data

Run `populate-commissioners-backgrounds.sql` to load 23+ California BPH commissioners with:

- Full names
- Detailed backgrounds
- LinkedIn or CDCR profile URLs
- Appointment dates
- Active status

### 4. Add PDF Storage Capability

Run `add-pdf-storage.sql` to add columns:

```sql
ALTER TABLE transcripts
ADD COLUMN pdf_url text,
ADD COLUMN pdf_file_size bigint;
```

This enables storing original PDF transcripts in Supabase Storage and linking them to transcript records.

### 5. Fix Status Columns (if needed)

Run `FIX_MISSING_STATUS_COLUMN.sql` to ensure these columns exist:

```sql
ALTER TABLE transcripts
ADD COLUMN IF NOT EXISTS status text DEFAULT 'unassigned',
ADD COLUMN IF NOT EXISTS assigned_to uuid REFERENCES volunteers(id),
ADD COLUMN IF NOT EXISTS assigned_at timestamp with time zone,
ADD COLUMN IF NOT EXISTS form_data jsonb;
```

### 6. Fix Volunteers Table (if needed)

Run `FIX_VOLUNTEERS_TABLE.sql` to ensure the volunteers table has proper structure and authentication fields.

## Storage Bucket Setup

### Create transcript-pdfs Bucket

1. Go to Storage in Supabase dashboard
2. Click "New Bucket"
3. Name: `transcript-pdfs`
4. Set to **Public** access
5. Click Create

This bucket stores uploaded PDF transcripts. The `pdf_url` column in the transcripts table stores the public URL to the file.

## Important Queries

### Get All Transcripts with Volunteer Info

```sql
SELECT 
  t.*,
  v.name as volunteer_name,
  v.email as volunteer_email
FROM transcripts t
LEFT JOIN volunteers v ON t.assigned_to = v.id
ORDER BY t.date_uploaded DESC;
```

### Get Commissioner Panel for a Case

```sql
SELECT 
  c.name,
  c.background_category,
  c.background_details,
  c.profile_url
FROM commissioner_hearings ch
JOIN commissioners c ON ch.commissioner_id = c.id
WHERE ch.transcript_id = 'your-transcript-id-here';
```

### Get AI Analysis for a Transcript

```sql
SELECT 
  claim_type,
  text,
  confidence,
  line_number,
  page_number
FROM predictions
WHERE transcript_id = 'your-transcript-id-here'
ORDER BY confidence DESC;
```

### Get Cases by Status (Kanban Columns)

```sql
-- Assigned to Me
SELECT * FROM transcripts 
WHERE status = 'assigned' AND assigned_to = 'volunteer-id';

-- Under Review
SELECT * FROM transcripts 
WHERE status = 'in_review' AND assigned_to = 'volunteer-id';

-- Forms Generated
SELECT * FROM transcripts 
WHERE status = 'completed' AND assigned_to = 'volunteer-id';

-- Commissioner Panel Scheduled
SELECT * FROM transcripts 
WHERE status = 'flagged' AND assigned_to = 'volunteer-id';
```

## Database Migrations

As the project evolves, migration files are stored in `supabase/patches/`:

- `add-brenna-kantrovitz.sql` - Added commissioner with LinkedIn profile
- `add-robert-barton.sql` - Added commissioner with CDCR profile
- `complete-commissioners-setup.sql` - Full commissioner schema with data

These are one-off patches. If starting fresh, use the main setup files instead.

## Row Level Security (RLS)

Supabase uses RLS for data access control:

- **Transcripts**: Users can only see transcripts they uploaded or are assigned to
- **Volunteers**: Users can only read their own profile
- **Commissioners**: Public read access (needed for analysis)
- **Predictions**: Users can only see predictions for their assigned cases

Check the SQL files for the exact RLS policies.

## Service Architecture

The app uses a service layer (`src/services/`) that abstracts database operations:

- `transcripts.ts` - CRUD operations for transcripts, PDF upload/download
- `commissioners.ts` - Commissioner lookup and panel analysis
- `volunteers.ts` - User authentication and profile management
- `innocenceDetector.ts` - Orchestrates AI analysis and stores results
- `gemini.ts` - Calls Gemini API for AI analysis
- `formProcessor.ts` - Extracts form data from transcripts
- `priority.ts` - Calculates AI-driven case priority scores

## Performance Considerations

- **Indexes**: The main tables have indexes on frequently queried columns (`transcript_id`, `volunteer_id`, etc.)
- **N+1 Queries**: Use JOINs to fetch related data in a single query
- **Pagination**: For large result sets, use `LIMIT` and `OFFSET`
- **Caching**: TanStack React Query caches API responses in the frontend

## Backup and Recovery

Supabase provides automatic backups:

- Free tier: Daily backups, 7-day retention
- Pro tier: Continuous backups with point-in-time recovery

Export data manually:

```sql
-- Export transcripts as JSON
SELECT json_agg(t) FROM transcripts t;

-- Export commissioners
SELECT json_agg(c) FROM commissioners c;
```
