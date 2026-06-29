<div align="center">

<img src="docs/screenshots/banner.png" alt="WarmUpView.ai Banner" width="100%"/>

# 🎙️ WarmUpView.ai

### AI-Powered Interview Preparation Platform

[![FastAPI](https://img.shields.io/badge/FastAPI-0.115.6-009688?style=for-the-badge&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![Google Gemini](https://img.shields.io/badge/Google_Gemini-2.5_Flash-4285F4?style=for-the-badge&logo=google&logoColor=white)](https://aistudio.google.com)
[![Groq](https://img.shields.io/badge/Groq-Llama_3.3_70B-00CC88?style=for-the-badge)](https://groq.com)
[![MediaPipe](https://img.shields.io/badge/MediaPipe-0.10.5-FF6F00?style=for-the-badge&logo=google&logoColor=white)](https://mediapipe.dev)
[![SQLite](https://img.shields.io/badge/SQLite-Database-003B57?style=for-the-badge&logo=sqlite&logoColor=white)](https://sqlite.org)
[![License](https://img.shields.io/badge/License-MIT-purple?style=for-the-badge)](LICENSE)

> **Practice interviews with AI, get real-time multimodal feedback on your voice, emotions, eye contact, posture, and CV — all in your browser.**

[✨ Features](#-features) • [🏗 Architecture](#-architecture) • [📸 Screenshots](#-screenshots) • [⚙️ Installation](#️-installation) • [📊 Tech Stack](#-tech-stack) • [🗄️ Database](#️-database-schema)

</div>

---

## ✨ Features

| Module | Description |
|---|---|
| 🎙️ **Warm-up Simulator** | Voice-only practice with 4 AI personas, real-time STT/TTS, 5-stage interview flow |
| 🎥 **Real Interview Mode** | Webcam recording, CV-tailored questions, live coding challenge, multimodal analysis |
| 📄 **CV Analyzer** | ATS scoring, skill gap detection, red-pen annotations, actionable roadmap |
| 📊 **Performance Report** | Emotion timeline, eye contact %, posture donut chart, voice tone, peak snapshots |
| 🗂️ **History Dashboard** | Track progress across sessions, filter by status, compare sessions |
| 🔐 **Auth System** | Secure login/signup with session management |

---

## 🏗 Architecture

### System Overview

![System Architecture](docs/images/system_architecture.png)

The platform is built on a **FastAPI** backend with **WebSocket**-driven real-time communication, connected to multiple AI services and a local ML pipeline for non-verbal analysis.

```
User Browser  ←──WebSocket / REST──→  FastAPI Backend  ←──→  Groq / Gemini / Deepgram / Edge-TTS
                                              │
                                    Analysis Worker (subprocess)
                                              │
                               ┌──────────────┼──────────────┐
                          Keras CNN      MediaPipe        Librosa
                         (Emotion)     (Gaze + Pose)     (Tone)
                                              │
                                         SQLite DB
```

---

### 🔄 Warm-up Interview Flow

![Warm-up Flow](docs/images/warmup_interview_flow.png)

The warm-up module uses a **5-stage conversation engine**:

```
Stage 1: Warmup  →  Stage 2: Behavioral  →  Stage 3: Communication
                                         →  Stage 4: Technical
                                         →  Stage 5: Reflection
```

Each stage selects 2–3 random questions from a pool. The system detects voice commands (`"end session"`) and auto-advances stages when quotas are met. Silence is handled gracefully with progressive nudges at 20s, 40s, and 60s.

---

### 🤖 AI Services Routing

| Feature | Service | Model |
|---|---|---|
| Warm-up conversation | **Groq** | `llama-3.3-70b-versatile` |
| Warm-up evaluation | **Groq** | `llama-3.3-70b-versatile` |
| Question generation (Real Interview) | **Groq** | `llama-3.3-70b-versatile` |
| Coding challenge generation | **Groq** | `llama-3.3-70b-versatile` |
| Audio tone analysis (primary) | **Gemini** | `gemini-2.5-flash` (audio inline) |
| Audio tone analysis (fallback) | **Librosa** | Local — confidence + hesitation + pace |
| CV analysis & feedback | **Gemini** | `gemini-2.5-flash` |

---

### 📊 Analysis Pipeline

![Analysis Pipeline](docs/images/analysis_pipeline.png)

After a Real Interview is completed, a **background subprocess** performs 5 steps:

1. **Frame Extraction** — OpenCV extracts 1 frame/sec from the recorded video
2. **Parallel Analysis** — Three analyzers run simultaneously:
   - 🔴 **Emotion Analyzer** — Keras CNN (48×48 grayscale → 7 emotion classes)
   - 🔵 **Gaze Analyzer** — MediaPipe FaceMesh iris landmarks → eye contact boolean
   - 🟢 **Pose Analyzer** — MediaPipe Pose 33 landmarks → posture score
3. **Timeline Builder** — Merges per-second JSON data from all analyzers
4. **Peak Snapshot Extractor** — Saves key frames as JPEG
5. **Score Calculator** — Computes weighted overall score

#### Scoring Formula

```
Overall Score = (Emotion × 0.25) + (Eye Contact × 0.30) + (Posture × 0.25) + (Tone × 0.20)
```

With Coding Task:
```
Acceptance Rate = (Emotion × 0.20) + (Eye Contact × 0.25) + (Posture × 0.20) + (Tone × 0.15) + (Coding × 0.20)
```

---

## 🗄️ Database Schema

![Database ERD](docs/images/database_erd.png)

Six core tables power the platform:

| Table | Purpose |
|---|---|
| `interview_sessions` | Warm-up session records with evaluation JSON |
| `session_messages` | Per-turn conversation transcript |
| `cv_profiles` | Extracted CV data with hash-based deduplication |
| `job_requirements` | Job descriptions and cached AI-generated questions |
| `real_interview_sessions` | Full interview session with video/timeline paths |
| `analysis_results` | Multimodal scores: emotion, gaze, pose, tone, snapshots |

---

## 📸 Screenshots

<table>
  <tr>
    <td align="center"><b>Dashboard</b></td>
    <td align="center"><b>Login</b></td>
  </tr>
  <tr>
    <td><img src="docs/screenshots/dashboard_dark.png" width="400"/></td>
    <td><img src="docs/screenshots/login.png" width="400"/></td>
  </tr>
  <tr>
    <td align="center"><b>Persona Selection</b></td>
    <td align="center"><b>Live Warm-up Session</b></td>
  </tr>
  <tr>
    <td><img src="docs/screenshots/warmup_persona.png" width="400"/></td>
    <td><img src="docs/screenshots/warmup_live.png" width="400"/></td>
  </tr>
  <tr>
    <td align="center"><b>Real Interview Setup</b></td>
    <td align="center"><b>Live Interview</b></td>
  </tr>
  <tr>
    <td><img src="docs/screenshots/real_interview_setup.png" width="400"/></td>
    <td><img src="docs/screenshots/real_interview_live.png" width="400"/></td>
  </tr>
  <tr>
    <td align="center"><b>Performance Report (5 Rings)</b></td>
    <td align="center"><b>Video + Emotion Timeline</b></td>
  </tr>
  <tr>
    <td><img src="docs/screenshots/report_rings.png" width="400"/></td>
    <td><img src="docs/screenshots/report_video.png" width="400"/></td>
  </tr>
  <tr>
    <td align="center"><b>CV Analysis</b></td>
    <td align="center"><b>Session History</b></td>
  </tr>
  <tr>
    <td><img src="docs/screenshots/cv_analysis.png" width="400"/></td>
    <td><img src="docs/screenshots/history.png" width="400"/></td>
  </tr>
</table>

---

## 📊 Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| **Backend** | Python 3.10+, FastAPI, Uvicorn | API server, WebSocket handling |
| **Database** | SQLite, SQLAlchemy (async) | Persistent data storage |
| **LLM (Primary)** | Groq — `llama-3.3-70b-versatile` | Fast inference for conversation & evaluation |
| **LLM (Secondary)** | Google Gemini 2.5 Flash | CV analysis, audio tone understanding |
| **Speech-to-Text** | Deepgram Nova-2 | Real-time streaming transcription |
| **Text-to-Speech** | Edge-TTS (Microsoft voices) | AI persona voices |
| **Emotion Detection** | Keras CNN + ONNX Runtime | 7-class facial emotion classification |
| **Gaze Tracking** | MediaPipe FaceMesh | Iris landmark-based eye contact detection |
| **Pose Analysis** | MediaPipe Pose | 33-landmark body language scoring |
| **Audio Analysis** | Librosa (fallback) | Confidence, hesitation, pace metrics |
| **CV Parsing** | PyPDF | PDF text extraction |
| **Frontend** | Vanilla HTML/CSS/JS | Zero-framework responsive UI |
| **Code Editor** | Monaco Editor (CDN) | In-browser coding challenge environment |
| **Video** | Web MediaRecorder API, FFmpeg | WebM recording, audio mixing |

---

## ⚙️ Installation

### Prerequisites

- Python 3.10+
- [Deepgram API Key](https://deepgram.com) — for real-time STT
- [Google Gemini API Key](https://aistudio.google.com) — for LLM & CV analysis
- [Groq API Key](https://console.groq.com) — for fast LLM inference

### 1. Clone the Repository

```bash
git clone https://github.com/abhbaty/WarmUpView.ai.git
cd WarmUpView.ai
```

### 2. Set Up Environment Variables

```bash
cd backend
copy .env.example .env
# Edit .env and fill in your API keys
```

Required keys in `.env`:
```env
GEMINI_API_KEY=your_gemini_key_here
GROQ_API_KEY=your_groq_key_here
DEEPGRAM_API_KEY=your_deepgram_key_here
```

### 3. Install Dependencies

```bash
cd backend
pip install -r requirements.txt
```

### 4. Run the Server

```bash
cd backend
uvicorn app.main:app --reload --port 8000
```

### 5. Open the App

Navigate to **http://localhost:8000** in your browser.

---

## 📁 Project Structure

```
WarmUpView.ai/
├── backend/
│   ├── app/
│   │   ├── main.py                  # FastAPI entry point, CORS, routers
│   │   ├── config.py                # Settings, API keys, file paths
│   │   ├── database.py              # SQLAlchemy async engine & sessions
│   │   ├── models.py                # ORM models (6 tables)
│   │   ├── warmup/                  # Warm-up interview module
│   │   │   ├── routes.py            # REST + WebSocket routes
│   │   │   └── websocket_handler.py # Real-time interview logic
│   │   ├── real_interview/          # Real interview module
│   │   │   ├── routes.py            # CV upload, sessions, reports
│   │   │   └── websocket_handler.py # Live interview WS
│   │   ├── analysis/                # ML analysis pipeline
│   │   │   ├── analysis_worker.py   # Background subprocess runner
│   │   │   ├── emotion_analyzer.py  # Keras CNN — 7 emotions
│   │   │   ├── gaze_analyzer.py     # MediaPipe iris tracking
│   │   │   ├── pose_analyzer.py     # MediaPipe body landmarks
│   │   │   ├── video_processor.py   # OpenCV frame extraction
│   │   │   └── report_generator.py  # Gemini score synthesis
│   │   ├── core/                    # Shared services
│   │   │   ├── llm_manager.py       # Groq/Gemini abstraction
│   │   │   ├── question_engine.py   # Stage-based Q&A engine
│   │   │   └── vtt_service.py       # Voice-to-text utilities
│   │   ├── auth/                    # Authentication
│   │   └── admin/                   # Admin panel
│   ├── requirements.txt
│   └── .env.example
├── frontend/
│   ├── index.html                   # Main dashboard
│   ├── admin.html                   # Admin panel
│   ├── profile.html                 # User profile
│   ├── warmup/                      # Warm-up UI
│   │   └── session.html
│   ├── real_interview/              # Real interview UI
│   │   ├── setup.html
│   │   ├── interview.html
│   │   └── report.html
│   ├── cv_analysis/                 # CV analysis UI
│   ├── auth/                        # Login / signup
│   ├── css/                         # Stylesheets
│   ├── js/                          # Shared JavaScript
│   └── images/
├── docs/
│   ├── images/                      # Architecture diagrams
│   ├── screenshots/                 # UI screenshots
│   └── architecture.md             # Detailed architecture docs
└── README.md
```

---

## 🤝 AI Personas

The warm-up module features 4 distinct AI interviewer personas:

| Persona | Style | Focus |
|---|---|---|
| 👩 **Salma** | Friendly & Supportive | Confidence building, general questions |
| 👨‍💼 **Mr. Hesham** | Strict & Professional | Pressure handling, formal tone |
| 👨‍💻 **Alex** | Technical & Peer-like | Technical depth, problem solving |
| 👥 **Mixed Panel** | Combined voices | Comprehensive, realistic panel simulation |

---

## 📝 How It Works

### Warm-up Interview
1. Select your AI persona → Session created in DB
2. AI greets you via TTS (Edge-TTS) + animated avatar lip-sync
3. Speak into microphone → streamed to Deepgram STT in real-time
4. Transcript buffered (3.5s silence threshold) → sent to Groq LLM
5. AI responds via TTS → saved to session transcript
6. Stage quota met → advance to next interview stage
7. Say **"End Session"** or complete all stages → Groq generates evaluation report

### Real Interview
1. Upload CV (PDF) → parsed by PyPDF, stored in DB
2. Paste job description → Groq generates 7 tailored questions
3. Interview begins: webcam records (WebM), AI asks questions via TTS
4. Optional: coding challenge in Monaco Editor with countdown timer
5. Session ends → video uploaded to server → analysis subprocess spawned
6. 5-step ML pipeline runs: frame extraction → 3 parallel analyzers → timeline → scoring
7. View comprehensive report: emotion spectrogram, eye contact %, posture chart, peak snapshots

---

## 📄 License

This project is licensed under the **MIT License** — see [LICENSE](LICENSE) for details.

---

<div align="center">

Built with ❤️ using FastAPI, Google Gemini, Groq, Deepgram & MediaPipe

**[⬆ Back to top](#%EF%B8%8F-warmupviewai)**

</div>
