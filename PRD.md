# Product Requirements Document (PRD)
## Automated Coding Practice Tracker
**Author:** Vishakha Singh | **Version:** 1.0 | **Status:** Under Development

---

## 1. Product Overview

A full-stack intelligent DSA practice platform that tracks solved coding problems, applies SM-2 spaced repetition for optimized revision scheduling, and uses AI to predict problem difficulty per user — helping competitive programmers retain knowledge long-term and prepare systematically for SDE interviews.

---

## 2. Problem Statement

Competitive programmers solve hundreds of problems but lack:
- A centralized system to track what they've solved
- Structured revision scheduling based on memory science
- Personalized difficulty assessment (LeetCode's difficulty is static, not user-specific)
- Intelligent hints to unblock without giving away solutions

This leads to wasted revision time, re-solving forgotten problems, and weak retention during interviews.

---

## 3. Target Users

| User Type | Description |
|-----------|-------------|
| Primary | CSE students preparing for SDE placements/internships |
| Secondary | Working professionals upskilling for FAANG-level interviews |

---

## 4. Goals

- Help users build long-term DSA retention through spaced repetition
- Predict how hard a specific problem will be *for that user* using their history
- Provide AI-generated hints that guide without spoiling
- Be CV-worthy: production-level multi-user full-stack Python project

---

## 5. Features

### 5.1 Authentication
- User registration and login
- JWT-based session management
- Each user's data is fully isolated

### 5.2 Problem Management
- Add problems manually (title, topic, platform, difficulty, notes)
- Fetch solved problems from coding platforms (LeetCode etc.) via API/scraping
- Tag problems by DSA topic (Arrays, Trees, DP, Graphs, etc.)

### 5.3 Spaced Repetition Engine (SM-2)
- After solving, user self-rates recall (0–5 scale)
- SM-2 algorithm calculates next review date based on rating + easiness factor
- Daily revision queue generated automatically
- Review intervals adjust dynamically over time

### 5.4 AI Difficulty Prediction
- Predicts how difficult a problem will be *for the specific user* based on:
  - Their past performance on similar topics
  - Time taken, rating history, failure patterns
- Outputs a personalized difficulty score (not LeetCode's static Easy/Medium/Hard)
- Model improves as user solves more problems

### 5.5 AI Hint Generation
- User can request a hint on any problem
- Hints are progressive: Level 1 (nudge) → Level 2 (approach) → Level 3 (near-solution)
- Powered by LLM API (e.g., OpenAI/Gemini/Claude)
- Hints do not give away full solution

### 5.6 Dashboard
- Today's revision queue
- Problems due soon
- Performance stats (problems solved, streak, weak topics)
- Personalized difficulty predictions for queued problems

### 5.7 Analytics
- Topic-wise performance breakdown
- Retention score over time
- Weak area detection

### 5.8 Document Export
- Export revision plan or problem notes as Word (.docx) document
- Useful for offline study and sharing

### 5.9 Background Scheduler
- APScheduler runs daily to update revision queues
- Sends reminders (optional email/in-app notification)

---

## 6. Out of Scope (v1)

- Mobile app
- Social/community features
- Real-time collaboration
- Premium/paid tiers

---

## 7. Success Metrics

- User returns daily to complete revision queue
- Difficulty prediction accuracy improves over time per user
- Problems solved without hints increases over weeks (retention proof)

---

## 8. Constraints

- Python-only backend (FastAPI)
- Minimal frontend (Jinja2 templates or basic HTML — AI-assisted later)
- Free-tier deployable (Render / Railway)
- No storage of third-party platform passwords
