# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Spendly is a Flask personal expense tracker built as a **step-based student curriculum project**. Large parts of the app are intentionally unimplemented scaffolding — routes exist and return placeholder strings like `"Add expense — coming in Step 7"`, and `database/db.py` is a docstring-only stub ("Students will write this file in Step 1"). When asked to implement a feature, check whether it's one of these placeholder steps before assuming it needs to be built from scratch.

## Commands

Run all commands from the repo root (`f:\Claude Project\expense-tracker`).

```bash
# Run the dev server (must set FLASK_DEBUG=1 or template edits won't be picked up —
# Flask/Jinja caches templates in memory otherwise)
FLASK_APP=app.py FLASK_DEBUG=1 ./venv/Scripts/python.exe -m flask run --port 5050

# Alternative: run directly (uses port 5001, debug already on via app.run(debug=True))
./venv/Scripts/python.exe app.py

# Run tests (pytest + pytest-flask are in requirements.txt; no test files exist yet)
./venv/Scripts/python.exe -m pytest

# Install/refresh dependencies into the existing venv
./venv/Scripts/python.exe -m pip install -r requirements.txt
```

There is no build step, linter, or bundler configured — plain Flask + Jinja + hand-written CSS/JS, no npm project.

## Architecture

- **`app.py`** — single-file Flask app, all routes defined here. Two tiers of routes:
  - Fully implemented: `/` (landing), `/register`, `/login`, `/terms`, `/privacy` — each just does `render_template(...)`.
  - Placeholder-only (return a plain string, no template, no logic): `/logout`, `/profile`, `/expenses/add`, `/expenses/<int:id>/edit`, `/expenses/<int:id>/delete`. The `register.html`/`login.html` forms POST to `/register`/`/login` but there are no POST handlers for those routes yet — that's expected, not a bug to silently "fix" without being asked.
- **`database/db.py`** — currently an empty stub with a comment describing the intended API (`get_db()`, `init_db()`, `seed_db()` using sqlite3). No DB is wired up yet; `database/__init__.py` is empty.
- **Templates (`templates/`)** — Jinja2 inheritance from `base.html`, which owns the `<head>`, navbar, and footer and exposes `{% block title %}`, `{% block head %}`, `{% block content %}`, `{% block scripts %}` for children to fill. All internal links use `url_for('<endpoint>')`, never hardcoded paths.
- **`static/css/style.css`** — single stylesheet, no preprocessor. Theme is driven entirely by CSS custom properties defined at the top of the file under `:root` (`--ink*`, `--paper*`, `--accent`, `--accent-2`, `--danger`, `--border*`, `--font-display`/`--font-body`, `--radius-*`). New UI should reuse these variables rather than hardcoding colors/fonts, to stay visually consistent with the rest of the site.
- **`static/js/main.js`** — currently empty; site-wide JS goes here. Page-specific JS (e.g. the demo-video modal on the landing page) lives inline in that template's `{% block scripts %}` instead.

## Notes specific to this repo

- No test files exist yet despite pytest/pytest-flask being installed — don't assume test coverage exists for a feature just because the dependency is present.
- `__MACOSX/` at the repo root is stray junk from a zip extraction, not part of the app.
- The venv lives at `./venv` (Windows-style `venv/Scripts/python.exe`, not `venv/bin/python`).
