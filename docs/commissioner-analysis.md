# Commissioner Analysis

This guide explains JusticeMAP's commissioner bias detection system and how it analyzes California Board of Parole Hearings (BPH) commissioners.

## Overview

JusticeMAP maintains a database of 23+ California BPH commissioners with detailed background information. The system analyzes parole transcripts to:

1. Identify which commissioners were on the panel
2. Categorize their professional backgrounds
3. Detect bias language patterns in their statements
4. Assess bias risk based on panel composition

## Commissioner Database

### Data Source

Commissioner information is sourced from:

- [California Department of Corrections BPH website](https://www.cdcr.ca.gov/bph)
- Public LinkedIn profiles
- Official CDCR commissioner profiles
- Appointment records

### Commissioner Fields

Each commissioner record includes:

- **Name**: Full name
- **Background Category**: One of five categories (see below)
- **Background Details**: Career history, education, previous roles
- **Profile URL**: LinkedIn or CDCR profile link
- **Photo URL**: Headshot (when available)
- **Appointment Date**: When appointed to BPH
- **Active Status**: Currently serving or not

### Background Categories

**1. Law Enforcement & Corrections**
- Former police officers, sheriffs, correctional officers
- Career in law enforcement agencies
- Higher bias risk for "tough on crime" stance

**2. Legal, Judicial & Prosecution**
- Former prosecutors, district attorneys
- Career focused on prosecution
- May have conviction-oriented mindset

**3. Legal, Judicial & Mixed Legal**
- Judges, defense attorneys, mixed legal experience
- Balanced legal perspective
- Lower bias risk

**4. Mental Health & Social Services**
- Psychologists, social workers, counselors
- Rehabilitation-focused background
- Lower bias risk, more supportive

**5. Parole Board Administration**
- Career in parole system administration
- Procedural focus
- Variable bias risk

## Bias Language Detection

The system detects **15 bias language patterns** commonly used to justify parole denials:

### Primary Patterns (Most Common)

1. **"lack of insight"** - Refusing to admit guilt
2. **"minimizing"** - Downplaying the offense
3. **"not taking responsibility"** - Not accepting blame
4. **"denial"** - Denying involvement
5. **"lack of remorse"** - Not showing enough regret

### Secondary Patterns

6. **"not suitable"** - Not ready for release
7. **"unrealistic"** - Plans are not believable
8. **"superficial"** - Understanding is shallow
9. **"not credible"** - Don't believe the person
10. **"manipulative"** - Trying to deceive the panel

### Risk Assessment Patterns

11. **"danger to society"** - Public safety threat
12. **"continues to pose risk"** - Still dangerous
13. **"not rehabilitated"** - Has not changed
14. **"risk to public safety"** - Would reoffend
15. **"unresolved issues"** - Problems remain

### How Bias Detection Works

**File**: `src/pages/CommissionerBreakdown.tsx`

1. **Extract commissioner names** from transcript
   - Looks for "PRESIDING COMMISSIONER" sections
   - Parses dialogue for commissioner mentions
   - Validates against database (filters false positives like "Commissioner AND")

2. **Count bias language** for each commissioner
   - Searches their statements for the 15 patterns
   - Tracks frequency and context
   - Associates patterns with specific commissioners

3. **Count innocence language** (12 patterns)
   - didn't do, innocent, wrongly convicted, falsely accused
   - was not there, someone else did, misidentification
   - actually innocent, false testimony, false confession
   - coerced confession, alibi, recant

4. **Calculate bias ratio**
   - Bias statements / Total statements
   - Higher ratio = more bias-oriented language

5. **Display results**
   - Commissioner name and background
   - Bias language count
   - Innocence language count
   - Background category badge
   - Profile link (LinkedIn or CDCR)

## Commissioner Profile Pages

Each commissioner has a dedicated profile page showing:

- **Full name and photo**
- **Background category** (color-coded badge)
- **Detailed background** (career history, education)
- **Profile link** (LinkedIn or CDCR)
- **Appointment date**
- **Statistics across all transcripts:**
  - Total hearings presided over
  - Bias language frequency
  - Innocence language mentions
  - Approval rate (if tracked)

**Route**: `/commissioner/:commissionerId`

## Panel Composition Analysis

For each case, the system analyzes the **commissioner panel** composition:

### Risk Assessment Factors

1. **Number of Law Enforcement backgrounds** (higher risk)
2. **Number of Prosecution backgrounds** (higher risk)
3. **Number of Mental Health backgrounds** (lower risk)
4. **Panel balance** (mixed backgrounds = lower risk)

### Scoring Algorithm

**File**: `src/services/priority.ts`

```
Commissioner Bias Score (0-25 points):
- All Law Enforcement: 25 points (highest risk)
- All Prosecution: 22 points
- Mixed LE/Prosecution: 18 points
- Mixed with Mental Health: 10 points
- All Mental Health: 5 points (lowest risk)
```

This score feeds into the overall **AI Priority Score** for case ranking.

## Commissioner Breakdown Page

The **Commissioner Breakdown** page shows analysis across **all 294 transcripts**:

### Features

1. **Real-time validation logging** (browser console)
   - Reports progress as transcripts are analyzed
   - Shows which commissioners were found
   - Identifies bias and innocence language counts

2. **Commissioner cards** with:
   - Name and background badge
   - Total bias language count
   - Total innocence language count
   - Profile link
   - Hearings presided count

3. **Filtering and sorting** (future enhancement):
   - Filter by background category
   - Sort by bias language frequency
   - Search by commissioner name

### Database Integration

The system uses a **database-first approach**:

1. Load all commissioners from Supabase
2. Parse transcripts for commissioner mentions
3. Match names against database (validates real commissioners)
4. Link commissioners to hearings in `commissioner_hearings` table
5. Track statistics in `commissioner_statistics` table

**File**: `src/services/commissioners.ts`

## Commissioner Data Updates

### Adding a New Commissioner

To add a commissioner to the database:

```sql
INSERT INTO commissioners (
  name,
  background_category,
  background_details,
  profile_url,
  appointed_date,
  is_active
) VALUES (
  'Commissioner Name',
  'Legal, Judicial & Prosecution',
  'Career prosecutor, 15 years DA office...',
  'https://www.linkedin.com/in/example',
  '2020-01-15',
  true
);
```

Or create a patch file in `supabase/patches/`:

- `add-new-commissioner.sql`

See existing examples:
- `add-brenna-kantrovitz.sql`
- `add-robert-barton.sql`

### Updating Commissioner Info

```sql
UPDATE commissioners
SET 
  background_details = 'Updated background...',
  profile_url = 'https://newurl.com'
WHERE name = 'Commissioner Name';
```

## Real-World Usage

### Identifying High-Risk Panels

Look for panels with:

- 2+ Law Enforcement commissioners
- 2+ Prosecution commissioners
- No Mental Health or Social Services representation
- High frequency of bias language (>10 instances)

### Flagging for Review

Cases with high-risk panels should be:

1. **Prioritized** in the AI Priority Picks
2. **Flagged** for attorney review
3. **Documented** in case notes
4. **Tracked** for appeal preparation

### Preparing for Hearings

Before a hearing, volunteers can:

1. Look up commissioners on the scheduled panel
2. Review their backgrounds and past language patterns
3. Prepare responses to common bias language
4. Brief the client on what to expect

## Performance

### Analysis Speed

- Single transcript: 1-2 seconds (commissioner extraction)
- All 294 transcripts: 10-15 seconds (browser console logging)
- Database lookups: <100ms (indexed queries)

### Data Quality

- **23+ commissioners** in database
- **Verified backgrounds** from official sources
- **Valid profile URLs** (LinkedIn or CDCR)
- **Regular updates** as commissioners change

## Troubleshooting

### Commissioners Not Showing

**Check:**
- Database has commissioners table
- `populate-commissioners-backgrounds.sql` was run
- Table has 23+ rows

**Solution:**
```sql
SELECT COUNT(*) FROM commissioners;
-- Should return 23+
```

### Background Showing "Unknown"

**Check:**
- `background_category` and `background_details` columns exist
- Data was properly inserted

**Solution:**
```sql
SELECT name, background_category, background_details 
FROM commissioners 
WHERE background_category IS NULL;
-- Fix any null values
```

### False Positive Commissioner Names

**Common issues:**
- "Commissioner AND" (parsed from "Commissioner X AND Commissioner Y")
- "PRESIDING" (title, not name)
- "DEPUTY" (different role)

**Solution:**
The name validation logic filters these out automatically. Check logs for "Skipping invalid commissioner name" messages.

## Future Enhancements

- Historical trend analysis (has a commissioner's language changed over time?)
- Comparative statistics (how does this commissioner compare to others?)
- Prediction models (likelihood of parole grant based on panel)
- Sentiment analysis of commissioner statements
- Tracking of appeal outcomes by commissioner
