# WarmUpView.ai — Architecture Documentation

> Complete technical reference for all system components and data flows.

---

## Table of Contents

1. [System Overview](#1-system-overview)
2. [Backend Modules](#2-backend-modules)
3. [Frontend Modules](#3-frontend-modules)
4. [AI Services Routing](#4-ai-services-routing)
5. [Analysis Pipeline](#5-analysis-pipeline)
6. [Database Schema (ERD)](#6-database-schema-erd)
7. [Scoring Formulas](#7-scoring-formulas)
8. [WebSocket Protocol](#8-websocket-protocol)
9. [Audio Mixing Flow](#9-audio-mixing-flow)
10. [Interview Stage Progression](#10-interview-stage-progression)

---

## 1. System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                         User Browser                           │
│  ┌──────────┐  ┌──────────────┐  ┌──────────┐  ┌──────────┐  │
│  │ Warm-up  │  │ Real Intrvw  │  │ CV Analy │  │ History  │  │
│  └────┬─────┘  └──────┬───────┘  └────┬─────┘  └────┬─────┘  │
└───────┼───────────────┼───────────────┼──────────────┼─────────┘
        │   WebSocket / REST API (HTTP)  │              │
        ▼                               ▼              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    FastAPI Backend (port 8000)                  │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │  /warmup    │  │/real-interview│  │     /cv-analysis     │  │
│  │  WebSocket  │  │  WebSocket   │  │       REST API       │  │
│  └──────┬──────┘  └──────┬───────┘  └──────────┬───────────┘  │
│         │                │                      │              │
│  ┌──────▼────────────────▼──────────────────────▼───────────┐  │
│  │              Core Services & Shared Layer                │  │
│  │  LLM Service │ Question Engine │ Auth │ Session Manager  │  │
│  │  Response Validator │ LLM Health │ Transcript Buffer     │  │
│  └──────────────────────────────────────────────────────────┘  │
│                         │                                      │
│  ┌──────────────────────▼───────────────────────────────────┐  │
│  │         Analysis Worker (background subprocess)          │  │
│  │  Emotion (Keras) │ Gaze (MediaPipe) │ Pose (MediaPipe)  │  │
│  │  Timeline Builder │ Snapshot Extractor │ Transcript Eval │  │
│  └──────────────────────┬───────────────────────────────────┘  │
└─────────────────────────┼───────────────────────────────────────┘
                          ▼
             ┌─────────────────────┐
             │      SQLite DB      │
             │   + File Storage    │
             └─────────────────────┘

External APIs:
  Groq (LLM) │ Google Gemini │ Deepgram (STT) │ ElevenLabs (TTS) │ Edge-TTS │ Simli (Avatar)
```

---

## 2. Backend Modules

### `backend/app/main.py`
FastAPI entry point — configures CORS + SessionMiddleware, mounts all sub-routers (warmup, real_interview, auth, admin), creates DB tables on startup, serves frontend as static files.

### `backend/app/models.py`
SQLAlchemy ORM definitions for all **7 database tables**: `users`, `interview_sessions`, `session_messages`, `cv_profiles`, `job_requirements`, `real_interview_sessions`, `analysis_results`.

### `backend/app/config.py`
Centralizes all environment variables: API keys (Groq, Gemini, Deepgram, ElevenLabs, Simli), Google OAuth Client ID, database URL, TTS voice config, auth secret.

### `backend/app/database.py`
Async SQLAlchemy engine + session factory + startup/shutdown lifecycle.

### `backend/app/services/`
| File | Purpose |
|---|---|
| `llm_service.py` | Groq LLM client (`GroqLLM` class) — multi-key rotation, health management, response validation, conversation history, anti-duplicate/anti-repetition guards |
| `stt_service.py` | Deepgram STT integration for real-time streaming transcription |
| `tts_service.py` | Edge-TTS synthesis (Microsoft Neural Voices) — used for warm-up module |
| `evaluation_service.py` | Session evaluation logic (Groq-based scoring) |
| `session_service.py` | Session CRUD operations |

### `backend/app/warmup/`
| File | Purpose |
|---|---|
| `routes.py` | REST endpoints for warm-up sessions CRUD + WebSocket upgrade |
| `websocket.py` | Real-time interview logic: Deepgram STT buffer, Groq LLM routing, Edge-TTS, stage management, silence detection |

### `backend/app/real_interview/`
| File | Purpose |
|---|---|
| `routes.py` | CV upload (PyPDF extraction), job requirements, session management, video upload, analysis trigger, report retrieval, Simli session tokens, ElevenLabs/Edge-TTS endpoints |
| `websocket.py` | Live interview WebSocket: question delivery, acknowledgment, session end signal |
| `recorder_manager.py` | Ensures storage directories exist (recordings/, snapshots/, uploads/) |
| `report_generator.py` | Synthesizes analysis data into report format |
| `validators.py` | Input validation for CV, job requirements, and sessions |

### `backend/app/analysis/`
| File | Purpose |
|---|---|
| `analysis_worker.py` | Spawns as subprocess via `Popen`, orchestrates all 5 pipeline steps |
| `emotion_analyzer.py` | Loads Keras CNN model, runs per-frame emotion classification (48×48 grayscale → 7 classes) |
| `gaze_analyzer.py` | MediaPipe FaceMesh — iris deviation → eye contact boolean (threshold: 0.06) |
| `pose_analyzer.py` | MediaPipe Pose — 33 landmarks → posture score + signal detection (hand_on_head, leaning, etc.) |
| `video_processor.py` | OpenCV frame extraction at 1 fps |
| `timeline_builder.py` | Merges per-second data from all 3 analyzers into unified JSON timeline |
| `snapshot_extractor.py` | Detects peak emotional/postural moments and saves key frames as JPEG |
| `transcript_evaluator.py` | Evaluates interview transcript quality using AI |
| `report_generator.py` | Calls Gemini to synthesize raw metrics into qualitative feedback |

### `backend/app/core/`
| File | Purpose |
|---|---|
| `question_engine.py` | Stage-based question pool management (5 stages × 5–10 questions each), random selection, no-repeat logic, persona configs |
| `llm_health_manager.py` | Tracks LLM success/failure rates, manages backoff delays, health gate for degraded mode |
| `response_validator.py` | Validates AI responses (completeness, format, persona compliance), generates repair instructions |
| `transcript_buffer.py` | Buffers incoming transcript fragments, flushes after 3.5s silence |

### `backend/app/auth/`
| File | Purpose |
|---|---|
| `routes.py` | Login, signup, Google OAuth callback, session management, profile endpoints |
| `deps.py` | Auth dependencies — current user extraction from session |

### `backend/app/admin/`
| File | Purpose |
|---|---|
| `routes.py` | Admin endpoints — user listing, admin promotion, user management |

---

## 3. Frontend Modules

```
frontend/
├── index.html                  ← Dashboard / navigation hub
├── admin.html                  ← Admin panel (user management)
├── profile.html                ← User profile & settings
├── auth/
│   ├── auth.html               ← Combined login & signup page
│   ├── auth.js                 ← Auth logic (email + Google OAuth)
│   └── auth.css                ← Auth styles
├── warmup/
│   ├── interview.html          ← Live warm-up interview + avatar
│   ├── interview.js            ← Interview WebSocket logic
│   ├── session.html            ← Session review / transcript viewer
│   ├── session.js              ← Session viewer logic
│   └── avatar.js               ← Animated AI avatar + lip-sync
├── real_interview/
│   ├── real_interview.html     ← Live interview + Monaco code editor
│   ├── real_interview.js       ← Interview WS + recording + Simli
│   ├── report.html             ← Multimodal analytics report
│   ├── report.js               ← Report rendering & charts
│   ├── history.html            ← Session history with filter & status
│   └── history.js              ← Status polling, filter, bulk delete
├── cv_analysis/
│   └── cv_analysis.html        ← CV upload + ATS analysis viewer
├── css/
│   ├── style.css               ← Global design system
│   ├── layout.css              ← Shared layout styles
│   └── home.css                ← Dashboard styles
├── js/
│   ├── layout.js               ← Shared nav, theme toggle, auth guard
│   ├── app.js                  ← Dashboard initialization
│   ├── home.js                 ← Home page logic
│   ├── admin.js                ← Admin panel logic
│   └── auth-guard.js           ← Route protection / auth check
└── images/
```

---

## 4. AI Services Routing

```
Warm-up Module
├── Conversation (all turns)     →  Groq  (llama-3.3-70b-versatile)
├── Evaluation Report            →  Groq  (llama-3.3-70b-versatile)
└── TTS                          →  Edge-TTS (Microsoft Neural Voices)

Real Interview Module
├── Question Generation          →  Groq  (llama-3.3-70b-versatile)
├── Coding Task Generation       →  Groq  (llama-3.3-70b-versatile)
├── Audio Tone Analysis          →  Gemini (gemini-2.5-flash, audio inline)
├── Tone Fallback                →  Librosa (local)
├── TTS (primary)                →  ElevenLabs (streaming PCM/MP3)
├── TTS (fallback)               →  Edge-TTS
└── Visual Avatar                →  Simli (WebRTC)

CV Analysis Module
└── ATS Feedback + Annotations   →  Gemini (gemini-2.5-flash)
```

**Why split?**
- **Groq** is used for high-frequency, latency-sensitive text generation (conversation turns, real-time)
- **Gemini** is used for multimodal tasks (audio understanding, document analysis)
- **ElevenLabs** provides higher-quality voices for real interviews; Edge-TTS is free for warm-up practice

---

## 5. Analysis Pipeline

```
Recorded Video (.webm)
        │
        ▼
┌───────────────────────────────────┐
│ Step 1: Frame Extraction (OpenCV) │
│   video_processor.py              │
│   → 1 frame per second            │
│   → frames[] array in memory      │
└──────────────┬────────────────────┘
               │ frames[]
       ┌───────┼────────┐
       │       │        │ (parallel)
       ▼       ▼        ▼
┌───────────┐ ┌─────────────┐ ┌──────────────┐
│ Emotion   │ │ Gaze        │ │ Pose         │
│ Analyzer  │ │ Analyzer    │ │ Analyzer     │
│ Keras CNN │ │ MediaPipe   │ │ MediaPipe    │
│ 48x48     │ │ FaceMesh    │ │ Pose         │
│ grayscale │ │ Iris devn   │ │ 33 landmarks │
│ 7 classes │ │ < 0.06      │ │ signals      │
└─────┬─────┘ └──────┬──────┘ └──────┬───────┘
      └───────────────┼───────────────┘
                      ▼
          ┌───────────────────────┐
          │ Step 3: Timeline      │
          │ timeline_builder.py   │
          │ Merge per-second JSON │
          └──────────┬────────────┘
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼
┌──────────────────┐ ┌────────────────────┐
│ Step 4: Peak     │ │ Step 5: Score      │
│ Snapshots        │ │ Calculator         │
│ snapshot_         │ │ Weighted formula   │
│ extractor.py     │ │ → overall_score    │
└──────┬───────────┘ └────────┬───────────┘
       └──────────┬────────────┘
                  ▼
       ┌────────────────────────┐
       │  SQLite DB             │
       │  analysis_results      │
       └────────────────────────┘
```

---

## 6. Database Schema (ERD)

```
users
─────
id (PK)
email (unique)
name
password_hash
auth_provider         # "email" | "google"
google_sub
picture_url
is_admin              # 0=user, 1=admin
created_at


interview_sessions          session_messages
─────────────────           ────────────────
id (PK)              1──<  id (PK)
user_id                     session_id (FK → interview_sessions)
persona                     role          # "user" | "salma" etc.
stage                       message
started_at                  timestamp
ended_at
evaluation_json
score
label


cv_profiles                 job_requirements
───────────                 ────────────────
id (PK)              1──<  id (PK)
user_id (FK → users)        user_id (FK → users)
file_path                   cv_profile_id (FK → cv_profiles)
extracted_data_json         requirements_text
cv_hash                     job_hash
created_at                  generated_questions_json
                            created_at


real_interview_sessions          analysis_results
───────────────────────          ────────────────
id (PK)                   1──1  id (PK)
user_id (FK → users)             session_id (FK → real_interview_sessions)
cv_profile_id (FK)               emotion_json
job_req_id (FK)                  gaze_json
status                           pose_json
video_path                       snapshots_json
timeline_path                    overall_scores
interview_transcript             non_verbal_feedback
started_at                       transcript_evaluation_json
ended_at                         created_at

Relationships:
  users ──< interview_sessions (via user_id)
  users ──< cv_profiles (via user_id)
  users ──< job_requirements (via user_id)
  users ──< real_interview_sessions (via user_id)
  interview_sessions ──< session_messages
  cv_profiles ──< job_requirements
  cv_profiles ──< real_interview_sessions
  job_requirements ──< real_interview_sessions
  real_interview_sessions ──1 analysis_results
```

---

## 7. Scoring Formulas

### Non-Verbal Score (without coding task)
```
Overall = (Emotion × 0.25) + (Eye Contact × 0.30) + (Posture × 0.25) + (Tone × 0.20)
```

### Acceptance Rate (with coding task)
```
Acceptance = (Emotion × 0.20) + (Eye Contact × 0.25) + (Posture × 0.20) + (Tone × 0.15) + (Coding × 0.20)
```

### Component Definitions
| Component | Source | Formula | Range |
|---|---|---|---|
| Emotion Score | Keras CNN | `positive_frames / total_frames` | 0.0 – 1.0 |
| Eye Contact | MediaPipe FaceMesh | `eye_contact_frames / total_frames` | 0.0 – 1.0 |
| Posture Score | MediaPipe Pose | `avg(1.0 - signals × 0.2) per frame` | 0.0 – 1.0 |
| Tone Score | Gemini / Librosa | `confidence×0.6 + (1-hesitation)×0.4` | 0.0 – 1.0 |
| Coding Score | Frontend | submitted=0.9 / timeout+code=0.6 / no code=0.3 | 0.3 – 0.9 |

**Positive emotions:** happy, neutral, surprise
**Eye contact threshold:** iris deviation < 0.06 (normalized)
**Posture signals:** hand_on_head, leaning_back, leaning_left/right, excessive_movement

### Warm-up Evaluation (Groq LLM)
```json
{
  "overall_score": 7.4,
  "communication_score": 8.0,
  "content_score": 7.0,
  "confidence_score": 7.2,
  "evaluation_confidence": "medium",
  "key_insight": "One-sentence summary",
  "strengths": ["specific point 1", "specific point 2"],
  "improvements": ["actionable tip 1", "actionable tip 2"],
  "summary": "Balanced paragraph feedback"
}
```

Performance bands: `0–3 Poor` | `3–5 Developing` | `5–7 Good` | `7–8.5 Strong` | `8.5–10 Excellent`

---

## 8. WebSocket Protocol

### Warm-up Interview (`/ws/warmup/interview?persona=friendly`)

**Client → Server:**
```json
{ "type": "audio", "data": "<base64 PCM chunk>" }
```

**Server → Client:**
```json
{ "type": "transcript_partial", "text": "I think..." }
{ "type": "transcript_final", "text": "I think the answer is..." }
{ "type": "audio_response", "data": "<base64 MP3>", "text": "AI reply" }
{ "type": "stage_change", "stage": "behavioral" }
{ "type": "server_end", "evaluation": { ... } }
```

### Real Interview (`/ws/real-interview?session_id=123`)

**Client → Server:**
```json
{ "type": "ready" }
{ "type": "next", "transcript": "User's answer text" }
```

**Server → Client:**
```json
{ "type": "question", "data": "Question text", "index": 0 }
{ "type": "ai_response", "text": "Acknowledged.", "next_question": "Next Q" }
{ "type": "coding_task_start", "data": { "question": "...", "starter_code": "...", "time_limit": 300 } }
{ "type": "interview_end" }
```

---

## 9. Audio Mixing Flow

```
Microphone Stream ──────────────────┐
                                    ▼
                          AudioContext (Web Audio API)
                                    │
TTS <audio> element ────────────────┤
                                    │
                          MediaStreamDestination ──→ MediaRecorder
                                    │
                         Speakers (user hears TTS)      │
                                                        ▼
                                              Combined .webm file
                                           (mic audio + TTS audio + video)
```

**Key notes:**
- `createMediaElementSource()` called once per TTS element — stored as `ttsElSource`
- TTS connected to BOTH speakers and recording mix simultaneously
- Fallback: if AudioContext unavailable → mic-only recording

---

## 10. Interview Stage Progression

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Stage 1    │───→│  Stage 2    │───→│  Stage 3    │
│  Warm-up    │    │  Behavioral │    │Communication│
│  2-3 Q's   │    │  2-3 Q's   │    │  2-3 Q's   │
└─────────────┘    └─────────────┘    └──────┬──────┘
                                             │
                                             ▼
                              ┌─────────────┐    ┌─────────────┐
                              │  Stage 4    │───→│  Stage 5    │───→ ✅ Done
                              │  Technical  │    │  Reflection │
                              │  2-3 Q's   │    │  2-3 Q's   │
                              └─────────────┘    └─────────────┘
```

- Each stage has a pool of 5–10 questions — 2–3 are selected randomly per session
- No question repeats within a session
- **Free Mode:** User can manually advance/skip stages
- Stage changes are sent via WebSocket `{ "type": "stage_change" }`
