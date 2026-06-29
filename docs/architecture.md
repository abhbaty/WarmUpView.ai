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
│  │              Core Services                               │  │
│  │  LLM Manager │ Question Engine │ Auth │ Session Manager  │  │
│  └──────────────────────────────────────────────────────────┘  │
│                         │                                      │
│  ┌──────────────────────▼───────────────────────────────────┐  │
│  │         Analysis Worker (background subprocess)          │  │
│  │  Emotion (Keras) │ Gaze (MediaPipe) │ Pose (MediaPipe)  │  │
│  └──────────────────────┬───────────────────────────────────┘  │
└─────────────────────────┼───────────────────────────────────────┘
                          ▼
             ┌─────────────────────┐
             │      SQLite DB      │
             │   + File Storage    │
             └─────────────────────┘

External APIs:
  Groq (LLM) │ Google Gemini │ Deepgram (STT) │ Edge-TTS
```

---

## 2. Backend Modules

### `backend/app/main.py`
FastAPI entry point — configures CORS, mounts all sub-routers, creates DB tables on startup.

### `backend/app/models.py`
SQLAlchemy ORM definitions for all 6 database tables.

### `backend/app/config.py`
Centralizes all environment variables: API keys, file system paths, feature flags.

### `backend/app/database.py`
Async SQLAlchemy engine + session factory + startup/shutdown lifecycle.

### `backend/app/warmup/`
| File | Purpose |
|---|---|
| `routes.py` | REST endpoints for sessions CRUD + WebSocket upgrade |
| `websocket_handler.py` | Real-time interview logic: STT buffer, LLM routing, TTS, stage management |

### `backend/app/real_interview/`
| File | Purpose |
|---|---|
| `routes.py` | CV upload, job requirements, session management, video upload, report retrieval |
| `websocket_handler.py` | Live interview WebSocket: question delivery, acknowledgment, session end signal |

### `backend/app/analysis/`
| File | Purpose |
|---|---|
| `analysis_worker.py` | Spawns as subprocess via `Popen`, runs all 5 pipeline steps |
| `emotion_analyzer.py` | Loads Keras CNN model, runs per-frame emotion classification |
| `gaze_analyzer.py` | MediaPipe FaceMesh — iris deviation → eye contact boolean |
| `pose_analyzer.py` | MediaPipe Pose — 33 landmarks → posture score + signal detection |
| `video_processor.py` | OpenCV frame extraction at 1 fps |
| `report_generator.py` | Calls Gemini to synthesize raw metrics into qualitative feedback |

### `backend/app/core/`
| File | Purpose |
|---|---|
| `llm_manager.py` | Unified wrapper for Groq and Gemini APIs |
| `question_engine.py` | Stage-based question pool management, random selection, no-repeat logic |
| `vtt_service.py` | Voice-to-text helper utilities |

---

## 3. Frontend Modules

```
frontend/
├── index.html              ← Dashboard / navigation hub
├── admin.html              ← Admin panel (user management)
├── profile.html            ← User profile & settings
├── auth/
│   ├── login.html
│   └── signup.html
├── warmup/
│   └── session.html        ← Live warm-up interface + avatar
├── real_interview/
│   ├── setup.html          ← CV upload + job description entry
│   ├── interview.html      ← Live interview + Monaco editor
│   ├── history.html        ← Session history with filter
│   └── report.html         ← Multimodal analytics report
├── cv_analysis/
│   └── index.html          ← CV upload + ATS analysis viewer
├── css/
│   └── style.css           ← Global design system
└── js/
    ├── layout.js           ← Shared nav, theme toggle, auth guard
    ├── history.js          ← Status polling, filter, bulk delete
    ├── avatar.js           ← Animated AI avatar + lip-sync
    └── app.js              ← Dashboard logic
```

---

## 4. AI Services Routing

```
Warm-up Module
├── Conversation (all turns)    →  Groq  (llama-3.3-70b-versatile)
└── Evaluation Report           →  Groq  (llama-3.3-70b-versatile)

Real Interview Module
├── Question Generation         →  Groq  (llama-3.3-70b-versatile)
├── Coding Task Generation      →  Groq  (llama-3.3-70b-versatile)
├── Audio Tone Analysis         →  Gemini (gemini-2.5-flash, audio inline)
└── Tone Fallback               →  Librosa (local)

CV Analysis Module
└── ATS Feedback + Annotations  →  Gemini (gemini-2.5-flash)
```

**Why split?**
- Groq is used for high-frequency, latency-sensitive text generation (conversation turns, real-time)
- Gemini is used for multimodal tasks (audio understanding, document analysis)

---

## 5. Analysis Pipeline

```
Recorded Video (.webm)
        │
        ▼
┌───────────────────────────────────┐
│ Step 1: Frame Extraction (OpenCV) │
│   → 1 frame per second            │
│   → frames[] array in memory      │
└──────────────┬────────────────────┘
               │ frames[]
       ┌───────┴────────┐
       │                │ (parallel)
       ▼                ▼
┌─────────────┐  ┌─────────────────┐  ┌──────────────────┐
│  Emotion    │  │ Gaze Analyzer   │  │  Pose Analyzer   │
│  Analyzer   │  │ MediaPipe Face  │  │  MediaPipe Pose  │
│  Keras CNN  │  │ Mesh + Iris     │  │  33 landmarks    │
│  48x48 gray │  │ deviation<0.06  │  │  posture signals │
│  7 emotions │  │ → eye_contact   │  │  → posture_score │
└──────┬──────┘  └───────┬─────────┘  └────────┬─────────┘
       └──────────────────┼────────────────────┘
                          ▼
              ┌───────────────────────┐
              │  Step 3: Timeline     │
              │  Builder              │
              │  Merge per-second     │
              │  JSON from all 3      │
              └──────────┬────────────┘
                         │
              ┌──────────┴──────────┐
              │                     │
              ▼                     ▼
   ┌──────────────────┐  ┌────────────────────┐
   │ Step 4: Peak     │  │ Step 5: Score      │
   │ Snapshots        │  │ Calculator         │
   │ Extract key      │  │ Weighted formula   │
   │ frames as JPEG   │  │ → overall_score    │
   └──────┬───────────┘  └────────┬───────────┘
          └──────────┬────────────┘
                     ▼
          ┌────────────────────────┐
          │  SQLite DB             │
          │  analysis_results table│
          └────────────────────────┘
```

---

## 6. Database Schema (ERD)

```
interview_sessions          session_messages
─────────────────           ────────────────
id (PK)              1──<  id (PK)
user_id                     session_id (FK)
persona                     role
stage                       message
started_at                  timestamp
ended_at
evaluation_json
label


cv_profiles                 job_requirements
───────────                 ────────────────
id (PK)              1──<  id (PK)
file_path                   cv_profile_id (FK)
extracted_data_json         requirements_text
cv_hash                     job_hash
created_at                  generated_questions_json
                            created_at


real_interview_sessions     analysis_results
───────────────────────     ────────────────
id (PK)              1──1  id (PK)
cv_profile_id (FK)          session_id (FK)
job_req_id (FK)             emotion_json
status                      gaze_json
video_path                  pose_json
timeline_path               snapshots_json
started_at                  overall_scores
ended_at                    non_verbal_feedback
                            created_at

cv_profiles ──< real_interview_sessions
job_requirements ──< real_interview_sessions
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

### Warm-up Evaluation (Groq)
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

### Warm-up Interview (`/ws/warmup/interview/{session_id}`)

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

### Real Interview (`/ws/real-interview/{session_id}`)

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
