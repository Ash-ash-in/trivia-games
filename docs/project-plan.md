# TriviaGames — High-Level Project Plan

## Context

Two-person team (Ash: Python/SQL/data; Tammy: design/content) building a browser-based trivia game platform starting with The Simpsons. The first game mode is a grid-style daily puzzle (like MovieGrid.io): a 2×2 grid where each cell requires a character who satisfies the row category AND the column category. Rarer answers score higher. The platform must be accurate — a wrong answer is a customer-ending event. Launch target is 3–6 months. Fully anonymous play (no user accounts). Global English audience. Revenue via Google AdSense.

---

## Team Responsibilities

### Ash (all technical work)
- All code: scrapers, database, API, quiz engine, backend, deployment
- GitHub setup and version control
- Data pipeline design and maintenance
- Writing user guides and code walkthroughs
- Reviewing and merging any AI-assisted code

### Tammy (design, content, and sign-off)
- Visual design briefs in plain English (implemented by Ash / AI)
- Category ideation: proposing grid category combinations
- Human fact-checking of flagged uncertain data
- Sign-off testing at the end of each stage (using the live site in a browser)
- Marketing strategy and copy
- Google AdSense account creation and management

---

## Language Decisions

| Layer | Language | Rationale |
|---|---|---|
| Data pipeline | Python + SQL | Ash's existing skill set |
| Database | PostgreSQL | Industry standard; excellent Python support |
| Backend / API | Python (FastAPI) | Modern, fast, easy to learn incrementally |
| Frontend templates | HTML + CSS (Jinja2) | Server-rendered by FastAPI; not a programming language |
| Frontend interactivity | JavaScript (minimal, vanilla) | **Unavoidable for a browser game.** The sole explicit exception to the Python/SQL rule. Kept to the smallest footprint possible. |
| Hosting config | YAML / shell | Standard infrastructure files |

> **Decision needed from Ash before Stage 9:** Approve JavaScript as the minimal frontend exception. No other languages will be added without explicit approval.

---

## Development Stages (in order)

---

### Stage 1 — Project Foundation
**Goal:** Create the skeleton every future stage will build on.

**Tasks:**
- Set up GitHub repository with a clear folder structure (`/data`, `/scraping`, `/database`, `/api`, `/frontend`, `/docs`)
- Write a `README.md` explaining the project structure
- Set up a Python virtual environment and `requirements.txt`
- Create a local PostgreSQL database for development
- Write a short user guide: *"How to set up your local environment"*

**Sign-off test (Ash):** Clone the repo from scratch on a fresh machine and follow the setup guide to confirm it works end-to-end.

---

### Stage 2 — Database Design
**Goal:** Design a schema that can hold all franchise character/episode data in a structured, cross-franchise-compatible way.

**Key tables (Simpsons pilot):**
- `characters` — id, name, franchise, gender, race, age_category, deceased, first_appearance_episode_id, etc.
- `episodes` — id, franchise, season, episode_number, title, air_date, imdb_id
- `character_appearances` — character_id, episode_id (many-to-many)
- `character_relationships` — character_id, related_character_id, relationship_type
- `locations` — id, name, franchise
- `character_locations` — character_id, location_id
- `facts` — id, character_id, field_name, value, source, confidence_score, reviewed, flagged
- `tickets` — id, fact_id, reason, status, resolution, resolved_by, created_at
- `quiz_categories` — id, franchise, label, field_name, sql_query
- `quiz_grids` — id, franchise, date, row_categories, col_categories
- `quiz_answers` — id, grid_id, cell_position, character_id, submitted_count, obscurity_score

**Notes:**
- The `facts` table is the accuracy backbone: every piece of data has a source and a confidence score
- The schema is designed to be franchise-agnostic from day one
- SQL migrations managed with Alembic

**Sign-off test (Ash):** Populate a handful of rows manually and run queries that simulate quiz logic — confirm the schema supports the required lookups.

---

### Stage 3 — Data Collection (Scrapers + APIs)
**Goal:** Populate the database with accurate Simpsons character and episode data from multiple sources.

**Sources to integrate:**
1. **Simpsons Fandom Wiki** (simpsonswiki.com) — scrape with `requests` + `BeautifulSoup`. Rich character infoboxes, appearance lists, relationships.
2. **Wikipedia** — scrape episode lists, character summaries. Good as a cross-reference.
3. **TMDB API** — episode metadata (air dates, descriptions, cast). Free tier is sufficient.
4. **IMDB** — ratings and cast via `cinemagoer` (IMDbPY) library or scraping.
5. **Kaggle / existing datasets** — use as a starting baseline to verify against, not as ground truth.

**Pipeline design:**
- Each source has its own Python module in `/scraping/sources/`
- A normalisation layer maps each source's fields to the shared database schema
- Each fact written to the `facts` table includes: `source`, `raw_value`, `normalised_value`, `confidence_score`
- Scrapers are run on-demand (not continuous), with rate limiting and polite crawl delays built in

**Sign-off test (Ash + Tammy):** Run scrapers for 20 well-known characters. Tammy manually checks 5 of them against her own knowledge of the show. Zero errors required.

---

### Stage 4 — Confidence Scoring & Cross-Referencing
**Goal:** Automatically detect conflicting data across sources and generate a confidence score for every fact.

**Logic:**
- If 3 sources agree on a value → high confidence (e.g. 0.95)
- If 2 sources agree, 1 disagrees → medium confidence (e.g. 0.7), flagged for review
- If only 1 source → low confidence (e.g. 0.4), flagged
- If sources actively contradict → confidence 0.0, ticket raised automatically

**Implementation:**
- Python script `confidence.py` runs after each scrape batch
- Updates `facts.confidence_score` and sets `facts.flagged = True` where needed
- Confidence thresholds are configurable constants at the top of the file
- Only facts above a configurable threshold (e.g. 0.8) are used in quiz generation

**Sign-off test (Ash):** Introduce a deliberate conflict in the test data; confirm the script flags it and creates a ticket automatically.

---

### Stage 5 — Ticket / Review System
**Goal:** A simple tool for Ash and Tammy to investigate flagged uncertain facts and either confirm, correct, or remove them.

**Implementation:**
- A Python CLI script (`tickets.py`) that lists open tickets, shows the conflicting sources, and lets the user resolve them
- Alternatively: a minimal admin web page (added once the backend exists in Stage 7)
- Resolved tickets update `facts.reviewed = True` and lock the value

**Tammy's role:** Tammy reviews tickets about content she knows well (character personalities, relationships, major plot points). Ash handles structural/technical tickets.

**Sign-off test (both):** Work through 10 real flagged tickets together. Confirm the tool is usable by Tammy without coding knowledge.

---

### Stage 6 — Grid Feasibility Analysis
**Goal:** Determine how many valid category combinations exist and how many answers fall in each cell, so we can confirm 2×2 is viable and understand the quiz space.

**Analysis steps (Python + SQL):**
1. List all candidate categories (e.g. "Female characters", "Characters who work at the nuclear plant", "Characters deceased before Season 10")
2. For each pair of categories, count how many characters satisfy both
3. Flag cells with fewer than N valid answers as too restrictive for a daily puzzle
4. Output a heatmap-style report showing category pair viability

**Output:** A ranked list of viable category pairs, plus a recommended minimum/maximum answer count per cell.

**This analysis directly informs Stage 8 (quiz generation).**

**Sign-off test (Ash + Tammy):** Review the heatmap together. Tammy assesses whether the categories feel interesting to players. Agree on minimum answer threshold before proceeding.

---

### Stage 7 — Backend API (FastAPI)
**Goal:** Build the Python server that powers the game and serves data to the frontend.

**Key endpoints:**
- `GET /quiz/today` — returns today's grid (categories + valid answer lists, obscurity scores)
- `POST /quiz/answer` — accepts a character name for a cell; returns whether it's valid and the obscurity score
- `GET /quiz/history` — returns past grids
- `POST /feedback` — accepts a user-submitted correction or comment
- `GET /admin/tickets` — lists open fact tickets (protected, localhost-only initially)

**Implementation:**
- FastAPI with Pydantic models for request/response validation
- SQLAlchemy ORM for database access
- A single `main.py` entry point; routers split by feature (`/routers/quiz.py`, `/routers/feedback.py`, etc.)
- Auto-generated API documentation at `/docs` (FastAPI provides this for free)

**Sign-off test (Ash):** Use the auto-generated `/docs` UI to test every endpoint manually. Confirm answers validate correctly for 5 characters per cell.

---

### Stage 8 — Quiz Generation Algorithm
**Goal:** Automatically generate a new daily grid puzzle and store it in the database.

**Algorithm:**
1. Select 2 row categories and 2 column categories from viable pairs (Stage 6 analysis)
2. Verify the resulting 4 cells each have at least N valid answers
3. Avoid repeating the same category combination within the last 30 days
4. Write the grid to `quiz_grids` with the date
5. A scheduled Python script (`generate_quiz.py`) runs daily (e.g. via Windows Task Scheduler or a cron job on the server)

**Obscurity scoring:**
- When a player submits an answer, `quiz_answers.submitted_count` is incremented
- Obscurity score = inverse of submission frequency (rarer answers score higher)
- Scores update in real time as answers come in

**Sign-off test (Ash + Tammy):** Run the generator for 7 days of puzzles. Tammy plays all 7 and assesses whether the categories are fair, interesting, and varied.

---

### Stage 9 — Frontend (Game Interface)
**Goal:** Build the browser-based game that players interact with.

**Stack:** FastAPI serving Jinja2 HTML templates. Vanilla JavaScript for game interactivity only (cell selection, answer submission, score display). CSS for layout and styling.

**Tammy's role:** Tammy provides a written design brief (colours, fonts, mood) and draws rough sketches if helpful. Ash (with AI assistance) implements the design in HTML/CSS.

**Pages:**
- `/` — today's grid puzzle (the main game)
- `/how-to-play` — rules and explanation
- `/archive` — past puzzles (view only)
- `/about` — project info, franchise list

**Google AdSense:** Ad units placed in the sidebar and below the grid. AdSense code injected into the base HTML template.

**Sign-off test (Tammy):** Tammy plays the full game on desktop and mobile. Signs off on design and usability. Zero layout issues on common screen sizes.

---

### Stage 10 — User Feedback System
**Goal:** Let anonymous players report errors or submit corrections without needing an account.

**Implementation:**
- A small form on the game page: "Report an issue with this answer"
- Submitted to `POST /feedback`; stored in a `feedback` table
- Ash reviews new feedback weekly; genuine errors create a ticket in Stage 5's system
- Feedback includes: grid_id, cell_position, character_id submitted, issue description (free text)

**Sign-off test (Ash):** Submit 5 test feedback entries through the UI. Confirm they appear in the database with correct metadata.

---

### Stage 11 — Deployment
**Goal:** Get the site live on the internet.

**Recommended hosting (MVP):** Railway (railway.app) or Render (render.com)
- Both support Python/FastAPI natively
- Free tier sufficient for early traffic
- One-click PostgreSQL add-on
- Can migrate to AWS/GCP later if traffic demands it

**Steps:**
- Set up a production PostgreSQL database
- Configure environment variables (DB credentials, API keys) — never committed to GitHub
- Connect GitHub repo for automatic deploys on push to `main`
- Run all scrapers against the production database
- Set up daily quiz generation as a scheduled job

**Sign-off test (Ash + Tammy):** Play the live site from different devices and locations. Confirm ads load, feedback form works, and the quiz generates correctly overnight.

---

### Stage 12 — User Guides and Code Walkthroughs
**Goal:** Ensure Ash understands every part of the system and both team members can operate it independently.

**Documents (in `/docs`):**
- `setup.md` — local environment setup from scratch
- `scraping.md` — how to run scrapers, add a new source
- `confidence.md` — how the scoring system works
- `tickets.md` — how to review and resolve tickets
- `quiz-gen.md` — how the daily puzzle is generated
- `deployment.md` — how to deploy and update the live site
- `adding-a-franchise.md` — checklist for onboarding a new franchise

**Walkthroughs:** After each stage, a session where the AI walks Ash through the code: what each file does, how modules connect, and why the key decisions were made.

---

### Stage 13 — Marketing
**Goal:** Drive traffic to the site at launch and sustain it.

**Tammy leads; Ash supports with technical setup.**

**Channels:**
- Social media: Reddit (r/TheSimpsons, r/trivia, r/webgames), Twitter/X, Instagram
- SEO: Descriptive page titles, meta tags, sitemap.xml (Ash implements; Tammy writes copy)
- Launch post on relevant communities when the site goes live
- Consider a "streak" mechanic or shareable result card (like Wordle) to drive organic sharing
- Google AdSense account created and verified before launch (Tammy owns this)

---

## Key Principles Throughout

1. **Accuracy first.** No fact enters the quiz unless it has a confidence score above the agreed threshold and has been reviewed.
2. **Test at every stage.** Each stage has a defined human sign-off test. Nothing progresses until it passes.
3. **Ash learns as we build.** Every stage includes a walkthrough session. Code is organised so modules are traceable.
4. **Tammy can always test.** The live site is always accessible to Tammy for review. Nothing is hidden behind a command line.
5. **One language exception.** JavaScript is the only addition to the Python/SQL stack. Any further language additions require an explicit decision.
6. **No surprises in GitHub.** All code changes go through branches and are described in plain-English commit messages.
