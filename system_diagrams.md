# WarmUpView.ai — System Diagrams for Report

> [!IMPORTANT]
> All diagrams below are based on the **actual implemented system**. You can recreate them in Canva/Draw.io, or screenshot the Mermaid renders directly.

---

## 1. Use Case Diagram (Chapter 3)

```mermaid
graph TB
    subgraph WarmUpView.ai System
        UC1["Start Warm-up Interview"]
        UC2["Select AI Persona"]
        UC3["Speak with AI Interviewer"]
        UC4["End Session via Voice Command"]
        UC5["View Warm-up Evaluation"]
        UC6["Upload CV"]
        UC7["Enter Job Requirements"]
        UC8["Start Real Interview"]
        UC9["Record Video During Interview"]
        UC10["Upload Video for Analysis"]
        UC11["View Analysis Report"]
        UC12["Analyze CV"]
        UC13["View Interview History"]
        UC14["Delete Past Session"]
    end

    User((User))
    Gemini[/"Google Gemini API"/]
    Deepgram[/"Deepgram STT API"/]
    EdgeTTS[/"Edge-TTS Service"/]
    CVEngine[/"CV Analysis Engine"/]

    User --> UC1
    User --> UC2
    User --> UC3
    User --> UC4
    User --> UC5
    User --> UC6
    User --> UC7
    User --> UC8
    User --> UC9
    User --> UC10
    User --> UC11
    User --> UC12
    User --> UC13
    User --> UC14

    UC3 --> Deepgram
    UC3 --> Gemini
    UC3 --> EdgeTTS
    UC5 --> Gemini
    UC7 --> Gemini
    UC8 --> Gemini
    UC10 --> CVEngine
    UC12 --> Gemini

    style User fill:#4f46e5,stroke:#fff,color:#fff
    style Gemini fill:#1e293b,stroke:#818cf8,color:#c7d2fe
    style Deepgram fill:#1e293b,stroke:#34d399,color:#a7f3d0
    style EdgeTTS fill:#1e293b,stroke:#fbbf24,color:#fde68a
    style CVEngine fill:#1e293b,stroke:#f87171,color:#fca5a5
```

---

## 2. Activity Diagram — Warm-up Interview (Chapter 3)

```mermaid
flowchart TD
    A([Start]) --> B[User Opens Warm-up Page]
    B --> C{Select Persona}
    C -->|Friendly| D[Load Salma Config]
    C -->|Strict| E[Load Mr. Hesham Config]
    C -->|Technical| F[Load Alex Config]
    C -->|Panel| G[Load Mixed Panel Config]
    
    D & E & F & G --> H[Create Session in DB]
    H --> I[Send AI Greeting via TTS]
    I --> J[Connect Deepgram STT]
    J --> K[Set Stage = Warmup]

    K --> L{User Speaking?}
    L -->|Yes| M[Stream Audio to Deepgram]
    M --> N[Receive Transcript]
    N --> O[Buffer Transcript - Wait 3.5s Silence]
    O --> P[Flush Buffer to Gemini LLM]
    P --> Q[Validate AI Response]
    Q --> R{Valid?}
    R -->|Yes| S[Send Response via TTS]
    R -->|No| T[Retry with Repair Prompt]
    T --> Q
    S --> U[Save Messages to DB]
    U --> V{Stage Quota Met?}
    V -->|Yes| W[Advance to Next Stage]
    V -->|No| L
    W --> X{Final Stage Done?}
    X -->|No| L
    X -->|Yes| Y[Send Closing Message]

    L -->|No - Silent| Z{Silence Duration?}
    Z -->|20s| AA["Nudge: Take your time"]
    Z -->|40s| AB["Nudge: Repeat question?"]
    Z -->|60s| AC["Nudge: Skip question?"]
    Z -->|90s| AD[Auto-End Session]
    AA & AB & AC --> L

    L -->|Voice Command| AE{End Command Detected?}
    AE -->|Yes| Y

    Y --> AF[Generate AI Evaluation Report]
    AF --> AG[Save Evaluation to DB]
    AG --> AH[Close WebSocket]
    AH --> AI([End])
    AD --> Y

    style A fill:#4f46e5,color:#fff
    style AI fill:#4f46e5,color:#fff
    style Y fill:#f59e0b,color:#000
    style AF fill:#10b981,color:#fff
```

---

## 3. Activity Diagram — Real Interview (Chapter 3)

```mermaid
flowchart TD
    A([Start]) --> B[User Opens Real Interview Page]
    B --> C[Upload CV as PDF]
    C --> D[Extract Text via PyPDF]
    D --> E[Store CV Profile in DB]
    E --> F[Enter Job Requirements Text]
    F --> G[Send CV + Job Req to Gemini]
    G --> H[Generate 7 Tailored Questions]
    H --> I[Cache Questions in DB]
    I --> J[Create Real Interview Session]
    J --> K[Start WebSocket Connection]
    K --> L[Begin Webcam Recording]
    
    L --> M[Send First Question via TTS]
    M --> N[User Answers - Speech Recorded]
    N --> O[Capture Transcript via Browser STT]
    O --> P[Send Transcript to Gemini]
    P --> Q[AI Generates Acknowledgment]
    Q --> R{More Questions?}
    R -->|Yes| S[Display Next Question]
    S --> N
    R -->|No| T[End Interview Signal]
    
    T --> U[Stop Webcam Recording]
    U --> V[Upload Video to Server]
    V --> W[Spawn Analysis Worker Process]
    
    W --> X[Step 1: Extract Frames - 1/sec]
    X --> Y[Step 2: Run 3 Analyzers in Parallel]

    Y --> Y1[Emotion Analyzer - Keras CNN]
    Y --> Y2[Gaze Analyzer - MediaPipe FaceMesh]
    Y --> Y3[Pose Analyzer - MediaPipe Pose]

    Y1 & Y2 & Y3 --> Z[Step 3: Build Master Timeline]
    Z --> AA[Step 4: Extract Peak Snapshots]
    AA --> BB[Step 5: Compute Overall Scores]
    BB --> CC[Save Results to DB]
    CC --> DD[Update Session Status to Done]
    DD --> EE[Report Available for Viewing]
    EE --> FF([End])

    style A fill:#4f46e5,color:#fff
    style FF fill:#4f46e5,color:#fff
    style W fill:#f59e0b,color:#000
    style Y1 fill:#ef4444,color:#fff
    style Y2 fill:#3b82f6,color:#fff
    style Y3 fill:#10b981,color:#fff
```

---

## 4. Sequence Diagram — Warm-up Interview Session (Chapter 3)

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant FastAPI
    participant Deepgram
    participant Gemini
    participant EdgeTTS
    participant SQLite

    User->>Browser: Select Persona & Click Start
    Browser->>FastAPI: WebSocket Connect /ws/warmup/interview
    FastAPI->>SQLite: CREATE interview_session
    SQLite-->>FastAPI: session_id
    FastAPI->>Gemini: Load persona system prompt
    FastAPI->>EdgeTTS: Synthesize greeting
    EdgeTTS-->>FastAPI: Audio bytes (MP3)
    FastAPI-->>Browser: {type: audio_response, data: base64}
    Browser->>User: Play AI greeting + Avatar lip-sync
    FastAPI->>Deepgram: Open STT WebSocket
    Deepgram-->>FastAPI: Connected

    loop Conversation Loop
        User->>Browser: Speak into microphone
        Browser->>FastAPI: {type: audio, data: base64}
        FastAPI->>Deepgram: Stream audio bytes
        Deepgram-->>FastAPI: Transcript (partial/final)
        FastAPI-->>Browser: {type: transcript_partial}
        
        Note over FastAPI: Buffer accumulates 3.5s silence
        
        FastAPI->>Gemini: Flushed user text + stage context
        Gemini-->>FastAPI: AI response text
        
        Note over FastAPI: Validate & repair if needed
        
        FastAPI->>SQLite: Save user msg + AI msg
        FastAPI->>EdgeTTS: Synthesize AI response
        EdgeTTS-->>FastAPI: Audio bytes
        FastAPI-->>Browser: {type: transcript + audio_response}
        Browser->>User: Display text + Play audio + Avatar
        
        Note over FastAPI: Check stage quota → Advance if met
        FastAPI-->>Browser: {type: stage_change, stage: behavioral}
    end

    User->>Browser: Say "End Session"
    Browser->>FastAPI: Voice command detected
    FastAPI->>Gemini: Generate evaluation report
    Gemini-->>FastAPI: JSON evaluation
    FastAPI->>SQLite: Save evaluation_json
    FastAPI-->>Browser: {type: server_end}
    FastAPI->>Deepgram: Close STT stream
    Browser->>User: Redirect to Dashboard
```

---

## 5. Sequence Diagram — Real Interview Session (Chapter 4)

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant FastAPI
    participant Gemini
    participant EdgeTTS
    participant SQLite
    participant Worker as Analysis Worker

    User->>Browser: Upload CV (PDF)
    Browser->>FastAPI: POST /api/real-interview/cv/upload
    FastAPI->>FastAPI: Extract text via PyPDF
    FastAPI->>SQLite: Save cv_profiles
    FastAPI-->>Browser: cv_profile_id

    User->>Browser: Paste Job Description
    Browser->>FastAPI: POST /api/real-interview/job-requirements
    FastAPI->>Gemini: Generate 7 questions from CV + Job
    Gemini-->>FastAPI: JSON question list
    FastAPI->>SQLite: Save job_requirements + questions
    FastAPI-->>Browser: job_req_id

    User->>Browser: Click Start Interview
    Browser->>FastAPI: POST /api/real-interview/sessions
    FastAPI->>SQLite: CREATE real_interview_session
    Browser->>FastAPI: WebSocket /ws/real-interview
    Browser->>Browser: Start MediaRecorder (webcam)

    Browser->>FastAPI: {type: ready}
    FastAPI-->>Browser: {type: question, data: Q1, index: 0}
    Browser->>EdgeTTS: TTS for question text
    Browser->>User: Play question audio

    loop For Each Question (1-7)
        User->>Browser: Answer verbally
        Browser->>FastAPI: {type: next, transcript: "answer text"}
        FastAPI->>Gemini: Generate acknowledgment
        Gemini-->>FastAPI: "Great point, thank you."
        FastAPI-->>Browser: {type: ai_response, text, next_question}
        Browser->>User: Play acknowledgment + Show next Q
    end

    FastAPI-->>Browser: {type: interview_end}
    Browser->>Browser: Stop MediaRecorder
    Browser->>FastAPI: POST /sessions/{id}/upload-video
    FastAPI->>SQLite: Update video_path

    Browser->>FastAPI: POST /sessions/{id}/finish
    FastAPI->>Worker: Spawn subprocess (Popen)
    FastAPI-->>Browser: {status: processing}

    Note over Worker: Runs independently

    Worker->>Worker: Step 1 - Extract frames (OpenCV)
    Worker->>Worker: Step 2 - Emotion (Keras CNN)
    Worker->>Worker: Step 2 - Gaze (MediaPipe FaceMesh)
    Worker->>Worker: Step 2 - Pose (MediaPipe Pose)
    Worker->>Worker: Step 3 - Build timeline
    Worker->>Worker: Step 4 - Extract snapshots
    Worker->>Worker: Step 5 - Compute scores
    Worker->>SQLite: Save analysis_results
    Worker->>SQLite: Update status = "done"

    Browser->>FastAPI: Poll GET /sessions/{id}
    FastAPI->>SQLite: Check status
    FastAPI-->>Browser: {status: done}
    Browser->>User: Show Analysis Report
```

---

## 6. Data Flow Diagram — Level 0 (Context Diagram)

```mermaid
flowchart LR
    User((User))

    subgraph WarmUpView.ai
        System["WarmUpView.ai\nPlatform"]
    end

    User -->|"Voice Audio\nCV File\nJob Description"| System
    System -->|"AI Interview Questions\nTTS Audio\nEvaluation Reports\nCV Feedback\nAnalysis Scores"| User

    Deepgram[/"Deepgram API"/] -->|"Transcribed Text"| System
    System -->|"Audio Stream"| Deepgram

    Gemini[/"Google Gemini"/] -->|"AI Responses\nQuestions\nEvaluations"| System
    System -->|"Prompts\nTranscripts"| Gemini

    TTS[/"Edge-TTS"/] -->|"Speech Audio"| System
    System -->|"Text to Speak"| TTS

    style User fill:#4f46e5,color:#fff
    style System fill:#1e293b,stroke:#818cf8,color:#fff
    style Deepgram fill:#0f172a,stroke:#34d399,color:#a7f3d0
    style Gemini fill:#0f172a,stroke:#818cf8,color:#c7d2fe
    style TTS fill:#0f172a,stroke:#fbbf24,color:#fde68a
```

---

## 7. Data Flow Diagram — Level 1 (Detailed)

```mermaid
flowchart TB
    User((User))

    subgraph P1 ["1.0 Warm-up Interview"]
        P1A["1.1 Capture\nVoice"]
        P1B["1.2 Transcribe\nSpeech"]
        P1C["1.3 Generate\nAI Response"]
        P1D["1.4 Synthesize\nSpeech"]
        P1E["1.5 Evaluate\nSession"]
    end

    subgraph P2 ["2.0 Real Interview"]
        P2A["2.1 Upload &\nParse CV"]
        P2B["2.2 Generate\nQuestions"]
        P2C["2.3 Conduct\nInterview"]
        P2D["2.4 Record\nVideo"]
        P2E["2.5 Analyze\nVideo"]
    end

    subgraph P3 ["3.0 CV Analysis"]
        P3A["3.1 Extract\nCV Text"]
        P3B["3.2 AI\nFeedback"]
    end

    subgraph P4 ["4.0 History"]
        P4A["4.1 List\nSessions"]
        P4B["4.2 View\nReports"]
    end

    DB[(SQLite DB)]
    Files[(File Storage)]
    Deepgram[/"Deepgram"/]
    Gemini[/"Gemini AI"/]
    TTS[/"Edge-TTS"/]
    CV_ML[/"ML Pipeline\nEmotion+Gaze+Pose"/]

    User -->|"Voice"| P1A
    P1A -->|"Audio bytes"| P1B
    P1B -->|"via Deepgram"| Deepgram
    Deepgram -->|"Transcript"| P1C
    P1C -->|"Prompt"| Gemini
    Gemini -->|"Response"| P1D
    P1D -->|"Text"| TTS
    TTS -->|"Audio"| User
    P1C -->|"Messages"| DB
    P1E -->|"Transcript"| Gemini
    Gemini -->|"Scores JSON"| P1E
    P1E -->|"Evaluation"| DB

    User -->|"CV + Job Desc"| P2A
    P2A -->|"CV text"| P2B
    P2B -->|"Prompt"| Gemini
    Gemini -->|"Questions"| DB
    P2C -->|"Q&A flow"| User
    P2D -->|"Video file"| Files
    Files -->|"Video"| P2E
    P2E -->|"Frames"| CV_ML
    CV_ML -->|"Scores"| DB

    User -->|"CV PDF"| P3A
    P3A -->|"Text"| P3B
    P3B -->|"Prompt"| Gemini
    Gemini -->|"Feedback"| User

    DB -->|"Sessions"| P4A
    P4A -->|"List"| User
    DB -->|"Analysis"| P4B
    P4B -->|"Report"| User

    style User fill:#4f46e5,color:#fff
    style DB fill:#1e293b,stroke:#818cf8,color:#c7d2fe
    style Files fill:#1e293b,stroke:#f59e0b,color:#fde68a
```

---

## 8. Entity Relationship Diagram — Database (Chapter 4)

```mermaid
erDiagram
    interview_sessions {
        int id PK
        string user_id
        string persona
        string stage
        datetime started_at
        datetime ended_at
        text evaluation_json
        string label
    }

    session_messages {
        int id PK
        int session_id FK
        string role
        text message
        datetime timestamp
    }

    cv_profiles {
        int id PK
        string file_path
        text extracted_data_json
        string cv_hash
        datetime created_at
    }

    job_requirements {
        int id PK
        int cv_profile_id FK
        text requirements_text
        string job_hash
        text generated_questions_json
        datetime created_at
    }

    real_interview_sessions {
        int id PK
        int cv_profile_id FK
        int job_req_id FK
        string status
        string video_path
        string timeline_path
        datetime started_at
        datetime ended_at
    }

    analysis_results {
        int id PK
        int session_id FK
        text emotion_json
        text gaze_json
        text pose_json
        text snapshots_json
        text overall_scores
        text non_verbal_feedback
        datetime created_at
    }

    interview_sessions ||--o{ session_messages : "has many"
    cv_profiles ||--o{ job_requirements : "has many"
    cv_profiles ||--o{ real_interview_sessions : "has many"
    job_requirements ||--o{ real_interview_sessions : "has many"
    real_interview_sessions ||--o| analysis_results : "has one"
```

---

## 9. Analysis Pipeline Flow (Chapter 4)

```mermaid
flowchart LR
    subgraph Input
        V["Recorded Video\n(.webm)"]
    end

    subgraph Step1 ["Step 1: Frame Extraction"]
        FE["OpenCV\nExtract 1 frame/sec"]
    end

    subgraph Step2 ["Step 2: Parallel Analyzers"]
        direction TB
        EA["Emotion Analyzer\n Keras CNN\n48x48 grayscale\n7 emotions"]
        GA["Gaze Analyzer\nMediaPipe FaceMesh\nIris landmarks\nEye contact bool"]
        PA["Pose Analyzer\nMediaPipe Pose\n33 body landmarks\nPosture score"]
    end

    subgraph Step3 ["Step 3: Timeline"]
        TL["Timeline Builder\nMerge all data\nper-second JSON"]
    end

    subgraph Step4 ["Step 4: Snapshots"]
        SN["Peak Detector\nExtract key frames\nSave as JPEG"]
    end

    subgraph Step5 ["Step 5: Scoring"]
        SC["Score Calculator"]
    end

    subgraph Output
        DB[(SQLite\nanalysis_results)]
    end

    V --> FE
    FE -->|"frames[]"| EA
    FE -->|"frames[]"| GA
    FE -->|"frames[]"| PA
    EA -->|"emotion, confidence"| TL
    GA -->|"eye_contact"| TL
    PA -->|"posture_score, signals"| TL
    TL --> SN
    TL --> SC
    SN -->|"snapshot paths"| DB
    SC -->|"overall scores"| DB

    style EA fill:#ef4444,color:#fff
    style GA fill:#3b82f6,color:#fff
    style PA fill:#10b981,color:#fff
    style DB fill:#1e293b,stroke:#818cf8,color:#c7d2fe
```

---

## 10. Overall Score Formula (Chapter 4)

The overall non-verbal performance score is computed as a weighted average of three components:

```
Overall Score = (Emotion Score × 0.3) + (Eye Contact Score × 0.4) + (Posture Score × 0.3)
```

Where:

| Component | Formula | Range |
|---|---|---|
| **Emotion Score** | `positive_frames / total_frames` | 0.0 – 1.0 |
| **Eye Contact Score** | `eye_contact_frames / total_frames` | 0.0 – 1.0 |
| **Posture Score** | `avg(posture_score per frame)` | 0.0 – 1.0 |

- **Positive emotions** = {happy, neutral, surprise}
- **Eye contact** = iris deviation < 0.06 (normalized)
- **Posture score** = `1.0 - (detected_signals × 0.2)` where signals include: hand_on_head, leaning_back, leaning_left/right, excessive_movement

---

## 11. Warm-up Evaluation Scoring (Chapter 4)

The AI evaluation uses Gemini to generate scores in a structured JSON format:

```
┌─────────────────────────────────────────────┐
│           AI Evaluation Report              │
├─────────────────────────────────────────────┤
│  overall_score        = avg(all scores)     │
│  communication_score  = 0–10                │
│  content_score        = 0–10                │
│  confidence_score     = 0–10                │
├─────────────────────────────────────────────┤
│  Performance Levels:                        │
│  0–3:   Poor                                │
│  3–5:   Developing                          │
│  5–7:   Good                                │
│  7–8.5: Strong                              │
│  8.5–10: Excellent                          │
├─────────────────────────────────────────────┤
│  evaluation_confidence: low/medium/high     │
│  key_insight: "one sentence summary"        │
│  strengths[]: specific positive points      │
│  improvements[]: max 2 actionable items     │
│  summary: balanced paragraph                │
└─────────────────────────────────────────────┘
```

**Fairness Rules:**
- Short session → lower confidence rating, NOT harsh scoring
- Struggling answers → partial credit
- Hesitation, filler words → NOT heavily penalized
- Sessions with < 10 total user words → automatic "too short" evaluation (no Gemini call)

---

## 12. Interview Stage Progression (Chapter 4)

```mermaid
flowchart LR
    S1["Stage 1\nWarmup\n2-3 Questions"]
    S2["Stage 2\nBehavioral\n2-3 Questions"]
    S3["Stage 3\nCommunication\n2-3 Questions"]
    S4["Stage 4\nTechnical\n2-3 Questions"]
    S5["Stage 5\nReflection\n2-3 Questions"]
    DONE["Session\nComplete"]

    S1 -->|"Quota met"| S2
    S2 -->|"Quota met"| S3
    S3 -->|"Quota met"| S4
    S4 -->|"Quota met"| S5
    S5 -->|"Final answer"| DONE

    style S1 fill:#818cf8,color:#fff
    style S2 fill:#a78bfa,color:#fff
    style S3 fill:#c084fc,color:#fff
    style S4 fill:#e879f9,color:#fff
    style S5 fill:#f472b6,color:#fff
    style DONE fill:#10b981,color:#fff
```

Each stage has a question pool (5–10 questions). The engine randomly selects 2–3 per stage, ensuring no repeats. In **Free Mode**, the user can manually switch stages.

---

## 13. AI Services Routing (Updated — Phase 2 Refactoring)

> [!IMPORTANT]
> After the Phase 2 refactoring, AI services are now split between **Groq** and **Gemini** based on the feature:

```mermaid
flowchart TB
    subgraph Warm-Up Module
        WU1["LLM Responses\n(Conversation)"]
        WU2["Evaluation Report\n(Scoring)"]
    end

    subgraph Real Interview Module
        RI1["Question Generation\n(7 tailored questions)"]
        RI2["Coding Task Generation\n(POST /interview/coding-task)"]
        RI3["Audio Tone Analysis\n(Gemini Audio Understanding)"]
        RI4["Tone Fallback\n(Librosa audio processing)"]
    end

    Groq[/"Groq API\nllama-3.3-70b-versatile"/]
    Gemini[/"Google Gemini\ngemini-2.5-flash"/]
    Librosa[/"Librosa\nLocal audio lib"/]

    WU1 --> Groq
    WU2 --> Groq
    RI1 --> Groq
    RI2 --> Groq
    RI3 --> Gemini
    RI3 -->|"on failure"| RI4
    RI4 --> Librosa

    style Groq fill:#10b981,color:#fff
    style Gemini fill:#818cf8,color:#fff
    style Librosa fill:#f59e0b,color:#000
```

**Routing Logic:**
| Feature | Service | Model/Lib |
|---|---|---|
| Warm-up conversation | Groq | `llama-3.3-70b-versatile` |
| Warm-up evaluation | Groq | `llama-3.3-70b-versatile` |
| Real interview question generation | Groq | `llama-3.3-70b-versatile` |
| Coding challenge generation | Groq | `llama-3.3-70b-versatile` |
| Audio tone analysis (primary) | Gemini | `gemini-2.5-flash` (audio inline) |
| Audio tone analysis (fallback) | Librosa | Local — confidence + hesitation + pace |
| WebSocket real-time acknowledgment | **Unchanged** | (not modified) |

---

## 14. Audio Mixing Flow (New Feature)

```mermaid
flowchart LR
    Mic["Microphone\nStream"]
    TTS["TTS Audio\n<audio> element"]
    AC["AudioContext\n(Web Audio API)"]
    MD["MediaStreamDestination\n(mix output)"]
    MR["MediaRecorder\n(combined stream)"]
    VID["Video File\n(.webm)"]
    SPK["Speakers\n(user hears TTS)"]

    Mic -->|"createMediaStreamSource"| AC
    TTS -->|"createMediaElementSource"| AC
    AC -->|"connect"| MD
    AC -->|"connect"| SPK
    MD -->|"audio track"| MR
    Mic -->|"video track"| MR
    MR --> VID
```

**Key Implementation Notes:**
- `createMediaElementSource()` can only be called **once** per element — stored as `ttsElSource`
- TTS is connected to **both** `audioCtx.destination` (speakers) and `mixDest` (recording)
- Combined stream = webcam video tracks + mixed audio tracks
- Fallback: if AudioContext fails → recording uses mic-only stream

---

## 15. Coding Task Feature Flow (New Feature)

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant FastAPI
    participant Groq

    Note over Browser,FastAPI: WebSocket already open (interview in progress)

    FastAPI-->>Browser: {type: coding_task_start, data: {question, starter_code, time_limit}}
    Browser->>Browser: Show Split Screen (#split-screen)
    Browser->>Browser: Initialize Monaco Editor (Python)
    Browser->>Browser: Start Countdown Timer

    User->>Browser: Write code in Monaco Editor
    
    alt User submits code
        User->>Browser: Click Submit
        Browser->>Browser: Capture editor code
        Browser->>Browser: Compute score (0.9 = submitted)
        Browser->>Browser: Close Split Screen
    else Timer expires
        Browser->>Browser: Auto-submit (score = 0.6 if code present)
    else User skips
        User->>Browser: Click Skip
        Browser->>Browser: Score = null (not attempted)
    end

    Note over Browser: coding_task_score saved in memory
    Note over Browser: Included in session report when available
```

**Coding Task Scoring:**
| Scenario | Score |
|---|---|
| Submitted with code | 0.9 (90%) |
| Timed out with partial code | 0.6 (60%) |
| Timed out without code | 0.3 (30%) |
| Skipped | null (not shown in report) |

---

## 16. Updated Overall Score Formula (Phase 2)

The overall non-verbal performance score now includes **Tone Score**:

```
Overall Score = (Emotion × 0.25) + (Eye Contact × 0.30) + (Posture × 0.25) + (Tone × 0.20)
```

**Acceptance Rate** (shown in report dashboard):

**Without Coding Task:**
```
Acceptance Rate = Emotion(0.25) + Eye Contact(0.30) + Posture(0.25) + Tone(0.20)
```

**With Coding Task:**
```
Acceptance Rate = Emotion(0.20) + Eye Contact(0.25) + Posture(0.20) + Tone(0.15) + Coding(0.20)
```

| Component | Source | Range |
|---|---|---|
| Emotion Score | Keras CNN — % positive frames | 0.0 – 1.0 |
| Eye Contact Score | MediaPipe FaceMesh | 0.0 – 1.0 |
| Posture Score | MediaPipe Pose | 0.0 – 1.0 |
| Tone Score | Gemini/Librosa | confidence×0.6 + (1-hesitation)×0.4 |
| Coding Score | Frontend scoring | 0.3 / 0.6 / 0.9 |

---

## 17. MediaPipe Pose Skeleton Overlay Flow

```mermaid
flowchart LR
    WC["Webcam Video\n(live)"]
    MP["MediaPipe Pose JS\n(CDN — lite model)"]
    LM["Landmarks\n33 body points"]
    CONF{"confidence\n> 0.7?"}
    DRAW["Draw Skeleton\non Canvas overlay"]
    SKIP["Clear Canvas\n(skip frame)"]

    WC -->|"every animation frame"| MP
    MP --> LM
    LM --> CONF
    CONF -->|"Yes"| DRAW
    CONF -->|"No"| SKIP
```

**Canvas Connections Drawn:**
- Shoulders (11↔12), Arms (11→13→15, 12→14→16)
- Torso (11→23, 12→24, 23↔24)
- Legs (23→25→27, 24→26→28)
- Color: `rgba(99, 102, 241, 0.7)` — indigo/violet dots & lines

---

> [!TIP]
> **How to use these diagrams:**
> 1. **For Canva/Draw.io:** Use these as blueprints — recreate them visually with prettier icons and colors
> 2. **For direct use:** If your report supports Mermaid, paste the code blocks directly
> 3. **For screenshots:** Open in a Mermaid viewer (GitHub, Notion, VS Code + Mermaid extension)

> [!IMPORTANT]
> **Key diagrams for your report chapters:**
> - Chapter 3: Use Case (#1), Activity Diagrams (#2, #3), Sequence Diagrams (#4, #5)
> - Chapter 4: ER Diagram (#8), Analysis Pipeline (#9), Score Formulas (#10, #11, #16), DFD (#6, #7), Stage Progression (#12)
> - **Phase 2 Updates:** AI Routing (#13), Audio Mixing (#14), Coding Task (#15), Pose Skeleton (#17)
