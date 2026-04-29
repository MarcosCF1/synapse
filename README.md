<div align="center">

<svg width="900" height="160" viewBox="0 0 900 160" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <linearGradient id="bg" x1="0%" y1="0%" x2="100%" y2="100%">
      <stop offset="0%" style="stop-color:#0d1117"/>
      <stop offset="100%" style="stop-color:#161b22"/>
    </linearGradient>
    <linearGradient id="accent" x1="0%" y1="0%" x2="100%" y2="0%">
      <stop offset="0%" style="stop-color:#01696f"/>
      <stop offset="100%" style="stop-color:#4fa4b0"/>
    </linearGradient>
  </defs>
  <rect width="900" height="160" rx="14" fill="url(#bg)"/>
  <rect x="0" y="140" width="900" height="3" rx="2" fill="url(#accent)"/>
  <rect x="44" y="40" width="3" height="85" rx="2" fill="url(#accent)"/>
  <circle cx="780" cy="50" r="4" fill="#4fa4b0" opacity="0.8"/>
  <circle cx="800" cy="75" r="6" fill="#01696f" opacity="0.9"/>
  <circle cx="820" cy="45" r="3" fill="#4fa4b0" opacity="0.6"/>
  <circle cx="840" cy="80" r="5" fill="#4fa4b0" opacity="0.7"/>
  <circle cx="858" cy="58" r="4" fill="#01696f" opacity="0.8"/>
  <line x1="780" y1="50" x2="800" y2="75" stroke="#01696f" stroke-width="1" opacity="0.5"/>
  <line x1="800" y1="75" x2="820" y2="45" stroke="#4fa4b0" stroke-width="1" opacity="0.4"/>
  <line x1="820" y1="45" x2="840" y2="80" stroke="#01696f" stroke-width="1" opacity="0.5"/>
  <line x1="840" y1="80" x2="858" y2="58" stroke="#4fa4b0" stroke-width="1" opacity="0.4"/>
  <line x1="780" y1="50" x2="858" y2="58" stroke="#01696f" stroke-width="0.5" opacity="0.2"/>
  <text x="68" y="95" font-family="Georgia,serif" font-style="italic" font-size="56" fill="#e6edf3" letter-spacing="-1">SYNAPSE</text>
  <text x="70" y="124" font-family="Helvetica Neue,Arial,sans-serif" font-size="12" fill="#4fa4b0" letter-spacing="4">ANDROID  ·  REAL-TIME AI AUDIO  ·  WEARABLE INTEGRATION  ·  CLOUD SYNC</text>
</svg>

[![Kotlin](https://img.shields.io/badge/Kotlin-Android-7F52FF?style=flat-square&logo=kotlin&logoColor=white)](https://kotlinlang.org)
[![Gemini](https://img.shields.io/badge/Google%20Gemini-AI%20Transcription-4285F4?style=flat-square&logo=google&logoColor=white)](https://ai.google.dev)
[![Supabase](https://img.shields.io/badge/Supabase-Backend-3ECF8E?style=flat-square&logo=supabase&logoColor=white)](https://supabase.com)
[![n8n](https://img.shields.io/badge/n8n-Automation-EA4B71?style=flat-square&logo=n8n&logoColor=white)](https://n8n.io)
[![BLE](https://img.shields.io/badge/Mi%20Band%205-BLE%20Integration-30363d?style=flat-square)](https://github.com/MarcosCF1/synapse)
[![Status](https://img.shields.io/badge/%E2%97%8F%20Active%20Development-v0.9-01696f?style=flat-square)](https://github.com/MarcosCF1/synapse)

*What if your phone could listen to your life, understand it through AI, and correlate what you said with how your body was feeling — in real time?*

</div>

---

## The Vision

Most personal knowledge tools (notes apps, journals, voice memos) require you to manually record, organize, and retrieve information. SYNAPSE removes that friction entirely.

You wear a Mi Band 5. Your phone listens. SYNAPSE captures ambient audio, transcribes it in real time via **Google Gemini**, tags it with biometric context (heart rate, activity state from the wearable), stores everything in a structured Supabase backend, and triggers automation workflows via n8n — all without you touching your phone.

The result: a **searchable, contextualized personal knowledge base** built passively from your own life.

---

## System Architecture

```
┌────────────────────────────────────────────────────────────────┐
│                        ANDROID DEVICE                              │
│                                                                    │
│  Microphone ─► AudioCapture Service ─► Gemini API ─► Transcript      │
│                                                        +            │
│  Mi Band 5  ─► BLE GATT Client  ─────────► Biometric Snapshot  │
│              (HR + activity)                                        │
└────────────────────────────────────┬─────────────────────────┘
                                     │
              ┌────────────────────┴─────────────────┐
              │                   CLOUD                   │
              │                                           │
    ┌────────┴───────┐         ┌───────────────────┘
    │  Supabase (PostgreSQL) │         │  n8n Workflows         │
    │  sessions              │         │  ► Daily AI summary    │
    │  biometric_snapshots   │         │  ► WhatsApp delivery   │
    │  RLS auth policies     │         │  ► Archive raw audio   │
    └───────────────────────         └───────────────────┘
```

---

## Core Features

### 🎤 Real-Time Audio Transcription
Background service captures audio in configurable chunks (2s–10s windows). Each chunk is sent to the **Gemini API** for transcription and results are streamed back and appended to the current session record, giving users a live transcript.

**Why Gemini over Whisper or on-device STT?** Gemini handles code, technical terms, and mixed-language input (PT/EN) without specialized fine-tuning. API latency is acceptable for non-conversational capture (≤1.5s per chunk at LTE). On-device STT accuracy drops significantly for technical vocabulary.

### ⌚ Wearable Biometric Sync (Mi Band 5 via BLE)
A GATT client connects to the Mi Band 5 over Bluetooth Low Energy and polls:
- Heart rate (characteristic UUID: `0x2A37`)
- Activity state (resting / active / workout)

Biometric snapshots are timestamped and correlated with transcript records — enabling queries like *"show me everything I said when my heart rate was above 90"*.

### ☁️ Structured Cloud Storage (Supabase)

```sql
CREATE TABLE sessions (
    id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id      UUID REFERENCES auth.users NOT NULL,
    started_at   TIMESTAMPTZ DEFAULT NOW(),
    ended_at     TIMESTAMPTZ,
    transcript   TEXT,
    duration_sec INTEGER
);

CREATE TABLE biometric_snapshots (
    id           SERIAL PRIMARY KEY,
    session_id   UUID REFERENCES sessions(id),
    recorded_at  TIMESTAMPTZ DEFAULT NOW(),
    heart_rate   SMALLINT,
    activity     VARCHAR(20)
);
```

Row Level Security policies ensure only the authenticated user can access their own records.

### ⚡ Automation Layer (n8n)
Webhook triggers fire when a session ends. n8n workflows generate a daily AI summary (Gemini prompt across all sessions from the last 24h) and deliver it via WhatsApp through Evolution API.

---

## Tech Stack

| Layer | Technology | Rationale |
|---|---|---|
| Mobile | Kotlin, Android (min API 26) | Coroutines + Flow for async audio pipeline |
| AI | Google Gemini Flash 1.5 | Best multilingual STT quality without fine-tuning |
| BLE | Android Bluetooth LE (GATT) | Consumer wearable integration without official SDK |
| Backend | Supabase (PostgreSQL + Auth) | Zero-server backend; RLS for data isolation |
| Automation | n8n (self-hosted) | Webhook-driven pipelines; no vendor lock-in |
| Messaging | Evolution API | WhatsApp delivery of daily AI summaries |

---

## Engineering Challenges

**Battery & Background Execution** — Android kills background processes. SYNAPSE uses a Foreground Service with a persistent notification to keep audio capture alive. BLE polling uses `WorkManager` with battery-aware constraints.

**BLE Reverse Engineering** — The Mi Band 5 has no official SDK. HR characteristic UUIDs were mapped by inspecting GATT service tables manually. The connection retry logic handles the band's aggressive power-saving disconnects with exponential backoff.

**Chunk Latency Budget** — Each audio chunk must complete the full pipeline (capture → Gemini API → UI) within the next chunk's recording window to avoid transcript gaps. Current 3s windows give ~1.5s budget for API round-trip. Overflow chunks are queued and displayed in order on flush.

---

## Roadmap

```
[v0.9  — current]  Core audio capture → Gemini transcription → Supabase storage
[v0.9.5]            Mi Band 5 BLE HR overlay on transcript timeline
[v1.0]              n8n daily summary → WhatsApp delivery (full pipeline live)
[v1.1]              Semantic search across transcripts (pgvector embeddings)
[v1.2]              Offline-first mode with cloud sync on reconnect
[v2.0]              Multi-device support + shared session spaces
```

---

## Getting Started

> Requires a Google Gemini API key and a Supabase project (both have free tiers).

```bash
git clone https://github.com/MarcosCF1/synapse.git
# Open in Android Studio (Hedgehog / Iguana+)
cp local.properties.example local.properties
# Set: GEMINI_API_KEY, SUPABASE_URL, SUPABASE_ANON_KEY
# Connect Android device (API 26+) → Run 'app'
```

> BLE features require a physical device. Pair Mi Band 5 via system Bluetooth before launching the app.

---

## Author

**Marcos Pires** — Full Stack Developer & AI Integration Engineer

Building at the intersection of AI, mobile, and automation. Open to Data Engineering and Backend roles.

[![LinkedIn](https://img.shields.io/badge/LinkedIn-marcos--pires--dev-0A66C2?style=flat-square&logo=linkedin)](https://linkedin.com/in/marcos-pires-dev)
[![GitHub](https://img.shields.io/badge/GitHub-MarcosCF1-181717?style=flat-square&logo=github)](https://github.com/MarcosCF1)

---

<sub>Active development — star the repo to follow progress.</sub>
