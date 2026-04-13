# 🌾 AgroReef — Autonomous Farming Assistant (Agentic AI)

> **Pakistan's first multi-agent AI farming OS** — empowering farmers with real-time intelligence, voice-first interaction, and autonomous decision-making in Urdu & English.
>All Source Code and sensitive info is Private.
---

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [App Flow](#app-flow)
- [AI Agent System](#ai-agent-system)
- [Email Alert System](#email-alert-system)
- [Database Schema](#database-schema)
- [Edge Functions](#edge-functions)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)

---

## Overview

AgroReef is a comprehensive agricultural mobile-first web application designed specifically for Pakistani farmers. It operates as a multi-agent autonomous AI system that monitors weather, market prices, pest risks, and provides personalized farming advice — all through natural conversation in Urdu or English.

### Key Highlights
- 🤖 **7 AI Agents** working collaboratively for intelligent farming decisions
- 🗣️ **Voice-first** — farmers can speak in Urdu; AI responds with text-to-speech
- 📧 **Gmail SMTP email alerts** — sends to ANY farmer's email 
- 🌍 **Bilingual** — full Urdu/English support with RTL layout
- 📱 **Mobile-first PWA** — works offline-ready with responsive design
- 🌙 **Dark/Light themes** — with system preference detection

---

## Features

### 🏠 Dashboard
- Personalized greeting with time-of-day awareness
- Real-time weather from GPS with auto-refresh every 5 minutes
- Location name via reverse geocoding (OpenStreetMap Nominatim)
- Quick access grid to all 9 modules
- Live field & notebook counts
- Agent status indicator with last activity timestamp
- Unread alert badge counter

### 🤖 AI Chat Advisor
- Multi-agent conversation engine powered by Gemini 2.5 Flash
- **Live camera analysis** — point camera at crops for instant disease/pest identification
- Image upload support for crop diagnosis
- Conversation history persistence per session
- Agent network visualization showing active agents
- Agent thinking panel with real-time reasoning logs
- **Email alert system** — agent autonomously detects critical issues and asks permission to send email alerts
- Voice input/output with ElevenLabs TTS
- Recent chat history viewer

### 🌾 Field Management
- Add/edit/delete farm fields with details (crop type, area, soil type, location)
- Leaflet-based interactive map with field boundaries
- GPS-based field location auto-detection
- Per-field notes and observations

### 📓 Crop Notebook
- Digital notebook for farming observations
- Categories: general, pest, weather, harvest, expenses
- Full CRUD with real-time sync
- Search and filter functionality

### 🌤️ Weather Dashboard
- Real-time weather from Open-Meteo API
- 3-day forecast with temperature, humidity, wind
- Weather code interpretation (Clear, Cloudy, Rain, Storm)
- GPS-based location with auto-refresh
- Ambient weather background video

### 📊 Market Intelligence
- Live crop prices from major Pakistani mandis
- Price change percentages and trends
- Interactive market map
- Bilingual crop & mandi names (Urdu/English)
- Market summary cards with key metrics
- Search and filter by crop or mandi
- Ticker-style price scroll

### 🔔 Alert History
- All AI-generated alerts with urgency levels (Low, Medium, High, Critical)
- Alert types: Weather, Market, Pest, Action
- Email delivery status indicator
- Mark as read / Mark all as read
- Real-time updates via database subscriptions
- Mini email preview for sent alerts

### 🧮 Farmer Calculator
- Agricultural calculations for Pakistani farming
- Seed rate, fertilizer, water, yield estimations
- Localized units (acres, maunds, kg)

### 💰 Expense Tracker
- Track farming expenses by category
- Income vs expense overview
- Date-based filtering

### 📅 Crop Calendar
- Seasonal planting guide for Pakistani crops
- Rabi/Kharif season awareness
- Crop lifecycle tracking

### 🧪 Soil Logger
- Record soil test results
- pH, nitrogen, phosphorus, potassium levels
- Field-linked soil data

### 👤 Profile & Settings
- Name, email, avatar management
- Avatar upload to cloud storage
- Language preference (Urdu/English)
- Theme preference (Light/Dark/System)
- Location & camera permissions
- Email alert toggle with visual status

### ❓ Help & Guide
- In-app usage guide
- Feature explanations
- Bilingual support documentation

---

## Architecture

```
┌─────────────────────────────────────────────────┐
│                  React Frontend                  │
│  (Vite + TypeScript + Tailwind + shadcn/ui)     │
├─────────────────────────────────────────────────┤
│              Lovable Cloud Backend               │
│  ┌──────────────┐ ┌──────────────────────────┐  │
│  │   Database    │ │    Edge Functions         │  │
│  │  (PostgreSQL) │ │  • agro-agents           │  │
│  │  + RLS + RT   │ │  • send-farmer-alert     │  │
│  └──────────────┘ │  • autonomous-monitor     │  │
│                    │  • elevenlabs-tts         │  │
│  ┌──────────────┐ │  • generate-chat-title    │  │
│  │    Storage    │ │  • chat-insights          │  │
│  │   (Avatars)   │ └──────────────────────────┘  │
│  └──────────────┘                                │
├─────────────────────────────────────────────────┤
│              External Services                   │
│  • AI Gateway (Gemini 2.5 Flash)                │
│  • Gmail SMTP (Email Alerts)                     │
│  • ElevenLabs (Text-to-Speech)                  │
│  • Open-Meteo (Weather API)                     │
│  • OpenStreetMap (Geocoding)                    │
└─────────────────────────────────────────────────┘
```

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| **Frontend** | React 18, TypeScript, Vite |
| **Styling** | Tailwind CSS, shadcn/ui, CSS design tokens |
| **State** | React Query, React Context, Custom hooks |
| **Routing** | React Router v6 |
| **Maps** | Leaflet + React-Leaflet |
| **Charts** | Recharts |
| **Backend** | Supabase |
| **Database** | PostgreSQL with RLS + Realtime |
| **Auth** | Email/password with JWT |
| **AI** | Gemini 2.5 Flash via AI Gateway |
| **Email** | Gmail SMTP (500 emails/day free) |
| **Voice** | ElevenLabs TTS + Web Speech API |
| **Weather** | Open-Meteo API (free, no key) |
| **Geocoding** | OpenStreetMap Nominatim (free) |

---

## App Flow

```
Landing Page → Auth (Login/Register) → Onboarding (Permissions)
    ↓
Dashboard (Home)
    ├── AI Chat Advisor (+ Camera + Voice)
    ├── Field Management (+ Map)
    ├── Crop Notebook
    ├── Weather Dashboard
    ├── Market Intelligence (+ Map)
    ├── Alert History
    ├── Farmer Calculator
    ├── Expense Tracker
    ├── Crop Calendar
    ├── Soil Logger
    ├── Profile & Settings
    └── Help & Guide
```

---

## AI Agent System

AgroReef uses a **7-agent collaborative architecture**:

| Agent | Role | Specialty |
|-------|------|-----------|
| 🧠 **Supervisor** | Orchestrator | Routes queries, coordinates agents |
| 👁️ **Vision** | Image Analysis | Crop disease & pest identification from photos |
| 🌤️ **Weather** | Meteorology | Weather forecasts, rain predictions, frost alerts |
| 📊 **Market** | Economics | Crop prices, selling timing, market trends |
| 🛡️ **Critic** | Quality Control | Validates advice, checks for errors |
| 🔬 **Reasoning** | Deep Analysis | Complex multi-factor agricultural reasoning |
| ⚡ **Executive** | Decision Maker | Final recommendations, action plans |

### How It Works
1. Farmer asks a question (text, voice, or image)
2. **Supervisor** analyzes the query and activates relevant agents
3. Specialist agents process in their domains
4. **Critic** validates the combined response
5. **Executive** formulates the final recommendation
6. If a critical issue is detected (risk score ≥ 6/10), the system triggers an email alert with farmer's permission

---

## Email Alert System

### Gmail SMTP Approach (Current)

### Alert Trigger Rules
| Condition | Threshold | Alert Type |
|-----------|-----------|------------|
| Pest detected in image | Risk ≥ 6/10 | 🐛 Pest Warning |
| Severe weather incoming | Risk ≥ 8/10 | 🌤️ Weather Alert |
| Critical crop issue | Risk ≥ 8/10 | 🎯 Action Required |
| Market price spike | Significant change | 📊 Market Update |

### Email Features
- Beautiful HTML templates with AgroReef branding
- Bilingual (Urdu/English) with RTL support
- Color-coded by alert type (red=pest, blue=weather, etc.)
- Urgency badges (Low, Medium, High, Critical)
- Action items checklist
- Pakistan Standard Time (Asia/Karachi) timestamps
- 3-retry mechanism with exponential backoff

---

## Database Schema

| Table | Purpose | RLS |
|-------|---------|-----|
| `profiles` | User profiles, settings, preferences | User owns own row |
| `fields` | Farm fields with boundaries & crop data | User CRUD own fields |
| `notebooks` | Farming observation notes | User CRUD own notebooks |
| `alerts` | AI-generated alerts with email status | User CRUD own alerts |
| `weather_logs` | Weather data snapshots | User insert/view own |
| `agent_memory` | AI agent decision history | User insert/view own |
| `crop_prices` | Market prices (public read) | Anyone can view |

All tables have **Row Level Security (RLS)** enforced. Realtime subscriptions enabled for live updates.

---

## Edge Functions

| Function | Purpose | Auth |
|----------|---------|------|
| `agro-agents` | Multi-agent AI chat engine | JWT |
| `send-farmer-alert` | Gmail SMTP email delivery | JWT / Service Key |
| `autonomous-monitor` | Background monitoring & alerts | Service Role Key |
| `elevenlabs-tts` | Text-to-speech conversion | Open |
| `generate-chat-title` | AI-generated chat titles | Open |
| `chat-insights` | Chat analysis & insights | Open |

---

## Getting Started

### Prerequisites
- Node.js 18+ & npm
- A Gmail account with 2FA enabled

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

### Gmail SMTP Setup (for Email Alerts)

## Environment Variables

### Frontend (auto-configured)

### Backend Secrets

---


## License

Private project — All rights reserved.

---

<p align="center">
  <strong>🌾 AgroReef — Empowering Pakistani Farmers with Agentic AI</strong><br>
  <em>Built with <3 by Ashub Shafqat </em>
</p>
