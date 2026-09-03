# Local Environment Setup

This guide walks through setting up the project on your machine from scratch.

## Prerequisites

- Python 3.11 or later ([python.org](https://python.org))
- PostgreSQL 15 or later ([postgresql.org](https://postgresql.org))
- Git ([git-scm.com](https://git-scm.com))
- GitHub CLI — `winget install --id GitHub.cli`, then `gh auth login`

## 1. Clone the repository

```bash
git clone https://github.com/Ash-ash-in/trivia-games.git
cd trivia-games
```

## 2. Create and activate the virtual environment

```bash
python -m venv venv
.\venv\Scripts\Activate.ps1
```

Your terminal prompt will show `(venv)` when the environment is active. Run this activation command every time you open a new terminal for this project.

## 3. Install dependencies

```bash
pip install -r requirements.txt
```

## 4. Set up environment variables

Create a file called `.env` in the project root (it is gitignored — never commit this file):

```
DATABASE_URL=postgresql://postgres:yourpassword@localhost:5432/triviagames
```

Replace `yourpassword` with your local PostgreSQL password.

## 5. Create the local database

Open a PostgreSQL shell (pgAdmin or psql) and run:

```sql
CREATE DATABASE triviagames;
```

## 6. Verify everything works

With the venv active, run:

```bash
python -c "import fastapi, sqlalchemy, psycopg2; print('All good')"
```

You should see `All good`.

---

## Day-to-day workflow

1. Open a terminal in the project folder
2. Activate the venv: `.\venv\Scripts\Activate.ps1`
3. Work on whatever stage you're in
4. When done, stage and commit your changes

## Adding a new package

If you install a new package, update `requirements.txt` so others can replicate:

```bash
pip install some-package
pip freeze > requirements.txt
```

Then commit the updated `requirements.txt`.
