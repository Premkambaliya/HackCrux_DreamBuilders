# AI Conversation Intelligence & Action Platform

> **HackCrux Hackathon | Team DreamBuilders**

Turn raw sales calls into revenue decisions — automatically.

---

## Table of Contents

- [Overview](#overview)
- [How It Works — User Flow](#how-it-works--user-flow)
- [Website Flow Diagram](#website-flow-diagram)
- [How Modules Connect](#how-modules-connect)
- [Live Features](#live-features)
- [AI Pipeline](#ai-pipeline)
- [Tech Stack](#tech-stack)
- [Libraries & Dependencies](#libraries--dependencies)
- [Folder Structure](#folder-structure)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)
- [API Reference](#api-reference)
- [Screenshots / Pages](#screenshots--pages)
- [Architecture](#architecture)
- [Team](#team)

---

## Overview

**SalesIQ** is a full-stack AI-powered platform that automatically analyzes sales call recordings or transcripts and converts them into structured, actionable intelligence. It extracts deal probabilities, salesperson performance scores, customer objections, buying signals, competitor insights, coaching feedback, and auto-drafted follow-up emails — all in one pass using state-of-the-art LLMs.

Built for sales managers, team leads, and revenue operations teams who want real-time visibility into every customer conversation without manual review.

---

## How It Works — User Flow

**Step-by-step journey of a user from signup to actionable intelligence:**

```
┌─────────────────────────────────────────────────────────────────────┐
│                        USER JOURNEY                                  │
└─────────────────────────────────────────────────────────────────────┘

  STEP 1 ── Visit Landing Page (/)
  │         └─ Learn features → Click "Get Started"
  │
  STEP 2 ── Signup / Login (/signup or /login)
  │         └─ JWT token issued → stored in localStorage
  │         └─ User role assigned: admin OR employee
  │
  STEP 3 ── Dashboard (/dashboard)
  │         └─ See live analytics, pipeline health, risk radar
  │         └─ Admin: manage Employees & Products via Sidebar
  │
  STEP 4 ── Analyze a Call (/dashboard/analyze)
  │         │
  │         ├─ Option A: Upload Audio/Video File
  │         │   └─ Drag & drop mp3/wav/mp4/mov etc.
  │         │   └─ File → Multer → MongoDB (status: "uploaded")
  │         │   └─ Groq Whisper transcribes → MongoDB (status: "transcribed")
  │         │
  │         └─ Option B: Paste Text Transcript
  │             └─ Text → MongoDB (status: "transcribed", skip Whisper)
  │
  STEP 5 ── AI Analysis Runs Automatically
  │         └─ LLaMA 3.3-70B reads transcript
  │         └─ Returns 30+ fields in structured JSON
  │         └─ Saved to MongoDB (status: "analyzed")
  │         └─ Redirect → Call Detail page
  │
  STEP 6 ── View Call Detail (/dashboard/call/:id)
  │         ├─ Tab 1: Action Center, Risk Banner, Objection Playbook
  │         ├─ Tab 2: Deal Probability, Skill Scores, Key Moments,
  │         │         Battle Card, Email Draft, Coaching Notes
  │         └─ Tab 3: Full Transcript, Edit Metadata, Download PDF
  │
  STEP 7 ── Explore Intelligence Layers
  │         ├─ All Calls    → Search, filter, compare calls
  │         ├─ Insights     → Aggregated signals from all calls
  │         ├─ Top Deals    → Highest probability opportunities
  │         ├─ High Risk    → At-risk deals needing attention
  │         ├─ Employees    → Rep performance & coaching (Admin)
  │         └─ Products     → Product health from customer feedback (Admin)
  │
  STEP 8 ── Take Action
            ├─ Copy follow-up email from email draft
            ├─ Use Objection Playbook scripts on next call
            ├─ Download PDF report and share with manager
            └─ Coach rep using AI-generated coaching actions
```

---

## Website Flow Diagram

```
╔══════════════════════════════════════════════════════════════════════╗
║                         SalesIQ WEBSITE FLOW                        ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║   [ Browser ]                                                        ║
║       │                                                              ║
║       ▼                                                              ║
║  ┌─────────────┐    Not Logged In     ┌───────────────────────┐     ║
║  │ Landing Page│ ──────────────────►  │  Login / Signup Page  │     ║
║  │    (/)      │                      │ POST /api/auth/login   │     ║
║  └─────────────┘                      └──────────┬────────────┘     ║
║                                                  │ JWT Token         ║
║                                                  ▼                   ║
║                                       ┌──────────────────────┐      ║
║                                       │  Dashboard (/dashboard│      ║
║                                       │  GET /api/dashboard/  │      ║
║                                       │  analytics + calls +  │      ║
║                                       │  competitors + risk   │      ║
║                                       └──────────┬───────────┘      ║
║                                                  │                   ║
║              ┌───────────────────────────────────┼──────────────┐   ║
║              │                                   │              │   ║
║              ▼                                   ▼              ▼   ║
║   ┌─────────────────┐              ┌──────────────────┐  ┌─────────┐║
║   │  Analyze Call   │              │  Call List       │  │Insights │║
║   │  (/analyze)     │              │  (/calls)        │  │(/insigh)│║
║   │                 │              │  GET /dashboard/ │  │         │║
║   │ Upload Audio ───┼──►Multer     │  calls           │  │Aggregat │║
║   │ OR Paste Text   │  File Store  └────────┬─────────┘  │ed from  │║
║   └────────┬────────┘       │               │            │all calls│║
║            │                ▼               ▼            └─────────┘║
║            │      ┌──────────────┐  ┌───────────────┐              ║
║            │      │ POST /audio/ │  │  Call Detail  │              ║
║            │      │ upload       │  │  /call/:id    │              ║
║            │      │ UUID callId  │  │               │              ║
║            │      └──────┬───────┘  │ Tab 1: Actions│              ║
║            │             │          │ Tab 2: Analysis│             ║
║            │             ▼          │ Tab 3: Transcrp│             ║
║            │    ┌──────────────────┐│ + PDF Download │             ║
║            │    │ POST /transcript/││               │              ║
║            │    │ transcribe/:id   │└───────────────┘              ║
║            │    │ Groq Whisper    │                                ║
║            │    │ → saves text    │                                ║
║            │    └──────┬──────────┘                                ║
║            │           │                                            ║
║            └───────────┤                                            ║
║                        ▼                                            ║
║               ┌──────────────────┐                                  ║
║               │ POST /ai/analyze │                                  ║
║               │ /:callId         │                                  ║
║               │ Groq LLaMA 3.3   │                                  ║
║               │ 70B Versatile    │                                  ║
║               │ → 30+ AI fields  │                                  ║
║               │ → saved MongoDB  │                                  ║
║               └──────┬───────────┘                                  ║
║                      │ Redirect                                      ║
║                      ▼                                               ║
║            ┌──────────────────────────────────────────────┐         ║
║            │              Call Detail Page                │         ║
║            │  /dashboard/call/:callId                     │         ║
║            │                                              │         ║
║            │  ┌────────────┐ ┌───────────┐ ┌──────────┐  │         ║
║            │  │ Overview & │ │   Deep    │ │Transcript│  │         ║
║            │  │  Actions   │ │ Analysis  │ │& Details │  │         ║
║            │  │            │ │           │ │          │  │         ║
║            │  │• Action Ctr│ │• Deal Prob│ │• Full    │  │         ║
║            │  │• Risk Bnr  │ │• 11 Skills│ │  Text    │  │         ║
║            │  │• Objection │ │• Phse Scrs│ │• Edit    │  │         ║
║            │  │  Playbook  │ │• Key Momnt│ │• PDF     │  │         ║
║            │  │            │ │• BattlCrd │ │  Report  │  │         ║
║            │  │            │ │• Email Dft│ │          │  │         ║
║            │  └────────────┘ └───────────┘ └──────────┘  │         ║
║            └──────────────────────────────────────────────┘         ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## How Modules Connect

```
╔══════════════════════════════════════════════════════════════╗
║              MODULE INTERCONNECTION MAP                      ║
╠══════════════════════════════════════════════════════════════╣
║                                                              ║
║  ┌─────────────────────────────────────────────────────┐    ║
║  │                   MongoDB Database                   │    ║
║  │   collections: users · calls · products · companies  │    ║
║  └──────────────────────────┬──────────────────────────┘    ║
║                             │ shared data layer              ║
║    ┌────────────┬───────────┼────────────┬────────────┐     ║
║    │            │           │            │            │     ║
║    ▼            ▼           ▼            ▼            ▼     ║
║ ┌──────┐  ┌─────────┐ ┌─────────┐ ┌──────────┐ ┌────────┐ ║
║ │ Auth │  │  Audio  │ │ Transcr │ │    AI    │ │ Dashbd │ ║
║ │      │  │ Module  │ │ Module  │ │ Analysis │ │ Module │ ║
║ │ JWT  │  │         │ │         │ │          │ │        │ ║
║ │ User │  │callId   │─►transcript│─►aiInsghts│─►reports │ ║
║ │ Auth │  │ UUID    │ │ text    │ │ 30+ flds │ │ pdf dld │ ║
║ └──┬───┘  └─────────┘ └─────────┘ └──────────┘ └────────┘ ║
║    │                                                         ║
║    ▼ companyId + employeeId (from JWT)                       ║
║    │                                                         ║
║    ├──────────────────────────────────┐                      ║
║    │                                  │                      ║
║    ▼                                  ▼                      ║
║ ┌──────────────────┐       ┌──────────────────────┐         ║
║ │ Employee         │       │  Product             │         ║
║ │ Intelligence     │       │  Intelligence        │         ║
║ │                  │       │                      │         ║
║ │ Reads: calls     │       │  Reads: calls        │         ║
║ │ Groups by:       │       │  Groups by:          │         ║
║ │  employeeId      │       │   productId / name   │         ║
║ │ Aggregates:      │       │  Aggregates:         │         ║
║ │  strengths       │       │   objections         │         ║
║ │  weaknesses      │       │   buying signals     │         ║
║ │  missed opps     │       │   competitors        │         ║
║ │  sentiments      │       │   sentiment          │         ║
║ │  avg scores      │       │   product rating     │         ║
║ └──────────────────┘       └──────────────────────┘         ║
║                                                              ║
║  ┌─────────────────────────────────────────────────────┐    ║
║  │               How Data Flows Per Call                │    ║
║  │                                                      │    ║
║  │  Upload ──► calls.status = "uploaded"               │    ║
║  │     │                                               │    ║
║  │     ▼                                               │    ║
║  │  Whisper ──► calls.transcript = "..."               │    ║
║  │             calls.status = "transcribed"            │    ║
║  │     │                                               │    ║
║  │     ▼                                               │    ║
║  │  LLaMA ───► calls.aiInsights = { ...30+ fields }   │    ║
║  │             calls.dealProbability = 74              │    ║
║  │             calls.sentiment = "positive"            │    ║
║  │             calls.salespersonRating = 8.2           │    ║
║  │             calls.riskLevel = "medium"              │    ║
║  │             calls.status = "analyzed"               │    ║
║  │     │                                               │    ║
║  │     ▼                                               │    ║
║  │  Dashboard reads ──► shows in all Intelligence      │    ║
║  │  pages (Employee Intel, Product Intel, Risk,        │    ║
║  │  Insights, Top Deals, High Risk, Call List)         │    ║
║  └─────────────────────────────────────────────────────┘    ║
╚══════════════════════════════════════════════════════════════╝
```

### Role-Based Access Flow

```
  After Login
       │
       ├─── Role: employee ──────────────────────────────────┐
       │         │                                            │
       │         ├─ Dashboard (own calls only)               │
       │         ├─ Analyze Call (upload own calls)          │
       │         ├─ Call List (own calls)                    │
       │         ├─ Insights (own data)                      │
       │         ├─ Top Deals (own data)                     │
       │         ├─ High Risk (own data)                     │
       │         └─ Profile                                  │
       │                                                      │
       └─── Role: admin ─────────────────────────────────────┘
                 │
                 ├─ Everything employee can see (company-wide)
                 ├─ Employees page (view all reps + add new)
                 ├─ Products page (manage product catalog)
                 ├─ Employee Intelligence (any employee)
                 └─ Product Intelligence (any product)
```

### How SalesIQ Is Useful — Value Map

```
 WHO USES IT          WHAT THEY DO                  WHAT THEY GAIN
 ─────────────────────────────────────────────────────────────────
 Sales Rep        →  Upload call recording       →  Instant feedback on
                                                    performance + email draft

 Sales Manager    →  View Employee Intelligence  →  Know which reps need
                     for each rep                   coaching & on what skills

 Product Manager  →  View Product Intelligence   →  Real customer objections
                     for each product               & improvement ideas

 Revenue Ops      →  Monitor Dashboard           →  Pipeline health,
                     + Risk Radar                   deal probability trends

 Team Lead        →  Top Deals + High Risk       →  Know where to focus
                     pages                          energy this week

 Any Stakeholder  →  Download PDF Report         →  Share polished call
                     for any call                   analysis in 1 click
```

---

## Live Features

### Core Intelligence
- Upload **audio/video files** (mp3, wav, m4a, ogg, webm, flac, aac, mp4) or **paste text transcripts**
- Auto-transcription using **Groq Whisper large-v3-turbo**
- Deep AI analysis using **LLaMA 3.3 70B** via Groq — extracts 30+ intelligence dimensions per call
- Auto-generated **follow-up email draft** (subject, body, tone, CTA)
- One-click **PDF report download** (fully branded, multi-section A4 layout)

### Dashboard & Analytics
- **Revenue Performance Command Center** — real-time pipeline metrics
- Sentiment distribution (Positive / Neutral / Negative) with visual donut chart
- Average Deal Probability, Rep Ratings, Customer Engagement scores
- Competitor Mention frequency analysis
- Competitor Advantage tracking

### Risk Management
- **Risk Radar** — auto-flags at-risk deals by deal probability, sentiment, and risk level
- **High Risk Page** — all deals with probability < 40% or negative sentiment in one view
- **Deal Recovery Plans** — AI-generated steps to bring deals back on track

### Employee Intelligence
- Per-employee performance drilldown
- Aggregated strengths, weaknesses, missed opportunities
- Sentiment distribution per rep
- Call-type breakdown
- AI-generated natural language performance summary
- Recent calls table per employee

### Product Intelligence
- Per-product analysis aggregated from all customer conversations
- Top objections, buying signals, positive points, and improvement suggestions
- Competitor breakdown per product
- **Overall Product Rating** (computed from real customer sentiment)
- Call-type distribution per product

### Call Management
- **Conversation Library** — searchable, filterable call list (by sentiment, type, text)
- **CallDetail** — full 3-tab deep-dive view per call
- **Top Deals** — calls ranked by deal probability
- Action Center with copyable response templates and deadlines
- Objection Playbook with per-objection response scripts
- Competitive Battle Card per call

### Team Management (Admin)
- Add / manage employees with role-based access
- Add / manage products with category and description
- Role-based UI — admin-only sections hidden from reps

---

## AI Pipeline

```
Audio / Text Input
        │
        ▼
[ 1. Upload & Store ]
  Multer → UUID callId → MongoDB
        │
        ▼
[ 2. Transcription ]
  Groq Whisper large-v3-turbo
  (ffmpeg fallback for video extraction)
  Retry: 3x with exponential backoff
        │
        ▼
[ 3. AI Analysis ]
  Groq LLaMA-3.3-70b-versatile
  System Prompt → 30+ structured fields
  → Normalized & saved to MongoDB
        │
        ▼
[ 4. Intelligence Layers ]
  Dashboard / Employee / Product / Risk APIs
        │
        ▼
[ 5. PDF Report Generation ]
  PDFKit → branded A4 multi-section report
```

### AI Output Fields (per call)

| Category | Fields |
|---|---|
| **Summary** | `summary`, `callTitle`, `callType` |
| **Deal** | `dealProbability` (0–100), `followUpRecommendation` |
| **Customer** | `sentiment`, `customerEngagementScore`, `urgencyLevel` |
| **Signals** | `buyingSignals[]`, `objections[]`, `positivePoints[]` |
| **Competition** | `competitors[]`, `competitorAdvantages[]`, `competitiveBattleCard` |
| **Rep Performance** | `salespersonRating` /10, 11 skill scores, 5 call phase scores |
| **Tone Analysis** | Tone breakdown, tone shifts, emotional intelligence score |
| **Conversation** | Talk ratio, question counts, keyTopics, painPoints, actionItems |
| **Risk** | `riskLevel` (critical/high/medium/low), `dealRecoveryPlan` |
| **Coaching** | `coachingActions`, `topSkillGap`, `managerSummary` |
| **Email** | `emailDraft` (subject, body, tone, CTA) |
| **Actions** | `actionCenter[]` — 3-5 prioritized actions with templates |
| **Objections** | `objectionPlaybook[]` — per-objection response scripts |
| **Key Moments** | 3-6 pivotal call moments with impact descriptions |
| **Product** | `productName`, `improvementsNeeded[]` |

---

## Tech Stack

### Backend
| Technology | Version | Purpose |
|---|---|---|
| Node.js | 18+ | Runtime |
| Express.js | ^5.2.1 | HTTP server & routing |
| MongoDB | ^7.1.0 | Database (native driver, no Mongoose) |
| Groq SDK | ^0.37.0 | LLM inference (Whisper + LLaMA) |
| JSON Web Tokens | ^9.0.0 | Authentication |
| bcryptjs | ^2.4.3 | Password hashing |
| Multer | ^2.1.1 | File uploads |
| PDFKit | ^0.17.2 | PDF report generation |
| uuid | ^13.0.0 | Unique call ID generation |
| dotenv | ^17.3.1 | Environment variable management |
| cors | ^2.8.6 | Cross-origin resource sharing |
| ffmpeg | system | Audio extraction from video files |
| nodemon | ^3.1.14 | Dev auto-reload |

### Frontend
| Technology | Version | Purpose |
|---|---|---|
| React | ^19.2.4 | UI framework |
| Vite | ^7.3.1 | Build tool & dev server |
| React Router DOM | ^7.13.1 | Client-side routing |
| Tailwind CSS | ^4.2.1 | Utility-first CSS framework |
| Recharts | ^3.8.0 | Data visualization (pie charts) |
| Lucide React | ^0.577.0 | Icon library |
| @vitejs/plugin-react | ^5.2.0 | Vite React plugin |
| @tailwindcss/vite | ^4.2.1 | Tailwind Vite integration |

### AI / Cloud Services
| Service | Model | Use Case |
|---|---|---|
| Groq Cloud | `whisper-large-v3-turbo` | Audio → Text transcription |
| Groq Cloud | `llama-3.3-70b-versatile` | Call analysis & intelligence |

---

## Libraries & Dependencies

### Backend — `package.json`

```json
{
  "dependencies": {
    "bcryptjs": "^2.4.3",
    "cors": "^2.8.6",
    "dotenv": "^17.3.1",
    "express": "^5.2.1",
    "groq-sdk": "^0.37.0",
    "jsonwebtoken": "^9.0.0",
    "mongodb": "^7.1.0",
    "multer": "^2.1.1",
    "pdfkit": "^0.17.2",
    "uuid": "^13.0.0"
  },
  "devDependencies": {
    "nodemon": "^3.1.14"
  }
}
```

### Frontend — `package.json`

```json
{
  "dependencies": {
    "lucide-react": "^0.577.0",
    "react": "^19.2.4",
    "react-dom": "^19.2.4",
    "react-router-dom": "^7.13.1",
    "recharts": "^3.8.0"
  },
  "devDependencies": {
    "@tailwindcss/vite": "^4.2.1",
    "@vitejs/plugin-react": "^5.2.0",
    "tailwindcss": "^4.2.1",
    "vite": "^7.3.1"
  }
}
```

---

## Folder Structure

```
HackCrux_DreamBuilders/
│
├── README.md                          ← You are here
│
├── Backend/
│   ├── app.js                         ← Express app setup, CORS, all route mounting
│   ├── server.js                      ← DB connect, server start, graceful shutdown
│   ├── package.json
│   │
│   ├── config/
│   │   └── db.js                      ← MongoDB connection (connectDB / disconnectDB)
│   │
│   ├── middlewares/
│   │   ├── auth.middleware.js          ← JWT verify middleware
│   │   └── upload.middleware.js        ← Multer config (audio + text uploads)
│   │
│   ├── modules/
│   │   ├── auth/
│   │   │   ├── auth.controller.js      ← signup, login, profile, update, delete, logout
│   │   │   ├── auth.routes.js
│   │   │   ├── auth.service.js
│   │   │   └── user.model.js           ← MongoDB users collection (native driver)
│   │   │
│   │   ├── audio/
│   │   │   ├── audio.controller.js     ← uploadAudio, uploadText, getAudio, getAllAudio
│   │   │   ├── audio.model.js          ← MongoDB calls collection
│   │   │   ├── audio.routes.js
│   │   │   └── audio.service.js        ← addTranscript, addAIInsights, getCallStats
│   │   │
│   │   ├── transcription/
│   │   │   ├── transcription.controller.js
│   │   │   ├── transcription.routes.js
│   │   │   └── transcription.service.js ← Groq Whisper, ffmpeg extraction, retry logic
│   │   │
│   │   ├── ai-analysis/
│   │   │   ├── ai.controller.js        ← analyzeCall, getInsights
│   │   │   ├── ai.routes.js
│   │   │   └── ai.service.js           ← Groq LLaMA, 30+ field extraction, normalizers
│   │   │
│   │   ├── dashboard/
│   │   │   ├── dashboard.controller.js ← getCalls, analytics, filter, risk, report
│   │   │   ├── dashboard.routes.js
│   │   │   └── dashboard.service.js    ← getAllCalls, analytics, PDF report builder
│   │   │
│   │   ├── employee-intelligence/
│   │   │   ├── ei.controller.js        ← overviewHandler, intelligenceHandler
│   │   │   ├── ei.routes.js
│   │   │   └── ei.service.js           ← per-employee aggregation, performance summary
│   │   │
│   │   ├── product-intelligence/
│   │   │   ├── pi.controller.js        ← overviewHandler, intelligenceHandler
│   │   │   ├── pi.routes.js
│   │   │   └── pi.service.js           ← per-product aggregation, rating computation
│   │   │
│   │   ├── products/
│   │   │   ├── product.controller.js
│   │   │   ├── product.model.js        ← MongoDB products collection
│   │   │   ├── product.routes.js
│   │   │   └── product.service.js      ← createProduct, getProductsByCompany
│   │   │
│   │   ├── users/
│   │   │   ├── user.controller.js      ← getEmployees, addEmployee
│   │   │   ├── user.routes.js
│   │   │   └── user.service.js
│   │   │
│   │   └── company/
│   │       └── company.model.js        ← MongoDB companies collection
│   │
│   └── utils/
│       └── fileValidator.js            ← Audio/video file type & size validation
│
└── Frontend/
    ├── index.html
    ├── vite.config.js
    ├── eslint.config.js
    ├── package.json
    │
    ├── public/
    │
    └── src/
        ├── App.jsx                     ← Router, auth state, protected routes
        ├── App.css
        ├── main.jsx
        ├── index.css
        │
        ├── api/
        │   └── api.js                  ← Centralized API client, normalizeCalls()
        │
        ├── lib/
        │   └── auth.js                 ← persistAuth, clearStoredAuth, fetchCurrentUser
        │
        ├── assets/
        │
        ├── components/
        │   ├── Sidebar.jsx             ← Collapsible nav, mobile overlay, role-aware links
        │   └── Topbar.jsx              ← Top navigation bar
        │
        └── pages/
            ├── LandingPage.jsx         ← Marketing landing page (SalesIQ homepage)
            ├── Dashboard.jsx           ← Revenue Performance Command Center
            ├── AnalyzeCall.jsx         ← 4-step upload & analysis pipeline UI
            ├── CallList.jsx            ← Searchable/filterable conversation library
            ├── CallDetail.jsx          ← Full 3-tab call analysis view
            ├── Insights.jsx            ← Aggregated signal insights across all calls
            ├── TopDeals.jsx            ← Top 10 deals by deal probability
            ├── HighRisk.jsx            ← At-risk deals (prob < 40% or negative sentiment)
            ├── Employees.jsx           ← Admin: employee list + inline intelligence view
            ├── Products.jsx            ← Admin: product list + inline intelligence view
            ├── EmployeeIntelligence.jsx ← Per-employee deep performance drilldown
            ├── ProductIntelligence.jsx  ← Per-product deep customer intelligence view
            ├── Profile.jsx             ← User profile & settings
            └── Auth/
                ├── Login.jsx
                └── Signup.jsx
```

---

## Getting Started

### Prerequisites

- Node.js 18+
- MongoDB (local or MongoDB Atlas)
- Groq API key (free at [console.groq.com](https://console.groq.com))
- ffmpeg (optional — required only for video file transcription)

### 1. Clone the Repository

```bash
git clone <repo-url>
cd HackCrux_DreamBuilders
```

### 2. Backend Setup

```bash
cd Backend
npm install
```

Create a `.env` file in the `Backend/` directory (see [Environment Variables](#environment-variables)).

```bash
npm run dev        # Development with nodemon
# or
npm start          # Production
```

Backend runs on `http://localhost:5000`

### 3. Frontend Setup

```bash
cd Frontend
npm install
npm run dev
```

Frontend runs on `http://localhost:5173`

---

## Environment Variables

Create `Backend/.env` with the following:

```env
# Server
PORT=5000
NODE_ENV=development

# MongoDB
MONGODB_URI=mongodb://localhost:27017/salesiq
# or MongoDB Atlas:
# MONGODB_URI=mongodb+srv://<user>:<password>@cluster.mongodb.net/salesiq

# Authentication
JWT_SECRET=your_super_secret_jwt_key_here
JWT_EXPIRES_IN=7d

# Groq AI
GROQ_API_KEY=your_groq_api_key_here
```

---

## API Reference

### Auth
| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/auth/signup` | Register new user |
| POST | `/api/auth/login` | Login & receive JWT |
| GET | `/api/auth/profile` | Get current user profile |
| PUT | `/api/auth/profile` | Update profile |
| DELETE | `/api/auth/account` | Delete account |
| POST | `/api/auth/logout` | Logout |
| GET | `/api/auth/verify` | Verify JWT token |

### Audio Upload
| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/audio/upload` | Upload audio/video file |
| POST | `/api/audio/upload-text` | Submit text transcript |
| GET | `/api/audio/:callId` | Get call by ID |
| GET | `/api/audio/all` | Get all calls |

### Transcription
| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/transcription/transcribe/:callId` | Transcribe audio to text (Groq Whisper) |

### AI Analysis
| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/ai/analyze/:callId` | Run full AI analysis on call |
| GET | `/api/ai/insights/:callId` | Get stored AI insights |

### Dashboard
| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/dashboard/calls` | Get all analyzed calls |
| GET | `/api/dashboard/call/:callId` | Get single call details |
| GET | `/api/dashboard/analytics` | Get aggregate analytics |
| GET | `/api/dashboard/filter` | Filter calls by criteria |
| GET | `/api/dashboard/competitors` | Get competitor intelligence |
| GET | `/api/dashboard/risk-radar` | Get at-risk deals |
| GET | `/api/dashboard/report/:callId` | Download PDF report |
| PUT | `/api/dashboard/call/:callId` | Update call metadata |

### Employee Intelligence
| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/employee-intelligence/overview` | All employees summary |
| GET | `/api/employee-intelligence/:employeeId` | Deep per-employee intelligence |

### Product Intelligence
| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/product-intelligence/overview` | All products summary |
| GET | `/api/product-intelligence/:productId` | Deep per-product intelligence |

### Products & Users (Admin)
| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/products` | Get company products |
| POST | `/api/products` | Add new product |
| GET | `/api/users/employees` | Get all employees |
| POST | `/api/users/employees` | Add new employee |

---

## Screenshots / Pages

| Page | Description |
|---|---|
| **Landing Page** | Marketing homepage — hero, features, how it works |
| **Dashboard** | Revenue command center with live analytics |
| **Analyze Call** | 4-step pipeline with drag-and-drop upload |
| **Call Detail** | Full 3-tab analysis: Overview, Deep Analysis, Transcript |
| **Call List** | Searchable & filterable conversation library |
| **Insights** | Aggregated buying signals, objections, competitors |
| **Top Deals** | Ranked opportunities by deal probability |
| **High Risk** | At-risk calls with recovery intelligence |
| **Employees** | Employee roster + inline performance drilldown |
| **Employee Intelligence** | 11 skill scores, sentiment, coaching summary |
| **Products** | Product catalog + inline intelligence view |
| **Product Intelligence** | Objections, signals, competitors, product rating |
| **Profile** | User settings & account management |

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     React Frontend                       │
│  Vite · React Router · Tailwind CSS · Recharts          │
└────────────────────────┬────────────────────────────────┘
                         │ REST API + JWT
┌────────────────────────▼────────────────────────────────┐
│                   Express Backend                        │
│                                                         │
│  ┌──────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │  Auth    │  │    Audio     │  │  Transcription   │  │
│  │  Module  │  │    Module    │  │     Module       │  │
│  └──────────┘  └──────────────┘  └────────┬─────────┘  │
│                                            │            │
│  ┌─────────────────────┐        ┌──────────▼─────────┐  │
│  │   AI Analysis       │◄───────│   Groq Whisper     │  │
│  │   Module            │        │   large-v3-turbo   │  │
│  └──────────┬──────────┘        └────────────────────┘  │
│             │ LLaMA 3.3-70B                              │
│  ┌──────────▼──────────────────────────────────────┐    │
│  │              Dashboard · Employee               │    │
│  │              Intelligence · Product             │    │
│  │              Intelligence · Risk Radar          │    │
│  └─────────────────────────────────────────────────┘    │
└────────────────────────┬────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────┐
│                     MongoDB                              │
│  Collections: users · calls · products · companies      │
└─────────────────────────────────────────────────────────┘
```

### Key Design Decisions

- **Module-based architecture** — each domain (audio, AI, dashboard, etc.) is a self-contained module with its own controller, service, and routes
- **Groq for inference** — chosen for speed and free tier availability during hackathon
- **JWT-based auth** — stateless, scalable, stored in localStorage with role-based access
- **Company-scoped data** — all queries filter by `companyId` from JWT; multi-tenancy built in from day one
- **Retry logic on Whisper** — 3 retries with exponential backoff for unstable network edge cases
- **PDF reports via PDFKit** — zero external service dependency for report generation

---

## Team

**Team DreamBuilders** — HackCrux Hackathon

> Built with passion to make every sales conversation count.

---

## License

This project was built for the **HackCrux Hackathon**. All rights reserved by Team DreamBuilders.
