# CaseWatch — Nyaya Setu for India

> An open-source civic-tech platform that helps citizens of India track court hearings, navigate government certificate procedures, download official document formats, and stay protected from legal touts — with zero login required.

---

> [!NOTE]
> **This repository is fully open-source.**
> Explore, self-host, fork, or contribute under the project license.

---

## Overview

**CaseWatch** is a bilingual (Hindi/English) legal-aid platform built for citizens of India who lack access to reliable legal information. By eliminating authentication barriers, the platform lets anyone look up their case by CNR number, understand what document they need for a government certificate, download the correct format, and verify whether a notice or tout is legitimate — all from a mobile browser.

The platform acts as a **trusted map**, not a legal replacement. Every output is framed as guidance for the user, not a substitute for a lawyer or court.

---

## Core Features (MVP)

### 1. Court Hearing Tracker (`/track`)

- Enter a 16-character **CNR number** to instantly fetch case details from eCourts
- View next hearing date, current case stage, bench details, and judge name
- Pin multiple CNR numbers to your local browser session (no account needed)
- Interactive chronological timeline of historical orders
- Plain-language Hindi/English summary of the latest order (AI-powered via Gemini, with graceful degradation)
- Case status indicators: `fetching` → `active` → `hearing_imminent` → `disposed` → `error_flagged`

### 2. Government Certificate Guidance (`/documents`)

Step-by-step guidance for obtaining government certificates issued in Haryana. Each certificate page is structured into five sections:

| Section | Content |
| --- | --- |
| **Plain-language explainer** | What this certificate is, who needs it, and why |
| **Before you start** | Checklist of documents and prerequisites |
| **Step-by-step process** | Exact steps with relevant state/central portal links |
| **Realistic timeline** | Processing time, office visit expectations |
| **Fraud alert** | Common scams and what to watch out for |

**Tier 1 Certificates (MVP):** Domicile, Caste (OBC / SC / ST), Income, Birth

**Tier 2 Certificates (MVP):** Death, Character, Marriage, Residence

> All document templates and guidance content are platform-authored. Outputs are framed as **"draft for review"** — not ready-to-file documents.

### 3. Downloadable Certificate Formats (`/formats`)

- Download official, standardized formats for Haryana government certificates
- Files served from **Cloudflare R2** object storage
- Preview layout, required fields, and structural formatting before downloading
- Bilingual labels (Hindi/English) on all format previews

### 4. Fraud & Tout Prevention Alerts (`/verify`)

- **"Verify My Notice"** tool — paste or describe a notice to check if it matches known legitimate court/government formats
- Active alerts for common tout scams targeting litigants across Indian courts
- Verified direct links to official government portals (Saral Haryana, eCourts, NALSA)
- Clear visual distinction between verified and unverified sources

### 5. Verified Portal Directory (`/portals`)

- Curated, regularly verified links to official government and court portals
- No third-party redirects; every link points directly to `.gov.in` or `.nic.in` domains
- Categorized by service type: courts, certificates, legal aid, police
- Covers central government portals (DigiLocker, NALSA, eCourts) and state-level service portals

---

## Case Status Reference

| Status | Meaning |
| --- | --- |
| `fetching` | Querying eCourts using the provided CNR number |
| `active` | Case retrieved; hearing updates being tracked |
| `hearing_imminent` | Hearing scheduled within 48 hours |
| `disposed` | Case closed, dismissed, or settled |
| `error_flagged` | CNR query failed or returned invalid data; retry scheduled |

---

## AI Engine

AI features are powered by the **Google Gemini API** and designed with **graceful degradation** — all core case tracking and document guidance works without AI. AI enhances but never blocks.

| Function | Model | Purpose |
| --- | --- | --- |
| `summarizeOrder()` | `gemini-1.5-pro` / `gemini-2.5-pro` | Plain-language summary of latest court order |
| `suggestDocumentFormat()` | `gemini-1.5-flash` / `gemini-2.5-flash` | Recommend the right certificate format based on user inputs |
| `draftCertificateGuidance()` | `gemini-1.5-pro` | Structured guidance draft for certificate procedures |

All AI outputs are citation-grounded against official source material to prevent hallucination. Generated drafts are clearly labelled as **"AI-generated draft — please verify before submitting."**

---

## Tech Stack

| Layer | Technology |
| --- | --- |
| **Frontend** | Next.js 14 (App Router), TypeScript |
| **Styling** | Tailwind CSS, Framer Motion |
| **Backend** | FastAPI (Python) |
| **Database** | PostgreSQL via Supabase |
| **Caching** | Redis |
| **Task Queue** | Celery |
| **File Storage** | Cloudflare R2 (via AWS S3 SDK) |
| **Court Data (Primary)** | eCourtsIndia Paid API |
| **Court Data (Fallback)** | `bharat-courts` SDK (MIT License) |
| **AI / LLM** | Google Gemini API (`gemini-1.5-pro`, `gemini-2.5-flash`) |
| **Frontend Deployment** | Vercel |
| **Backend Deployment** | Render (Starter Plan) |

> ⚠️ **GPL-3.0 Isolation:** The `openjustice-in/ecourts` library (GPL-3.0) is used **only** in offline seeding scripts. It is never imported into the production FastAPI container to prevent license contamination.

---

## Architecture

```
casewatch/
├── frontend/                        # Next.js App (Vercel)
│   ├── app/
│   │   ├── page.tsx                 # Landing page
│   │   ├── track/                   # CNR hearing tracker
│   │   ├── documents/               # Certificate guidance hub
│   │   ├── formats/                 # Downloadable certificate formats
│   │   ├── verify/                  # Fraud & tout prevention
│   │   └── portals/                 # Verified portal directory
│   ├── components/
│   │   └── shared/                  # Reusable UI components
│   └── lib/
│       ├── gemini.ts                # Gemini API orchestration
│       ├── r2.ts                    # Cloudflare R2 file client
│       ├── supabase.ts              # Supabase client
│       └── utils.ts
│
├── backend/                         # FastAPI App (Render)
│   ├── main.py
│   ├── routers/
│   │   ├── courts.py                # CNR lookup, case tracking
│   │   └── documents.py             # Certificate guidance, format delivery
│   ├── services/
│   │   ├── ecourts.py               # eCourtsIndia API integration
│   │   ├── bharat_courts.py         # Fallback court data layer
│   │   ├── gemini.py                # AI summarization service
│   │   └── r2.py                    # R2 storage operations
│   ├── workers/
│   │   └── celery_tasks.py          # Background hearing sync jobs
│   └── db/
│       └── models.py                # PostgreSQL schema
│
└── scripts/                         # Offline seeding only (GPL-isolated)
    └── seed_ecourts.py              # Uses openjustice-in/ecourts (never in production)
```

---

## API Routes

| Endpoint | Router | Purpose |
| --- | --- | --- |
| `GET /api/courts/cnr/{cnr}` | courts | Fetch case details by CNR number |
| `GET /api/courts/hearings/{cnr}` | courts | Get upcoming and past hearings |
| `GET /api/documents/certificates` | documents | List all available certificate guides |
| `GET /api/documents/certificates/{slug}` | documents | Get full guidance for a certificate type |
| `GET /api/documents/formats` | documents | List downloadable format files from R2 |
| `GET /api/documents/formats/{id}/download` | documents | Serve signed R2 download URL |

---

## Environment Variables

```env
# Supabase
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

# Cloudflare R2
CLOUDFLARE_ACCOUNT_ID=
CLOUDFLARE_R2_ACCESS_KEY_ID=
CLOUDFLARE_R2_SECRET_ACCESS_KEY=
CLOUDFLARE_R2_BUCKET_NAME=
CLOUDFLARE_R2_PUBLIC_URL=

# Google Gemini
GEMINI_API_KEY=

# eCourts API
ECOURTS_API_KEY=

# Redis / Celery (Backend)
REDIS_URL=

# FastAPI
DATABASE_URL=
```

---

## Running Locally

```bash
# Frontend
cd frontend
npm install
cp .env.example .env.local   # fill in your tokens
npm run dev
# → http://localhost:3000

# Backend
cd backend
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
uvicorn main:app --reload
# → http://localhost:8000

# Celery worker (separate terminal)
celery -A workers.celery_tasks worker --loglevel=info
```

---

## Deployment

| Service | Platform | Plan |
| --- | --- | --- |
| Frontend | Vercel | Free |
| Backend (FastAPI) | google cloud | 
| Database | Supabase | Free tier |
| File Storage | Cloudflare R2 | Pay-as-you-go |


---

## Post-MVP Roadmap

| Feature | Version |
| --- | --- |
| SMS hearing reminders (MSG91) | V2 |
| Court timeline visualization | V2 |
| Advocate / lawyer tracker | V2 |
| MapmyIndia court locator | V2 |
| DigiLocker integration | V2 |
| AI order translation (plain Hindi) | V2 |
| Bhashini API (22-language support) | V3 |
| AI legal assistant (RAG + pgvector) | V3 |
| OCR for physical notices | V3 |
| Offline PWA mode | V3 |

---

## Important Constraints

- **No user accounts or login** — all session data stored in browser `localStorage`
- **GPL-3.0 isolation** — `openjustice-in/ecourts` confined to offline scripts only
- **Document liability** — all generated outputs labelled as drafts requiring professional review
- **DigiLocker & Bhashini** — pending MeitY approval, scoped to post-MVP

---

## License

This project is licensed under open-source terms. See the [LICENSE](LICENSE) file for details.

---

*Built for the citizens of India. Inspired by the principle that access to legal information is a right, not a privilege.*
