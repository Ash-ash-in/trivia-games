# TriviaGames

A browser-based trivia game platform. The first franchise is The Simpsons. The first game mode is a daily grid puzzle: a 2×2 grid where each cell requires a character who satisfies both a row category and a column category. Rarer answers score higher.

## Project structure

```
trivia-games/
│
├── data/           Raw data files (downloaded datasets, exports)
├── scraping/       Web scrapers and API connectors
├── database/       Schema definitions and migration files
├── api/            FastAPI backend (serves the game and quiz logic)
├── frontend/       HTML templates, CSS, and JavaScript for the game
└── docs/           Guides and reference documents for the team
```

## Team

- **Ash** — all technical work: data, backend, deployment
- **Tammy** — design, content, fact-checking, marketing

## Tech stack

| Layer | Tool |
|---|---|
| Data pipeline | Python + SQL |
| Database | PostgreSQL |
| Backend | Python (FastAPI) |
| Frontend | HTML/CSS (Jinja2 templates) + minimal JavaScript |
| Hosting | TBD (Railway or Render for MVP) |

## Getting started

See [`docs/setup.md`](docs/setup.md) for local environment setup instructions.

## Docs

- [`docs/project-plan.md`](docs/project-plan.md) — full project plan and stage breakdown
- [`docs/github-guide.html`](docs/github-guide.html) — GitHub introduction (what it is, how to use it)
- [`docs/github-commands.html`](docs/github-commands.html) — GitHub commands reference (commits, branches, fixing mistakes)
