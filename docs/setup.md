# Setup Guide

This guide walks you through setting up JusticeMAP locally for development.

## Prerequisites

- Node.js 18+ ([install with nvm](https://github.com/nvm-sh/nvm))
- A Supabase account (free tier works fine)
- Google Gemini API key ([get one here](https://aistudio.google.com/apikey))

## Installation Steps

### 1. Clone and Install

```bash
git clone https://github.com/aimiwamy03/innocence-grid.git
cd innocence-grid
npm install
```

### 2. Set Up Supabase Database

1. Create a new Supabase project at [supabase.com](https://supabase.com)
2. Open the SQL Editor in your project dashboard
3. Run the SQL files in this order (located in the `supabase/` directory):
   - `database-merged-setup.sql` - Creates core tables (transcripts, predictions, volunteers, case_assignments, case_notes)
   - `database-commissioners-setup.sql` - Creates commissioners tables
   - `populate-commissioners-backgrounds.sql` - Populates 23+ California BPH commissioners
   - `add-pdf-storage.sql` - Adds PDF storage columns
   - `FIX_MISSING_STATUS_COLUMN.sql` - Adds status tracking columns
   - `FIX_VOLUNTEERS_TABLE.sql` - Ensures volunteers table is correct

4. Create a Storage bucket:
   - Go to Storage in the Supabase dashboard
   - Create a new bucket named `transcript-pdfs`
   - Set the bucket to **public** access

### 3. Configure Environment Variables

Create a `.env` file in the project root:

```bash
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your-anon-key-here
VITE_GEMINI_API_KEY=your-gemini-api-key-here
```

**How to get these values:**

- **Supabase URL and Key**: Go to Project Settings > API in your Supabase dashboard
  - Copy "Project URL" as `VITE_SUPABASE_URL`
  - Copy "anon/public" key as `VITE_SUPABASE_ANON_KEY`
- **Gemini API Key**: Visit [Google AI Studio](https://aistudio.google.com/apikey) and create a new API key

### 4. Start Development Server

```bash
npm run dev
```

Visit [http://localhost:5173](http://localhost:5173) (Vite's default port)

## First Steps After Setup

1. **Create a volunteer account** on the login page
2. **Navigate to Clients page** to see the case dashboard
3. **Upload test transcripts** via the Upload page (accepts PDF or TXT files)
4. **View AI-generated insights** on each client card
5. **Assign cases to yourself** using the "Assign to Me" button
6. **Track your cases** in "My Case Pipeline" (the kanban board)

## Available Scripts

- `npm run dev` - Start development server (Vite)
- `npm run build` - Build for production
- `npm run lint` - Run ESLint
- `npm run preview` - Preview production build

## Troubleshooting

### Port Already in Use

If port 5173 is already in use:

```bash
# Kill the process
lsof -ti:5173 | xargs kill -9

# Or use a different port
PORT=3000 npm run dev
```

### AI Insights Not Loading

- Verify `VITE_GEMINI_API_KEY` is set correctly in `.env`
- Check the browser console for API errors
- Note: Insights load progressively (first 20 cards, then batches of 5)
- Free tier limit: 15 requests/minute

### Cases Not Moving Between Kanban Columns

- Ensure all database migrations have been run
- Check that the `status`, `assigned_to`, `assigned_at`, and `form_data` columns exist on the `transcripts` table
- Run `FIX_MISSING_STATUS_COLUMN.sql` if needed

### Commissioner Data Not Showing

- Run `database-commissioners-setup.sql` and `populate-commissioners-backgrounds.sql`
- Verify the `commissioners` table has 23+ rows
- Check that `background_category` and `background_details` columns exist

### PDF Viewing Not Working

- Run `add-pdf-storage.sql` to add PDF columns
- Create the `transcript-pdfs` bucket in Supabase Storage
- Set the bucket to public access
- Re-upload transcripts to generate PDF URLs

## Architecture Overview

```
JusticeMAP/
├── Frontend: React + TypeScript + Vite
├── Styling: Tailwind CSS + shadcn/ui components
├── Database: Supabase (PostgreSQL)
├── Authentication: Supabase Auth
├── Storage: Supabase Storage (PDFs)
├── AI Analysis: Google Gemini 2.0 Flash Lite
└── State Management: TanStack React Query
```

## Tech Stack Details

- **React 18** with **TypeScript 5.8** for type safety
- **Vite 5.4** for fast builds and hot module replacement
- **Tailwind CSS 3.4** for utility-first styling
- **shadcn/ui** for high-quality, accessible UI components
- **React Router 6** with lazy loading for optimal performance
- **Supabase Client** (`@supabase/supabase-js`) for backend operations
- **Google Generative AI** SDK for Gemini API integration
- **TanStack React Query** for server state management
- **React Hook Form** + **Zod** for form validation
- **PDF.js** for PDF text extraction
- **Lucide React** for icons

## Need Help?

Check the other documentation files in this folder:

- `database.md` - Database schema and setup details
- `ai-innocence-detection.md` - How the AI analysis works
- `commissioner-analysis.md` - Commissioner bias detection system
- `form-and-memo-generator.md` - Automated form generation
