# Keshav16
# Beyond the Number

**A diagnostic assessment platform that treats a mark as a starting point, not the answer.**
By Eklavy & Keshav.

## The two problems it answers

**1. "The Same Number"** - two learners can score the same mark: one understood, the other guessed.
Every answer is submitted with a 1-tap confidence tag (Sure / Unsure / Guessing), so a *correct + Sure* answer
(`secure`) is told apart from a *correct + Guessing* answer (`fragile_correct`).

**2. "One Error, Several Stories"** - the same wrong answer can come from a mistaken rule, a missed condition,
or running out of time. We treat a mistake as a **clue, not a verdict**:

| Evidence                                             | Story              | Profile state        |
|------------------------------------------------------|--------------------|----------------------|
| wrong + Sure, reworded **probe also wrong**          | `mistaken_rule`    | Deep Misconception   |
| wrong + Sure, reworded **probe right**               | `missed_condition` | Careless Slip        |
| wrong + not Sure + answered far faster than expected | `rushed`           | Careless Slip        |
| wrong + not Sure, normal pace                        | `knowledge_gap`    | Lucky / Fragile      |
| wrong + Sure, probe **not answered yet**             | `unresolved`       | *"evidence needed"*  |

While a story is `unresolved`, the profile says exactly what evidence is still missing instead of guessing.

## Quick start

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
uvicorn app.main:app --reload
```

Open http://127.0.0.1:8000, click **Seed Data & Student Cohort**, and explore.
API docs: http://127.0.0.1:8000/docs

Run the tests:

```bash
pip install -r requirements-dev.txt
pytest -q
```

## Project structure

```
beyond-the-number/
├── app/
│   ├── main.py            # FastAPI routes
│   ├── database.py        # engine/session (DATABASE_URL switches SQLite -> PostgreSQL)
│   ├── models.py          # database schema (SQLAlchemy)
│   ├── schemas.py         # request validation (Pydantic)
│   ├── engine.py          # diagnostic logic: evidence -> story -> profile
│   ├── question_bank.py   # upload / upsert questions
│   └── seed.py            # demo question bank + 10 simulated students
├── data/
│   ├── question_bank.json     # items + probes with misconception codes
│   └── misconceptions.json    # description + remediation per misconception code
├── static/index.html      # Tailwind + Alpine.js + Chart.js frontend
├── tests/
├── .github/workflows/ci.yml
└── requirements.txt
```

## Database schema

```mermaid
erDiagram
    SKILLS ||--o{ ITEMS : has
    SKILLS ||--o{ MISCONCEPTIONS : has
    ITEMS ||--o{ OPTIONS : has
    ITEMS ||--o| ITEMS : "probe of"
    MISCONCEPTIONS ||--o{ OPTIONS : "tagged on"
    CLASSROOMS ||--o{ LEARNERS : contains
    LEARNERS ||--o{ RESPONSES : submits
    OPTIONS ||--o{ RESPONSES : chosen
    RESPONSES ||--o| RESPONSES : "probe of"
    LEARNERS ||--o{ PROFILES : has
    SKILLS ||--o{ PROFILES : "for"
    MISCONCEPTIONS ||--o{ PROFILES : dominant
```

| Table | Purpose |
|-------|---------|
| `skills` | Topics being assessed |
| `misconceptions` | Code + description + suggested remediation |
| `items` | Questions; probe items point to their parent via `parent_item_id` |
| `options` | One row per answer choice; flags the correct one and the misconception it reveals |
| `classrooms`, `learners` | Who is answering |
| `responses` | Raw evidence: option, confidence, time, plus the derived `story` |
| `profiles` | Latest cognitive state per learner + skill, with story counts and `evidence_needed` |

## Uploading your own questions

Use the **Upload Questions** tab (or `POST /api/items/bulk`). Format:
see `data/question_bank.json`. Rules: exactly one correct option per item, unique option ids,
probe items must set `parent_item_id`, and `expected_time_s` (default 30) is used to detect rushed answers.

## Deploying with PostgreSQL

```bash
pip install psycopg2-binary
export DATABASE_URL=postgresql+psycopg2://user:password@host:5432/beyond_the_number
```

## Putting this on GitHub

```bash
git init
git add .
git commit -m "Beyond the Number v3"
git branch -M main
git remote add origin https://github.com/<your-username>/beyond-the-number.git
git push -u origin main
```
