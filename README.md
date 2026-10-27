# JusticeMAP

AI-powered wrongful conviction analysis platform for parole hearing transcripts. Built for an innocence center to help volunteers identify and prioritize potential wrongful conviction cases.

## What It Does

JusticeMAP combines AI analysis with multi-user case management to streamline the review of parole hearing transcripts. Volunteers can quickly identify cases with strong innocence signals, collaborate on case reviews, and generate comprehensive intake forms and memos for legal teams.

**For whom:** Innocence center volunteers, legal advocates, and case reviewers analyzing California parole hearings.

**The problem:** Reviewing parole transcripts manually is time-consuming. Important innocence indicators can be buried in hundreds of pages. Extracting structured data for legal forms takes hours per case.

**The solution:** Automated AI analysis surfaces the most compelling innocence signals, while smart form extraction converts transcripts into structured legal documents in seconds.

## Key Features

### AI Innocence Detection

The platform analyzes transcripts to detect four types of signals:

1. **Explicit claims** - Direct statements of innocence ("I didn't do it", "I'm innocent")
2. **Contextual signals** - Evidence-based indicators like coerced confessions, alibi evidence, witness recantations, or evidence gaps
3. **Implicit signals** - Maintaining innocence despite consequences (refusing plea deals, accepting harsher sentences)
4. **Bias language** - Institutional patterns like "lack of insight" (often used when someone refuses to admit guilt)

Every client card displays an AI-generated summary of the most compelling innocence indicator from their transcript. No more generic "available for review" text.

**How it works:**
- Transcripts are intelligently chunked to preserve speaker context
- Each chunk is analyzed by Gemini 2.0 Flash Lite
- System validates all findings with exact quotes, line numbers, and page citations
- Results are ranked by confidence and deduplicated across chunks

### Multi-User Case Management

- Personal kanban board tracks each volunteer's cases through four stages: Assigned to Me, Under Review, Forms Generated, and Commissioner Panel Scheduled
- Smart assignment workflow lets volunteers claim cases with an "Assign to Me" button
- Each volunteer sees only their assigned cases on their dashboard
- AI Priority Picks sidebar ranks unassigned cases by innocence strength, commissioner bias risk, urgency, and assignment status

### Commissioner Bias Analysis

The system maintains a database of 23+ California Board of Parole Hearings commissioners with detailed backgrounds. For each case, it:

- Identifies which commissioners were on the panel
- Categorizes backgrounds (Law Enforcement, Prosecution, Legal/Mixed, Mental Health, Parole Administration)
- Detects 15 bias language patterns in commissioner statements
- Assesses bias risk based on panel composition
- Links to LinkedIn or CDCR profiles for each commissioner

This helps volunteers and attorneys prepare for hearings and identify high-risk panel compositions.

### Automated Form Generation

The form processor extracts 45+ structured fields from transcripts to auto-fill client intake forms:

- Personal info (name, CDCR number, DOB, facility, education, military service)
- Attorney information (trial counsel, appellate counsel)
- Case details (crime date, conviction, sentence, appeals, parole eligibility)
- Family contacts and support information
- Evidence of wrongful conviction across 20 categories

The system uses multi-pattern matching (3-7 extraction attempts per field) and generates downloadable memos with exact citations. This saves volunteers 20-40 minutes per case.

## Tech Stack

**Frontend:**
- React 18 with TypeScript 5.8 for type safety
- Vite 5.4 for fast builds and hot module replacement
- Tailwind CSS 3.4 with shadcn/ui component library
- React Router 6 with lazy loading

**Backend:**
- Supabase (PostgreSQL database, authentication, file storage)
- Google Generative AI (Gemini 2.0 Flash Lite for transcript analysis)
- TanStack React Query for server state management

**Additional:**
- PDF.js for PDF text extraction
- React Hook Form with Zod validation
- Lucide React icons

## Project Structure

```
innocence-grid/
├── src/
│   ├── services/          # Backend logic
│   │   ├── gemini.ts                 # Gemini API integration
│   │   ├── innocenceDetector.ts      # Analysis orchestrator
│   │   ├── transcripts.ts            # Transcript CRUD + PDF storage
│   │   ├── commissioners.ts          # Commissioner lookup and panel analysis
│   │   ├── volunteers.ts             # User management
│   │   ├── formProcessor.ts          # Multi-pattern field extraction
│   │   ├── memoGeneratorEnhanced.ts  # Memo generation with citations
│   │   └── priority.ts               # AI-driven priority scoring
│   ├── utils/             # Utilities
│   │   ├── transcriptChunker.ts      # Intelligent chunking
│   │   ├── speakerParser.ts          # Speaker identification
│   │   └── transcriptPreprocessor.ts # Text cleanup and normalization
│   ├── pages/             # React pages
│   │   ├── Clients.tsx               # Main dashboard with AI insights
│   │   ├── Dashboard.tsx             # Personal kanban board
│   │   ├── TranscriptDetail.tsx      # Analysis view with citations
│   │   ├── Cases.tsx                 # Case analysis with commissioner panels
│   │   ├── CommissionerBreakdown.tsx # Bias analysis by commissioner
│   │   ├── CommissionerProfile.tsx   # Individual commissioner pages
│   │   ├── Transcripts.tsx           # Transcript management
│   │   ├── Upload.tsx                # File upload (PDF and TXT)
│   │   └── Login.tsx                 # Authentication
│   ├── components/        # UI components
│   │   ├── AIRecommendationsSidebar.tsx  # Priority picks
│   │   └── layout/                   # Navigation and layout
│   └── lib/               # Types, config, utils
├── supabase/              # Database setup
│   ├── database-merged-setup.sql
│   ├── database-commissioners-setup.sql
│   ├── populate-commissioners-backgrounds.sql
│   └── README.md
├── docs/                  # Documentation
│   ├── setup.md
│   ├── database.md
│   ├── ai-innocence-detection.md
│   ├── commissioner-analysis.md
│   └── form-and-memo-generator.md
└── README.md
```

## Quick Start

### Prerequisites

- Node.js 18+
- Supabase account (free tier works)
- Google Gemini API key (free at [aistudio.google.com](https://aistudio.google.com/apikey))

### Setup

1. Clone the repository:
```bash
git clone https://github.com/aimiwamy03/innocence-grid.git
cd innocence-grid
npm install
```

2. Set up Supabase:
   - Create a new project at [supabase.com](https://supabase.com)
   - Run the SQL files in `supabase/` (see `supabase/README.md` for order)
   - Create a storage bucket named `transcript-pdfs` (public access)

3. Configure environment variables:
```bash
# Create .env file
VITE_SUPABASE_URL=your-project-url
VITE_SUPABASE_ANON_KEY=your-anon-key
VITE_GEMINI_API_KEY=your-gemini-api-key
```

4. Start the development server:
```bash
npm run dev
```

Visit [http://localhost:5173](http://localhost:5173)

### Available Scripts

- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm run lint` - Run ESLint
- `npm run preview` - Preview production build

## How the AI Analysis Works

1. **Preprocessing** - Transcripts are cleaned (line numbers removed, speaker labels normalized, OCR errors fixed)
2. **Chunking** - Text is split into 3000-4000 character chunks at speaker boundaries to preserve context
3. **AI Analysis** - Gemini 2.0 Flash Lite analyzes each chunk for the four signal types using a specialized prompt
4. **Validation** - Every finding must include exact quotes, line numbers, page numbers, and speaker identification
5. **Aggregation** - Results are combined, deduplicated, ranked by confidence, and stored with citations

The system uses a fallback strategy: AI-generated insights (best), pattern matching (good), or generic text (fallback).

For transcripts, the first 20 visible client cards load AI insights immediately, with remaining insights loading progressively in batches of 5 to optimize performance.

## Team / Credits

JusticeMAP was built as a team hackathon project for an innocence center. This repository is a fork of the original project by [Kaiwen Zhou](https://github.com/kaiwenzhou/innocence-grid).

**Team members:**
- Kaiwen Zhou
- Aimi Wang

The project demonstrates how AI can accelerate the important work of identifying and advocating for wrongfully convicted individuals.

## Documentation

See the `docs/` folder for detailed guides:

- `setup.md` - Installation and troubleshooting
- `database.md` - Schema, migrations, and queries
- `ai-innocence-detection.md` - How the AI analysis works
- `commissioner-analysis.md` - Bias detection and panel analysis
- `form-and-memo-generator.md` - Automated form extraction

## License

This project was built as part of a hackathon for social good. License TBD.
