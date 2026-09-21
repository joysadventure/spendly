# Spec: Registration

## Overview
Adds account creation to Spendly by implementing the `POST /register` handler. Users can already reach the registration form (`GET /register` renders `register.html`, which already POSTs to `/register`), but there is no backend logic to validate input, hash the password, or insert the new user. On success the user is shown with a success message and then redirected to the login page. This is the entry point for all authenticated features that follow (login, profile, expenses).

## Depends on
Step 1 — Database Setup (`.claude/specs/01-database-setup.md`). Requires `get_db()` and the `users` table to already exist, which they do.

## Routes
- `POST /register` — validates form input and creates a new user account — public
- `GET /register` already exists and is unchanged.
- `GET /login` — extended (not new) to accept an optional `?registered=1` query flag and show a success message — public.

## Database changes
No database changes. The existing `users` table (`id`, `name`, `email`, `password_hash`, `created_at`) already covers registration; no new columns or tables needed.

## Templates
- **Modify:** `templates/register.html` — repopulate the `name` and `email` inputs with the previously submitted values when re-rendering after a validation error, so the user isn't forced to retype the whole form. The existing `{% if error %}` block is reused as-is.
- **Modify:** `templates/login.html` — add a success banner (`{% if success %}...{% endif %}`) shown when the user arrives via `?registered=1` after a successful registration, alongside the existing `{% if error %}` block.

## Files to change
- `app.py` — add `POST` handling to `/register`: read and validate form fields, check for duplicate email, hash the password, insert the user, redirect to `/login?registered=1` on success. Extend `/login` to read the `registered` query flag and pass it to the template.
- `templates/register.html` — repopulate `name`/`email` on validation error.
- `templates/login.html` — add the success banner.
- `static/css/style.css` — add an `.auth-success` rule using existing CSS custom properties (no hardcoded hex values), mirroring how `.auth-error` is already styled.

## Files to create
None.

## New dependencies
No new dependencies. `werkzeug.security.generate_password_hash` is already available (werkzeug is a Flask dependency, already used in `database/db.py`).

## Rules for implementation
- No SQLAlchemy or ORMs
- Parameterised queries only
- Passwords hashed with werkzeug
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`
- Name, email, and password are all required; password must be at least 8 characters
- Email uniqueness is enforced at the database level (`UNIQUE` constraint on `users.email`) — catch the resulting `IntegrityError` and show a friendly "Email already registered" error rather than a 500
- No session/flash mechanism is introduced; the success message travels via a `?registered=1` query flag rather than Flask sessions, since no `SECRET_KEY` or session usage exists anywhere else in this app

## Definition of done
- [ ] Submitting the form with valid name/email/password creates a new row in `users` with a hashed password (not plaintext)
- [ ] Submitting with an email that already exists shows an error message on the page and does not create a duplicate row or crash the app
- [ ] Submitting with a password under 8 characters shows a validation error and does not insert a row
- [ ] Submitting with any required field empty shows a validation error and does not insert a row
- [ ] After a validation error, the previously entered name and email are still shown in the form
- [ ] On successful registration, the browser is redirected to `/login?registered=1` and a success message is shown there
