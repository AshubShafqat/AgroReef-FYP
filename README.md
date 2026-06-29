<p align="center">
  <img src="agroreef-icon-512.png" alt="AgroReef" width="140" height="140" />
</p>

<h1 align="center">AgroReef - Agentic AI Agricultural Advisor</h1>

<p align="center">
  <em>Pakistan's bilingual, voice first, multi-agent farming assistant — built as an installable PWA.</em>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/React-18-61DAFB?style=flat-square&logo=react" />
  <img src="https://img.shields.io/badge/TypeScript-5-3178C6?style=flat-square&logo=typescript" />
  <img src="https://img.shields.io/badge/Vite-5-646CFF?style=flat-square&logo=vite" />
  <img src="https://img.shields.io/badge/Tailwind-3-38BDF8?style=flat-square&logo=tailwindcss" />
  <img src="https://img.shields.io/badge/PWA-Installable-5A0FC8?style=flat-square&logo=pwa" />
  <img src="https://img.shields.io/badge/Backend-PostgreSQL%20%2B%20Edge%20Functions-3FCF8E?style=flat-square" />
</p>

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [App Flow](#app-flow)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Agentic AI Pipeline](#agentic-ai-pipeline)
- [PWA & Installability](#pwa--installability)
- [Email Alert System](#email-alert-system)
- [Database Schema](#database-schema)
- [Edge Functions](#edge-functions)
- [Design System](#design-system)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)
- [Deployment](#deployment)
- [License](#license)

---

## Overview

AgroReef is a mobile-first agricultural Progressive Web App built for Pakistani farmers. It runs an **agentic AI pipeline** that fuses live GPS, weather, field records, market prices, and vision input to produce grounded farming advice through natural conversation in Urdu or English.

### Key Highlights
- **7-agent pipeline** (Supervisor, Vision, Weather, Market, Critic, Reasoning, Executive) coordinated by an OPARD loop.
- **Voice-first** — farmers speak in Urdu/English; replies are spoken back via ElevenLabs (English) and Web Speech API (Urdu, `ur-PK`, strict no-Hindi policy).
- **Live camera mode** — 25-second auto-capture loop with on-screen countdown for continuous crop / pest / disease analysis.
- **Live data only** — every weather call uses `cache: "no-store"` plus a cache-buster, and the service worker (`agroreef-v10`) bypasses cache for Open-Meteo, Nominatim and IP-geo hosts.
- **Autonomous email alerts** via Gmail SMTP (no domain required) with farmer-permission workflow.
- **Bilingual** with full Urdu RTL support; Inter for English, Noto Nastaliq Urdu for Urdu.
- **Installable PWA** on Android, iOS and Desktop (registers only on real deployments, never inside the editor preview).
- **Light theme by default** for outdoor readability; theme persisted per user in the database.
- **Secure by default** — RLS on every user table; roles never stored on `profiles`.

---

## Features

### Dashboard (Home)
Zero-scroll professional hub with time-aware bilingual greetings, real-time GPS weather, reverse-geocoded location name, live field & notebook counts, agent activity indicator, unread alert badge, and a compact navigation grid.

### AI Chat Advisor
Multi-agent conversation engine powered by **Gemini 2.5 Flash** through the AI Gateway.
- Live Camera Mode (25-second auto-capture loop)
- Image upload for crop disease / pest diagnosis
- Conversation history with AI-generated chat titles
- Agent network visualization + thinking panel
- Autonomous email alert dialog when risk score ≥ 6/10
- Progressive text reveal for live typing feel
- Voice input/output, language-gated to prevent dual-voice playback (AbortController)

### Field Management
- CRUD fields with crop type, area, soil type and GPS location
- Interactive **Leaflet** map (`react-leaflet` 4.2.1) with Satellite (Esri), Street (OSM) and Terrain (OpenTopoMap) layers
- Boundary points stored as JSONB; auto-detect field location from GPS

### Crop Notebook ("My Notes")
- Rich-text editor (`contentEditable` + `document.execCommand`) with bold, italic, lists, headings
- Categories: general, pest, weather, harvest, expenses
- JSON-serialized entries, full CRUD, realtime sync, search & filter

### Weather Dashboard
- Live data from **Open-Meteo** (free, no API key)
- Current conditions + **7-day forecast** (temp, humidity, wind, weather code, precipitation)
- GPS-only coordinates (no IP fallback) with 10-minute auto-refresh and manual refresh
- Background **video** mapped to the live WMO weather code
- Realtime Postgres subscription on `weather_logs` so autonomous monitor pushes update the UI

### Market Intelligence
- Live crop prices from major Pakistani mandis with % change & trend
- Interactive market map and ticker-style price scroll
- Bilingual crop & mandi names, summary cards, search/filter

### Alert History
- AI-generated alerts with urgency (Low/Medium/High/Critical) and type (Weather/Market/Pest/Action)
- Email delivery status indicator + mini email preview
- Mark as read / mark all read, realtime updates

### Farmer Calculator
Localized agricultural calculations: seed rate, fertilizer, water, yield estimation in acres / maunds / kg.

### Expense Tracker
Track farming income & expenses by category with date filtering.

### Crop Calendar
Seasonal planting guide with Rabi/Kharif awareness and crop lifecycle tracking.

### Soil Logger
Record pH, N, P, K levels linked to specific fields.

### Profile & Settings
Name, email, avatar (cloud upload), language preference, theme (Light/Dark/System with persistence), location & camera permissions, email alert toggle.

### Help & Guide
Bilingual in-app usage guide and feature documentation.

### Auth (Sign In / Sign Up)
Standard email/password (no anonymous signup). Glassmorphism card over a cinematic farm background with animated emerald/teal glow orbs and a compact balanced form.

---

## App Flow

```
Landing → Auth (Sign In / Sign Up) → Onboarding (Permissions & Language)
    ↓
Dashboard (Home)
    ├── AI Chat Advisor (+ Live Camera + Voice)
    ├── Field Management (+ Leaflet Map)
    ├── Crop Notebook (Rich Text)
    ├── Weather Dashboard (live video background)
    ├── Market Intelligence (+ Map + Ticker)
    ├── Alert History (Realtime)
    ├── Farmer Calculator
    ├── Expense Tracker
    ├── Crop Calendar
    ├── Soil Logger
    ├── Profile & Settings
    └── Help & Guide
```

---

## Architecture

```
┌───────────────────────────────────────────────────┐
│           React 18 + Vite 5 + TypeScript          │
│        Tailwind v3 + shadcn/ui + HSL tokens       │
│       PWA (manifest + sw.js v10, deploy-only)     │
├───────────────────────────────────────────────────┤
│                  Cloud Backend                     │
│  ┌──────────────┐   ┌──────────────────────────┐  │
│  │ PostgreSQL   │   │      Edge Functions       │  │
│  │ + RLS + RT   │   │  • agro-agents            │  │
│  │              │   │  • send-farmer-alert      │  │
│  └──────────────┘   │  • autonomous-monitor     │  │
│  ┌──────────────┐   │  • elevenlabs-tts         │  │
│  │   Storage    │   │  • generate-chat-title    │  │
│  │  (avatars)   │   │  • chat-insights          │  │
│  └──────────────┘   └──────────────────────────┘  │
├───────────────────────────────────────────────────┤
│                External Services                   │
│  • AI Gateway (Gemini 2.5 Flash)                   │
│  • Gmail SMTP (email alerts, 500/day free)         │
│  • ElevenLabs (English TTS)                        │
│  • Open-Meteo (weather, free, no key)              │
│  • OpenStreetMap Nominatim (reverse geocoding)     │
└───────────────────────────────────────────────────┘
```

The system runs an **OPARD loop** (Observe → Plan → Act → Reflect → Deliver) for semi-autonomous behavior driven by `pg_cron` + the `autonomous-monitor` edge function.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React 18, TypeScript 5, Vite 5 |
| Styling | Tailwind CSS v3, shadcn/ui, HSL semantic design tokens |
| State | TanStack Query, React Context, custom hooks |
| Routing | React Router v6 |
| Maps | Leaflet + React-Leaflet 4.2.1 |
| Charts | Recharts |
| PWA | Web App Manifest + custom service worker (`sw.js`, cache `agroreef-v10`) |
| Backend | PostgreSQL + Edge Functions (Supabase-compatible) |
| Database | PostgreSQL with Row Level Security + Realtime |
| Auth | Email/password + JWT (no anonymous sign-up; no role storage on profiles) |
| AI | Gemini 2.5 Flash via AI Gateway |
| Email | Gmail SMTP |
| Voice | ElevenLabs TTS (English) + Web Speech API (Urdu `ur-PK`) |
| Weather | Open-Meteo (forecast + WMO weather codes) |
| Geocoding | OpenStreetMap Nominatim |

---

## Agentic AI Pipeline

| Agent | Role | Specialty |
|-------|------|-----------|
| Supervisor | Orchestrator | Routes queries, coordinates the DAG |
| Vision | Image Analysis | Crop disease / pest ID from photos & live frames |
| Weather | Meteorology | Forecasts, rain & frost alerts |
| Market | Economics | Prices, selling timing, trends |
| Critic | QA | Validates advice, catches hallucinations |
| Reasoning | Deep Analysis | Multi-factor agronomic reasoning |
| Executive | Decision Maker | Final recommendation & action plan |

### Pipeline Flow
1. Farmer sends a query (text / voice / image / live camera frame)
2. Client (`useAgroAgents`) attaches **live device GPS** (`getLiveCoords`) and field context, then invokes `agro-agents`
3. **Supervisor** activates the relevant specialist subset
4. Specialists run in parallel; the Weather agent fetches live Open-Meteo data for the user's coordinates, the Market agent reads the latest `crop_prices`, the Vision agent processes base64 images
5. **Critic** validates the combined response with weighted confidence aggregation
6. **Executive** formulates the final recommendation
7. Every significant decision is logged to `agent_memory` (confidence, latency, agents used)
8. If risk ≥ 6/10 → triggers the **email alert dialog** for farmer permission

A `cleanAgentMetadata` regex helper strips internal agent metadata before display. AI Gateway 402/429 errors are handled with exponential backoff.

---

## PWA & Installability

- **`public/manifest.json`** — name, short_name, theme color `#16a34a`, AgroReef icons (192 / 512 / maskable), `display: standalone`
- **`public/sw.js`** — minimal service worker (cache `agroreef-v10`) that is **network-only** for Open-Meteo, Nominatim and IP-geo hosts, and network-first for static assets
- **Registration logic** (`src/main.tsx`) — registered only on real deployments; inside iframes, `localhost`, `127.0.0.1`, `id-preview--*` and `lovableproject.com` hosts the SW is **unregistered** to keep previews fresh
- **`InstallPWAButton`** — listens to `beforeinstallprompt`, shows an install CTA, and hides itself once installed (`appinstalled` + `display-mode: standalone` check)
- Architected for AAB generation via PWABuilder; the `capture` attribute is intentionally avoided on file inputs for APK compatibility

> Install prompts only fire on the deployed/published HTTPS build — not inside the editor preview.

---

## Email Alert System

### Gmail SMTP (no domain required)
- Sends to any farmer email — no DNS / domain setup
- 500 emails/day free quota
- Authenticated via `GMAIL_USER` + `GMAIL_APP_PASSWORD` (16-character App Password)

### Trigger Rules
| Condition | Threshold | Alert Type |
|-----------|-----------|------------|
| Pest detected in image | Risk ≥ 6/10 | Pest Warning |
| Severe weather incoming | Risk ≥ 8/10 | Weather Alert |
| Critical crop issue | Risk ≥ 8/10 | Action Required |
| Market price spike | Significant change | Market Update |

### Email Features
- Branded HTML template (AgroReef colors + logo)
- Bilingual (Urdu/English) with RTL support
- Color-coded by alert type, urgency badges, action checklist
- Timestamps in **Pakistan Standard Time (Asia/Karachi)**
- 3-retry mechanism with exponential backoff

---

## Database Schema

| Table | Purpose | RLS |
|-------|---------|-----|
| `profiles` | User profile, language, avatar, theme, onboarding status | User owns own row |
| `fields` | Farm fields with boundary_points (JSONB) and crop data | User CRUD own |
| `notebooks` | Farming notes (JSON-serialized entries) | User CRUD own |
| `alerts` | AI-generated alerts with email delivery status | User CRUD own |
| `weather_logs` | Cached weather snapshots (raw_data JSONB) | User insert/view own |
| `agent_memory` | Logged AI decisions (confidence, latency, agents) | User insert/view own |
| `crop_prices` | Market prices | Public read |

All user tables enforce Row Level Security. Realtime is enabled on `alerts`, `notebooks` and `weather_logs`. Roles are never stored on `profiles` — a separate `user_roles` table + `has_role()` security-definer function pattern is used to prevent privilege escalation. Time-based validations use **triggers** rather than CHECK constraints.

---

## Edge Functions

| Function | Purpose | `verify_jwt` |
|----------|---------|--------------|
| `agro-agents` | Multi-agent AI chat engine (7 agents, Gemini 2.5 Flash) | false |
| `send-farmer-alert` | Gmail SMTP email delivery with retry | false |
| `autonomous-monitor` | `pg_cron`-driven background monitoring + alert generation | false |
| `elevenlabs-tts` | English text-to-speech | false |
| `generate-chat-title` | AI-generated short titles for chat history | false |
| `chat-insights` | Conversation analysis & insights | false |

JWT verification is performed inside each function via `supabaseClient.auth.getUser()` where authenticated context is required.

---

## Design System

- **Aesthetic** — professional, high-density, mobile-first, zero-scroll dashboard hub
- **Theme** — Light Mode default for outdoor readability; persisted per user via `useThemePersistence`
- **Typography** — `Inter` for English UI, `Noto Nastaliq Urdu` for Urdu script
- **Colors** — defined as **HSL semantic tokens** in `src/index.css` and `tailwind.config.ts`. Components never hardcode colors; they use tokens like `bg-primary`, `text-foreground`, `bg-card`
- **Branding** — transparent multi-leaf green AgroReef icon with golden accents, used for app icon, splash and auth header
- **Auth Page** — glassmorphism card (`backdrop-blur-2xl`, `bg-white/15`, `border-white/30`) over a cinematic farm background with animated emerald/teal glow orbs

---

## Getting Started

### Prerequisites
- Node.js 18+ and npm (or bun)
- A Gmail account with 2-Step Verification (only if you want email alerts)

### Installation

```bash
# Clone the repository
git clone <YOUR_GIT_URL>
cd agroreef

# Install dependencies
npm install

# Start development server
npm run dev
```

### Available Scripts

| Script | Purpose |
|--------|---------|
| `npm run dev` | Start Vite dev server |
| `npm run build` | Production build |
| `npm run build:dev` | Development-mode build |
| `npm run preview` | Preview the production build |
| `npm run lint` | Run ESLint |
| `npm run test` | Run Vitest once |
| `npm run test:watch` | Run Vitest in watch mode |

### Gmail SMTP Setup (for Email Alerts)

1. Enable 2-Step Verification at <https://myaccount.google.com/security>
2. Generate an App Password at <https://myaccount.google.com/apppasswords> (choose "Mail" / custom name "AgroReef") and copy the 16-character password
3. Add the secrets `GMAIL_USER` and `GMAIL_APP_PASSWORD` to the cloud backend

---

## Environment Variables

### Frontend (auto-configured — do not edit `.env` manually)
| Variable | Description |
|----------|-------------|
| `VITE_SUPABASE_URL` | Backend API URL |
| `VITE_SUPABASE_PUBLISHABLE_KEY` | Public (anon) API key |
| `VITE_SUPABASE_PROJECT_ID` | Project identifier |

### Backend Secrets
| Secret | Description |
|--------|-------------|
| `LOVABLE_API_KEY` | AI Gateway access key (auto-provided) |
| `GMAIL_USER` | Gmail address used to send alerts |
| `GMAIL_APP_PASSWORD` | Gmail App Password (16 characters) |
| `ELEVENLABS_API_KEY` | ElevenLabs text-to-speech API key |
| `SUPABASE_SERVICE_ROLE_KEY` | Backend admin access (auto-provided) |

---

## Deployment

The deployed/published HTTPS build is where the **PWA install prompt**, the service worker (`agroreef-v10`) and the email pipeline behave fully. For a custom domain, point an A record to `185.158.133.1` and configure the domain in the project settings.

---

## License

Private project - All rights reserved.

---

## Author

**Ashub Shafqat** - BS Software Engineering (2022–2026), University of Agriculture, Faisalabad
**Supervisor:** Dr. Nayyar Iqbal

<p align="center">
  <img src="agroreef-icon-192.png" alt="AgroReef" width="56" height="56" /><br>
  <strong>AgroReef - Empowering Pakistani Farmers with Agentic AI</strong>
</p>
