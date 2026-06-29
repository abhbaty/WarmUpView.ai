# WarmUpView 3rd - Project Architecture Overview

This document provides a concise summary of the project's file structure and the primary function of each component.

---

## 1. Backend (FastAPI Application)
Core logic, database management, and API endpoints.

*   **`backend/app/main.py`**: The application entry point. Initializes FastAPI, configures CORS, and includes all sub-routers.
*   **`backend/app/models.py`**: Defines the SQLAlchemy database schema (Sessions, Analysis, CV Profiles, Transcripts).
*   **`backend/app/database.py`**: Handles database engine creation, session management, and connectivity logic.
*   **`backend/app/config.py`**: Centralizes environment variables, API keys (e.g., Google Gemini), and file system paths.
*   **`backend/app/warmup/`**: Contains routes and logic for the audio-only practice simulator (Warm-up mode).
*   **`backend/app/real_interview/`**: Manages real interview recording uploads, report retrieval, and CV analysis endpoints.
*   **`backend/app/core/`**: Shared services such as LLM managers, Question engines, Voice-to-Text (VTT), and Health checks.

---

## 2. AI Analysis System
Background processing and machine learning modules for interview evaluation.

*   **`backend/app/analysis/analysis_worker.py`**: A background task runner that monitors the processing queue and executes the analysis pipeline for new videos.
*   **`backend/app/analysis/emotion_analyzer.py`**: Implements the facial expression recognition logic using the `emotion_model.h5`.
*   **`backend/app/analysis/gaze_analyzer.py`**: Detects eye contact and calculates gaze metrics using MediaPipe.
*   **`backend/app/analysis/pose_analyzer.py`**: Analyzes body language, posture stability, and hand-to-face contact.
*   **`backend/app/analysis/video_processor.py`**: Utility for frame extraction and video property analysis.
*   **`backend/app/analysis/report_generator.py`**: Uses AI (Gemini) to synthesize raw metrics into qualitative feedback and scores.
*   **`emotion_model.h5`**: The pre-trained Keras model file used for emotion classification.

---

## 3. Frontend (Web UI)
Modern, responsive user interface built with HTML, Vanilla CSS, and JavaScript.

*   **`frontend/index.html`**: The main user dashboard and navigation hub.
*   **`frontend/real_interview/history.html`**: A unified history manager for both Warm-up and Real interviews, featuring status filters and bulk deletion.
*   **`frontend/real_interview/report.html`**: The professional analytics report viewer with video playback and emotion spectrograms.
*   **`frontend/warmup/`**: Contains the interactive simulator (`session.html`) and practice-specific views.
*   **`frontend/cv_analysis/`**: The dedicated module for uploading Resumes and receiving AI-driven ATS feedback.
*   **`frontend/js/history.js`**: Client-side logic for filtering, polling analysis status, and managing state persistence.
*   **`frontend/js/layout.js`**: Shared logic for navigation, theme management, and UI consistency across pages.

---

## 4. Data & Utilities
Configuration and persistent storage files.

*   **`backend/warmupview.db`**: The SQLite database file containing all session history and analysis data.
*   **`backend/requirements.txt`**: Lists all Python dependencies (FastAPI, SQLAlchemy, MediaPipe, etc.).
*   **`backend/.env`**: Stores sensitive configurations and API credentials.
*   **`prd.md`**: Product Requirement Document outlining the project's features and technical specifications.
*   **`project_roadmap.md`**: Tracks the development progress and future feature implementations.

---

## 5. Storage Directories
*   **`backend/recordings/`**: Stores uploaded interview videos.
*   **`backend/snapshots/`**: Stores extracted key frames for visual analysis.
*   **`backend/uploads/`**: Temporary storage for uploaded documents and CVs.
