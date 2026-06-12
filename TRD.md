# Technical Requirements Document (TRD)
## Automated Coding Practice Tracker
**Author:** Vishakha Singh | **Version:** 1.0 | **Status:** Under Development

---

## 1. Tech Stack

| Layer | Technology |
|-------|------------|
| Backend Framework | FastAPI (Python 3.11+) |
| Database | PostgreSQL |
| ORM | SQLAlchemy (async) |
| Authentication | JWT (python-jose + passlib) |
| Spaced Repetition | Custom SM-2 implementation (Python) |
| AI - Difficulty Prediction | scikit-learn (Random Forest / Gradient Boosting) |
| AI - Hint Generation | LLM API (OpenAI / Gemini / Claude API) |
| Background Jobs | APScheduler |
| Document Export | python-docx |
| Frontend | Jinja2 templates (minimal, AI-polished later) |
| Deployment | Render / Railway (free tier) |
| Version Control | Git + GitHub |

---

## 2. Database Schema

### users
```
id            UUID PRIMARY KEY
username      VARCHAR UNIQUE NOT NULL
email         VARCHAR UNIQUE NOT NULL
password_hash VARCHAR NOT NULL
created_at    TIMESTAMP
```

### problems
```
id            UUID PRIMARY KEY
user_id       UUID FOREIGN KEY → users.id
title         VARCHAR NOT NULL
platform      VARCHAR (LeetCode, Codeforces, custom)
topic         VARCHAR (Arrays, DP, Graphs, Trees, etc.)
url           VARCHAR
difficulty_static  VARCHAR (Easy/Medium/Hard — platform's rating)
notes         TEXT
created_at    TIMESTAMP
```

### reviews
```
id               UUID PRIMARY KEY
user_id          UUID FOREIGN KEY → users.id
problem_id       UUID FOREIGN KEY → problems.id
reviewed_at      TIMESTAMP
self_rating      INTEGER (0–5, SM-2 scale)
easiness_factor  FLOAT (SM-2 EF, starts at 2.5)
interval_days    INTEGER (days until next review)
repetitions      INTEGER (number of successful reviews)
next_review_date DATE
```

### difficulty_predictions
```
id                UUID PRIMARY KEY
user_id           UUID FOREIGN KEY → users.id
problem_id        UUID FOREIGN KEY → problems.id
predicted_score   FLOAT (0.0 – 1.0, user-specific difficulty)
predicted_at      TIMESTAMP
model_version     VARCHAR
```

### hints
```
id           UUID PRIMARY KEY
user_id      UUID FOREIGN KEY → users.id
problem_id   UUID FOREIGN KEY → problems.id
hint_level   INTEGER (1, 2, or 3)
hint_text    TEXT
generated_at TIMESTAMP
```

---

## 3. SM-2 Algorithm Implementation

```python
def sm2_update(self_rating: int, easiness_factor: float, interval: int, repetitions: int):
    """
    self_rating: 0-5 (0=blackout, 5=perfect)
    Returns: (new_interval, new_ef, new_repetitions, next_review_date)
    """
    if self_rating < 3:
        # Failed recall — reset
        repetitions = 0
        interval = 1
    else:
        if repetitions == 0:
            interval = 1
        elif repetitions == 1:
            interval = 6
        else:
            interval = round(interval * easiness_factor)
        repetitions += 1

    # Update easiness factor
    ef = easiness_factor + (0.1 - (5 - self_rating) * (0.08 + (5 - self_rating) * 0.02))
    ef = max(1.3, ef)  # EF never drops below 1.3

    next_review = date.today() + timedelta(days=interval)
    return interval, ef, repetitions, next_review
```

---

## 4. AI Difficulty Prediction

### Approach
- **Model:** Gradient Boosting Classifier (scikit-learn)
- **Per-user model:** Train/fine-tune on each user's review history
- **Features:**
  - Topic category (encoded)
  - Platform difficulty (encoded)
  - User's average rating on same topic
  - User's average rating overall
  - Number of times user has reviewed this topic
  - Time since last review of topic
- **Output:** Predicted difficulty score (0.0 = trivial for user, 1.0 = very hard for user)
- **Cold start:** Use platform difficulty as default until user has ≥10 reviews

### Retraining
- Model retrained per user after every 5 new reviews (APScheduler background job)
- Model stored as serialized `.pkl` per user in `/models/{user_id}.pkl`

---

## 5. AI Hint Generation

### Approach
- LLM API call with structured prompt
- Hint levels:
  - Level 1: "Think about what data structure fits here"
  - Level 2: Conceptual approach, no code
  - Level 3: Pseudocode or key insight, still no full solution
- Prompt template:

```
Problem: {title}
Topic: {topic}
User's note: {notes}
Hint level requested: {level}

Generate a hint that guides the user without revealing the solution.
Hint level 1 = vague nudge, level 2 = approach, level 3 = near-solution pseudocode.
```

---

## 6. API Endpoints

### Auth
```
POST /auth/register       → Register new user
POST /auth/login          → Login, returns JWT token
```

### Problems
```
GET    /problems          → List all problems for user
POST   /problems          → Add new problem
GET    /problems/{id}     → Get problem detail
DELETE /problems/{id}     → Delete problem
POST   /problems/fetch    → Fetch from LeetCode (platform integration)
```

### Reviews
```
GET  /reviews/queue       → Today's revision queue for user
POST /reviews             → Submit review with self-rating
GET  /reviews/{problem_id}/history → Review history for problem
```

### AI
```
POST /ai/hint             → Get hint for problem (body: problem_id, level)
GET  /ai/difficulty/{problem_id} → Get predicted difficulty for user
```

### Analytics
```
GET /analytics/dashboard  → Stats summary
GET /analytics/topics     → Topic-wise breakdown
GET /analytics/retention  → Retention trend over time
```

### Export
```
GET /export/docx          → Download revision plan as .docx
```

---

## 7. Project Structure

```
coding-tracker/
├── app/
│   ├── main.py
│   ├── config.py
│   ├── database.py
│   ├── models/
│   │   ├── user.py
│   │   ├── problem.py
│   │   ├── review.py
│   │   └── hint.py
│   ├── schemas/
│   │   ├── user.py
│   │   ├── problem.py
│   │   └── review.py
│   ├── routers/
│   │   ├── auth.py
│   │   ├── problems.py
│   │   ├── reviews.py
│   │   ├── ai.py
│   │   ├── analytics.py
│   │   └── export.py
│   ├── services/
│   │   ├── sm2.py           # SM-2 algorithm
│   │   ├── difficulty_model.py  # AI difficulty prediction
│   │   ├── hint_service.py  # LLM hint generation
│   │   ├── scheduler.py     # APScheduler jobs
│   │   └── exporter.py      # python-docx export
│   ├── ml/
│   │   └── models/          # per-user .pkl files
│   └── templates/           # Jinja2 HTML templates
├── tests/
├── docs/
├── requirements.txt
├── .env
└── README.md
```

---

## 8. Security

- Passwords hashed with bcrypt (passlib)
- All endpoints protected with JWT bearer token
- User data isolated by `user_id` on every query
- No storage of third-party platform credentials
- `.env` for all secrets (never committed to Git)

---

## 9. Deployment

- **Platform:** Render (free tier) or Railway
- **Database:** Supabase (free PostgreSQL) or Render Postgres
- **Environment variables:** Set via platform dashboard
- **CI:** GitHub Actions (lint + test on push)
