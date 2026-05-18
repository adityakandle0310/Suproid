# 🎓 Suproid - Campus Placement Portal with AI Shortlisting

> A full-stack placement management platform inspired by Superset — built for college TPOs, students, and recruiters. Features an AI engine that automatically scores and ranks applicants against job descriptions.

**Live Demo:** `https://your-app.vercel.app` ← replace after deployment  
**Demo Video:** `https://youtube.com/your-demo` ← record after Week 5

---

## 👥 Team

| Name | Role | Responsibilities |
|------|------|-----------------|
| [Name 1] | Full-Stack Lead | Backend API, Database, Auth |
| [Name 2] | Frontend Dev | React dashboards, UI/UX |
| [Name 3] | AI/ML Dev | Scoring engine, Resume parser |
| [Name 4] | DevOps | Deployment, Seeding, README |

---

## ✨ Features

### Student
- Register/login and build a profile (branch, CGPA, skills)
- Upload resume PDF
- Browse open placement drives and apply with one click
- Track application status in real time (Applied → Shortlisted → Selected)
- Receive email + in-app notifications on status change

### TPO (Training & Placement Officer)
- Create and manage companies and job drives
- Set eligibility criteria (min CGPA, max backlogs, required skills)
- View analytics dashboard — placement rate, branch-wise stats, drive funnel
- Schedule interview slots and send automated notifications

### Recruiter
- Post job descriptions with required skill tags
- View all applicants ranked automatically by AI score
- See skill match breakdown per candidate
- Shortlist candidates and schedule interviews

---

## 🤖 How AI Shortlisting Works

When a student applies to a drive, the system automatically sends their resume data and the job description to the AI microservice, which returns a score from 0–100.

**Scoring formula (weighted):**

| Component | Weight | How it's calculated |
|-----------|--------|-------------------|
| Skill match | 50% | `matched_skills / required_skills` |
| CGPA score | 25% | `student_cgpa / 10.0` (normalised) |
| Semantic similarity | 25% | Cosine similarity of resume + JD embeddings via `sentence-transformers` |

All applicants for a drive are then re-ranked by total score. Recruiters see the ranked list with a colour-coded score bar (green > 70, amber 40–70, red < 40) and a breakdown of which skills matched.

---

## 🛠 Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React 18 + Vite, React Router, React Hook Form, Recharts |
| Backend | Node.js, Express.js, Prisma ORM |
| Database | PostgreSQL (hosted on Railway) |
| AI Service | Python, Flask, pdfplumber, spaCy, sentence-transformers |
| Auth | JWT (JSON Web Tokens), bcryptjs |
| File Storage | Cloudinary (resume PDFs) |
| Email | Nodemailer + Gmail SMTP |
| Deployment | Vercel (frontend), Render (backend + AI), Railway (DB) |

---

## 🗄 Database Schema (8 tables)

```
USERS ──────────── STUDENT_PROFILES
  │                      │
  │               APPLICATIONS ──── SHORTLISTS
  │                    │
COMPANIES ──── JOB_DRIVES ──── INTERVIEW_SLOTS
  
USERS ──── NOTIFICATIONS
```

**Key design decisions:**
- `parsedResume` stored as JSON in `StudentProfile` — avoids re-parsing on every score request
- `aiScore` + `aiRank` stored on `Application` — shortlist sorting is a simple `ORDER BY aiRank`
- `@@unique([studentId, driveId])` on Application — prevents duplicate applications at DB level

---

## 📁 Folder Structure

```
placement-portal/
├── client/                  ← React + Vite frontend
│   └── src/
│       ├── pages/           ← Login, StudentDashboard, TPODashboard, RecruiterDashboard
│       ├── components/      ← DriveCard, ApplicationList, ProfileForm, ScoreBar
│       ├── api/             ← axios instance with JWT interceptor
│       └── context/         ← AuthContext
├── server/                  ← Node.js + Express API
│   ├── routes/              ← auth.js, students.js, drives.js, applications.js
│   ├── middleware/          ← auth.js (JWT + role guard)
│   └── utils/              ← triggerAIScore.js, notify.js
├── ai/                      ← Python Flask microservice
│   ├── app.py               ← /score endpoint
│   ├── parser.py            ← PDF text extraction + skill/CGPA extraction
│   └── scorer.py            ← weighted scoring formula
├── prisma/
│   ├── schema.prisma        ← all 8 table definitions
│   └── seed.js              ← 30 students, 5 companies, 8 drives
└── README.md
```

---

## 🚀 Local Setup

### Prerequisites
- Node.js v18+
- Python 3.10+
- PostgreSQL (or use Railway cloud DB)

### 1. Clone the repo
```bash
git clone https://github.com/your-username/placement-portal.git
cd placement-portal
```

### 2. Backend setup
```bash
cd server
npm install
cp ../.env.example .env
# Fill in DATABASE_URL, JWT_SECRET, CLOUDINARY_*, EMAIL credentials
npx prisma migrate dev --name init
npx prisma generate
node ../prisma/seed.js   # seed dummy data
npm run dev              # runs on http://localhost:5000
```

### 3. Frontend setup
```bash
cd ../client
npm install
# Set VITE_API_URL=http://localhost:5000/api in .env.local
npm run dev              # runs on http://localhost:5173
```

### 4. AI service setup
```bash
cd ../ai
python -m venv venv
source venv/bin/activate     # Windows: venv\Scripts\activate
pip install -r requirements.txt
python -m spacy download en_core_web_sm
python app.py               # runs on http://localhost:8000
```

### 5. Test accounts (after seeding)
| Role | Email | Password |
|------|-------|----------|
| Student | student1@college.edu | password123 |
| TPO | tpo@college.edu | password123 |
| Recruiter | recruiter@tcs.com | password123 |

---

## 📅 5-Week Implementation Plan

| Week | Focus | Key Deliverables |
|------|-------|-----------------|
| 1 | Project setup + Auth | Monorepo, DB schema, JWT auth, login UI |
| 2 | Profiles + Job drives | Resume upload, apply flow, TPO drive creation |
| 3 | AI engine | Resume parser, scoring formula, Flask microservice |
| 4 | Recruiter dashboard + analytics | Ranked shortlist UI, charts, notifications, interview slots |
| 5 | Polish + deployment | Seed data, Vercel/Render deploy, README, demo video |

---

## 🔌 API Endpoints

| Method | Route | Role | Description |
|--------|-------|------|-------------|
| POST | `/api/auth/register` | Public | Register new user |
| POST | `/api/auth/login` | Public | Login, returns JWT |
| GET | `/api/drives` | Student | Get all open drives |
| POST | `/api/drives` | TPO | Create a new drive |
| POST | `/api/drives/:id/apply` | Student | Apply + trigger AI scoring |
| GET | `/api/applications/drive/:id` | Recruiter/TPO | Get ranked applicants |
| PATCH | `/api/applications/:id/shortlist` | Recruiter | Shortlist a candidate |
| GET | `/api/analytics/summary` | TPO | Placement statistics |
| POST | `/score` (port 8000) | Internal | AI scoring microservice |

---

## 🧠 Interview Talking Points

These are questions you'll likely be asked — prepare answers for them:

1. **"Why PostgreSQL and not MongoDB?"** — Applications have strict relational integrity (a student can apply to a drive exactly once, shortlists reference applications). Relational DB enforces this at the DB level; MongoDB leaves it to the application code.

2. **"Why a separate Python microservice for AI?"** — Python has better ML libraries (spaCy, sentence-transformers, scikit-learn). Building it as a microservice means the Node backend just calls an HTTP endpoint — clean separation of concerns and the AI service can be scaled independently.

3. **"How does your AI scoring work?"** — Walk through the three components: keyword skill match (50%), CGPA normalisation (25%), and semantic cosine similarity using sentence-transformers (25%). Mention the model `all-MiniLM-L6-v2` runs in ~50ms on CPU.

4. **"What if the AI score is biased?"** — The recruiter still makes the final shortlist decision. The AI score is a ranking aid, not a binary decision. Recruiters can override by filtering/sorting manually.

5. **"How did you handle duplicate applications?"** — Prisma `@@unique([studentId, driveId])` constraint at the database level — the DB rejects duplicates before any application code runs.

---

## 📌 Contributing (for team members)

1. Branch naming: `feature/your-name/what-you-built` (e.g. `feature/rahul/ai-scorer`)
2. Commit messages: `feat: add resume parser`, `fix: JWT expiry`, `docs: update README`
3. Open a PR to `main` — get at least one review before merging
4. Never commit `.env` files — use `.env.example` with placeholder values

---

## 📄 License

MIT — free to use, modify, and showcase in your portfolio.
