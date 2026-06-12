# App Flow Document
## Automated Coding Practice Tracker
**Author:** Vishakha Singh | **Version:** 1.0 | **Status:** Under Development

---

## 1. User Journey Overview

```
Landing Page
    │
    ├── Register → Onboarding → Dashboard
    │
    └── Login ──────────────→ Dashboard
                                  │
                    ┌─────────────┼──────────────┐
                    │             │              │
              Add Problem    Revision Queue   Analytics
                    │             │              │
              Problem Detail  Review Screen  Topic Breakdown
                    │
              Request Hint / View AI Difficulty
```

---

## 2. Screen-by-Screen Flow

---

### 2.1 Landing Page
**URL:** `/`

**Content:**
- App name + tagline
- "Login" and "Register" buttons

**Actions:**
- → Register Page
- → Login Page

---

### 2.2 Register Page
**URL:** `/auth/register`

**Fields:** Username, Email, Password, Confirm Password

**Flow:**
1. User fills form → submits
2. POST `/auth/register`
3. On success → redirect to Login
4. On error → show inline validation message

---

### 2.3 Login Page
**URL:** `/auth/login`

**Fields:** Email, Password

**Flow:**
1. User submits credentials
2. POST `/auth/login` → returns JWT token
3. Token stored in session/cookie
4. Redirect → Dashboard

---

### 2.4 Dashboard (Home)
**URL:** `/dashboard`

**Sections:**
- **Today's Queue** — problems due for revision today (from SM-2 engine)
- **Quick Stats** — streak, total solved, weak topics
- **Add Problem** button
- **Navigation** — Problems | Queue | Analytics | Export

**Flow:**
- Click problem in queue → Review Screen
- Click "Add Problem" → Add Problem Page
- Click topic in weak areas → Topic Detail

---

### 2.5 Add Problem Page
**URL:** `/problems/add`

**Fields:**
- Title (required)
- Platform (LeetCode / Codeforces / Custom)
- Topic (dropdown: Arrays, DP, Graphs, Trees, Strings, etc.)
- Static Difficulty (Easy / Medium / Hard)
- Problem URL (optional)
- Notes (optional)

**OR:**
- Paste LeetCode profile URL → auto-fetch solved problems

**Flow:**
1. User fills form → submits
2. POST `/problems`
3. SM-2 initialized: EF=2.5, interval=1, repetitions=0
4. AI difficulty prediction triggered in background
5. Redirect → Problem Detail Page

---

### 2.6 Problem Detail Page
**URL:** `/problems/{id}`

**Shows:**
- Problem title, topic, platform, URL
- Static difficulty + **AI-predicted difficulty for this user**
- Review history (dates + ratings)
- Notes
- Next review date

**Actions:**
- "Get Hint" → opens Hint Panel
- "Mark Reviewed" → Review Screen
- "Edit" / "Delete"

---

### 2.7 Hint Panel (within Problem Detail)

**Flow:**
1. User clicks "Get Hint"
2. Chooses hint level (1 / 2 / 3)
3. POST `/ai/hint` with problem_id + level
4. LLM generates hint → displayed inline
5. Hint saved to DB (so same hint shown if requested again)

**Hint Levels shown to user:**
- 🟡 Level 1 — Nudge (vague direction)
- 🟠 Level 2 — Approach (conceptual)
- 🔴 Level 3 — Near-solution (pseudocode)

---

### 2.8 Review Screen
**URL:** `/reviews/{problem_id}`

**Shows:**
- Problem title + topic
- Link to problem URL
- "How well did you recall this?" rating buttons

**Rating Scale (SM-2):**
| Rating | Label |
|--------|-------|
| 0 | Complete blackout |
| 1 | Wrong but familiar |
| 2 | Wrong, easy to recall |
| 3 | Correct with effort |
| 4 | Correct with hesitation |
| 5 | Perfect recall |

**Flow:**
1. User selects rating → submits
2. POST `/reviews`
3. SM-2 calculates new interval + next review date
4. AI difficulty model retraining triggered (background, if ≥5 new reviews)
5. Show result: "Next review in X days" → redirect to Dashboard

---

### 2.9 Revision Queue Page
**URL:** `/queue`

**Shows:**
- All problems due today (sorted by topic)
- Problems due in next 3 days (preview)
- AI-predicted difficulty badge on each problem

**Actions:**
- Click problem → Review Screen

---

### 2.10 Analytics Page
**URL:** `/analytics`

**Sections:**
- **Overall Stats** — total problems, avg rating, current streak
- **Topic Breakdown** — bar chart of avg rating per topic
- **Retention Trend** — line chart of ratings over time
- **Weak Topics** — topics where avg rating < 3
- **Predicted Hard Problems** — AI-flagged problems user will likely struggle with

---

### 2.11 Export Page
**URL:** `/export`

**Options:**
- Export today's revision queue as .docx
- Export full problem list with notes as .docx
- Export topic-wise performance report as .docx

**Flow:**
1. User clicks export option
2. GET `/export/docx?type={queue|problems|report}`
3. python-docx generates file
4. File downloaded to user's browser

---

## 3. Background Flows (Non-UI)

### Daily Scheduler (APScheduler)
- Runs at midnight
- Scans all reviews → finds problems where `next_review_date = today`
- Updates each user's daily queue
- Optional: sends email reminder

### AI Model Retraining
- Triggered after user submits a review (if review count % 5 == 0)
- Runs in background thread
- Re-trains user's personal difficulty prediction model
- Saves updated `.pkl` file

---

## 4. Navigation Map

```
/                    → Landing
/auth/register       → Register
/auth/login          → Login
/dashboard           → Home (requires auth)
/problems            → All Problems list
/problems/add        → Add Problem form
/problems/{id}       → Problem Detail + Hints
/reviews/{id}        → Review Screen
/queue               → Today's Revision Queue
/analytics           → Analytics & Stats
/export              → Export Options
```

---

## 5. Error States

| Scenario | Handling |
|----------|----------|
| Invalid login | "Invalid credentials" message on login page |
| JWT expired | Redirect to login with "Session expired" message |
| No problems added yet | Dashboard shows empty state with "Add your first problem" CTA |
| AI hint API failure | Show "Hint unavailable, try again" — fallback gracefully |
| LeetCode fetch fails | Show error + allow manual entry |
| Empty revision queue | Show "You're all caught up! 🎉" message |
