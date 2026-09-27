# Spec: Login and Logout

## Overview
Implements real authentication for Spendly: `POST /login` verifies a user's email and password against the `users` table and starts a session, and `GET /logout` ends it. This is the first step that introduces server-side session state (`flask.session`) — registration (Step 2) deliberately avoided sessions, but real login requires them so subsequent requests can know who's signed in. This unblocks Step 4 (Profile), which needs to know the current user.

## Depends on
- Step 1 — Database Setup (`.claude/specs/01-database-setup.md`): `get_db()` and the `users` table.
- Step 2 — Registration (`.claude/specs/02-registration.md`): users must be able to exist (via registration, or the seeded demo user `demo@spendly.com` / `demo123`) before they can log in.

## Routes
- `POST /login` — validates email + password, starts a session on success — public
- `GET /logout` — clears the session — logged-in (safe to call when already logged out; just redirects)
- `GET /login` already exists and is unchanged.

## Database changes
No database changes. The existing `users` table (`email`, `password_hash`) already has everything login needs to verify credentials.

## Templates
- **Modify:** `templates/login.html` — repopulate the `email` input on a failed login attempt (mirrors the pattern already used in `register.html`), same as-is `{% if error %}` block.
- **Modify:** `templates/base.html` — navbar conditionally shows signed-in state (user's name + a "Sign out" link to `/logout`) instead of "Sign in" / "Get started", using Flask's automatically-injected `session` object directly in Jinja (`{% if session.user_id %}`) — no new context processor needed.

## Files to change
- `app.py`:
  - Set `app.secret_key` (required for `flask.session` to work) — a dev-only constant is fine, no secrets manager exists in this project.
  - Add `POST` handling to `/login`: read `email`/`password` from `request.form`, look up the user via `get_db()`, verify the password with `werkzeug.security.check_password_hash` against `password_hash`. On success: set `session["user_id"]` and `session["user_name"]`, redirect to `/`. On failure (no such email, or wrong password): re-render `login.html` with a single generic error message and the submitted email repopulated.
  - Implement `GET /logout`: `session.clear()`, redirect to `/`.
- `templates/login.html` — repopulate `email` on error.
- `templates/base.html` — navbar logged-in/out conditional.

## Files to create
None.

## New dependencies
No new dependencies. `werkzeug.security.check_password_hash` is already available (its counterpart `generate_password_hash` is already used in `database/db.py` and `app.py`). `flask.session` is part of Flask itself.

## Rules for implementation
- No SQLAlchemy or ORMs
- Parameterised queries only
- Passwords hashed with werkzeug (verified with `check_password_hash`, never compared as plaintext)
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`
- The login error must be a single generic message (e.g. "Invalid email or password.") for both "no such user" and "wrong password" cases — do not reveal which one it was, to avoid leaking which emails are registered
- Only `user_id` and `user_name` go into `session` — never `password` or `password_hash`
- `/logout` must fully clear the session, not just unset one key

## Definition of done
- [ ] Logging in with the seeded demo user (`demo@spendly.com` / `demo123`) succeeds and redirects to `/`
- [ ] Logging in with a wrong password for an existing email shows "Invalid email or password." and does not start a session
- [ ] Logging in with a non-existent email shows the exact same generic error (no observable difference between the two failure cases)
- [ ] After a successful login, the navbar shows the signed-in state (name + "Sign out") instead of "Sign in" / "Get started"
- [ ] Visiting `/logout` clears the session, and the navbar reverts to the signed-out state on the next page load
- [ ] Inspecting the session cookie/data confirms no password or password_hash is ever stored in it
