# WarmUpView 3rd - UI Documentation

This document outlines the complete user interface structure for the WarmUpView platform, detailing every screen from authentication to the final performance reports.

---

## 1. 🔐 Authentication & Onboarding

### 1.1. Login Page
The entry point for returning users. Features a clean, professional design with email and password fields.
![Login Page](./UI/1_login_fixed_1777579561399.png)
*Figure 1: Login Page*

### 1.2. Sign Up Page
The registration page for new users to create their accounts.
![Sign Up Page](./UI/2_signup_1777579541762.png)
*Figure 2: Sign Up Page*

---

## 2. 🏠 Main Dashboard (Lobby)

The central navigation hub where users select which module they want to practice.
- Features large, interactive cards/buttons for: Warm-up Interview, Real Interview, CV Analysis, and History.
- Displays user profile summary and quick stats.

![Main Dashboard (Dark Mode)](./UI/02.png)
*Figure 3: Main Dashboard (Dark Mode)*

![Main Dashboard (Light Mode)](./UI/01.png)
*Figure 4: Main Dashboard (Light Mode)*

---

## 3. 🎙️ Warm-up Interview Simulator

A low-pressure environment for verbal practice without video recording.

### 3.1. Persona Selection
Users choose their AI interviewer based on the strictness and style they prefer (Friendly Salma, Strict Mr. Hesham, Technical Alex, or a Mixed Panel).
![Persona Selection Overview](./UI/4_warmup_persona_1777579594826.png)
*Figure 5: Persona Selection Overview*

![Persona: Salma (Friendly)](./UI/%232.png)
*Figure 6: Persona: Salma (Friendly)*

![Persona: Alex (Peer)](./UI/%233.png)
*Figure 7: Persona: Alex (Peer)*

![Persona: Mr. Hesham (Strict)](./UI/%234.png)
*Figure 8: Persona: Mr. Hesham (Strict)*

![Persona: Mixed Panel](./UI/%235.png)
*Figure 9: Persona: Mixed Panel*

### 3.2. Live Warm-up Session
The conversational interface featuring the animated AI avatar, a live audio visualizer/waveform, and real-time speech-to-text transcriptions.
![Warm-up Session (Initial)](./UI/5_warmup_live_1777579600206.png)
*Figure 10: Warm-up Session (Initial)*

![Warm-up Session (Active)](./UI/%236.png)
*Figure 11: Warm-up Session (Active)*

![Warm-up Session (Transcript View)](./UI/%237.png)
*Figure 12: Warm-up Session (Transcript View)*

![Warm-up Session (End Session View)](./UI/%238.png)
*Figure 13: Warm-up Session (Conversation View)

---

## 4. 🧑💼 Real Interview Simulator

This module simulates a high-pressure interview environment. It consists of three main phases:

### 4.1. Setup Page (Pre-Interview)
Users upload their CV and provide the Job Description to generate tailored questions.
![Real Interview Setup](./UI/6_real_setup_1777579611760.png)
*Figure 14: Real Interview Setup*

![Real Interview Gateway](./UI/2.png)
*Figure 15: Real Interview Gateway*

### 4.2. Live Interview Phase (Recording & AI Conversation)
The core interview experience where the user is recorded via webcam. The AI interviewer delivers questions verbally, and the system records the user's audio and video for post-session analysis.
![Live Interview Phase](./UI/3.png)
*Figure 16: Live Interview Phase*

### 4.3. Coding Challenge Phase (Coding Screen)
If a coding task is triggered, the interface transitions into a dedicated coding environment featuring a Monaco code editor, the problem instructions, and a live countdown timer.
![Coding Challenge Phase](./UI/4.png)
*Figure 17: Coding Challenge Phase*

### 4.4. Analysis Processing
While the system processes the multimodal data, the user sees a real-time progress indicator.
![Analysis Processing](./UI/5_finishing_interview.png)
*Figure 18: Analysis Processing*

---

## 5. 📄 CV Analysis Dashboard

The module dedicated to reviewing and improving the user's resume.

### 5.1. Setup & CV Preview
The interface where the user uploads their PDF, inputs the job description, and previews the document.

![CV Analysis Page(Light Mode)](./UI/%2311.png)
*Figure 19: CV Analysis Page(Light Mode)*

![CV Analysis Page(Dark Mode)](./UI/%2310.png)
*Figure 20: CV Analysis Page(Dark Mode)*

![Job Input View](./UI/%2312.png)
*Figure 21: After upload the cv view

### 5.2. ATS Score & Detailed Breakdown
An overall ATS compatibility score, alongside detailed breakdowns of skills, experience, and education matches.
![Score Breakdown](./UI/%2314.png)
*Figure 22: Score Breakdown*

### 5.3. Gaps Analysis & Action Plan
Highlights critical skill and experience gaps, providing recommended steps and an actionable roadmap to improve the resume.
![Gaps Analysis](./UI/%2315.png)
*Figure 23: Gaps Analysis*

![Action Plan](./UI/%23222%23.png)
*Figure 24: Action Plan*

### 5.4. Red-Pen Feedback & Annotations
The analysis view displaying surgical, hand-drawn style annotations (checkmarks, X-marks, and connectors) directly on the CV.
![CV Annotations Report](./UI/%2313.png)
*Figure 25: CV Annotations Report*

---

## 6. 🗂️ History & Progress

The tracking dashboard where users can review all their past sessions and track improvement over time.
- A list/table of all past Warm-up and Real interviews.
- Status indicators (Completed, Failed, Processing).

![History Overview](./UI/8_history_1777579632933.png)
*Figure 26: History Overview*

![Detailed History View](./UI/%231.png)
*Figure 27: Detailed History View*

---

## 7. 📊 Interview Performance Report

After completing a Real Interview, the system generates a comprehensive, multimodal performance report. This dashboard provides actionable feedback across various dimensions:

### 7.1. Video Playback & Emotion Timeline
The recorded interview video is presented alongside a synced timeline waveform that highlights the emotional state of the user throughout the session.
![Video Playback (Original)](./UI/9.png)
*Figure 28: Video Playback (Original)*

![Interview Video Report (Update)](./UI/%239.png)
*Figure 29: Interview Video Report (Update)*

### 7.2. Performance Matrix (The 5 Rings)
A visual dashboard displaying the user's scores across five key metrics: Overall Acceptance Rate, Emotion, Eye Contact, Posture, and Voice Tone.
![Performance Matrix Overview](./UI/6.png)
*Figure 30: Performance Matrix Overview*

### 7.3. Voice & Tone Analysis
A dedicated section detailing the acoustic features of the user's voice, including progress bars for Confidence, Fluency, and Pace.
![Voice and Tone Analysis](./UI/7.png)
*Figure 31: Voice and Tone Analysis*

### 7.4. Body Language & Posture Summary
A visual breakdown (Donut Chart) of the user's physical posture during the interview, showing the percentage of time spent in various states (e.g., Upright, Leaning, Hand on Head).
![Body Language & Pose Analysis](./UI/8.png)
*Figure 32: Body Language & Pose Analysis*

### 7.5. Peak Moment Snapshots
Key visual frames extracted automatically from the video recording to highlight moments of significant emotional or postural shifts.
![Peak Moment Snapshots](./UI/10.png)
*Figure 33: Peak Moment Snapshots*

### 7.6. Non-Verbal AI Feedback
A text-based summary generated by the AI explaining the user's non-verbal communication patterns and providing tips for improvement.
![Interview Conversation Log](./UI/11.png)
*Figure 34: Interview Conversation Log*
