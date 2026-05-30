# Spec: Registration

## Overview
Implement user registration so new visitors can create a Spendly account. This step upgrades the existing stub GET /register route into a fully functional form that accepts a POST, validates input, hashes the password, and inserts a new row into the users table. On success the user is shown with a success message and then redirected to the login page. This is the entry point for all authenticated features that follow.

## Depends on
- Step 1: Database setup — `users` table exists, `werkzeug` is installed

## Routes
- `GET /register` — render the registration form (already exists; update to also accept POST)
- `POST /register` — validate input, check for duplicate email, hash password, insert user, redirect to `/login?registered=1` — public

## Database changes
No schema changes. Two helper functions are added to `database/db.py`:
- `create_user(name, email, password_hash)` — inserts a new row into `users`
- `get_user_by_email(email)` — returns a `Row` or `None` (used for duplicate-email check)

## Templates
- **Modify:** `templates/register.html` — already has `{% if error %}` block; no structural changes needed
- **Modify:** `templates/login.html` — add a success banner shown when `request.args.get('registered')` is truthy (e.g. `?registered=1`)

## Files to change
- `app.py` — set `app.secret_key` from an env variable (default to a dev fallback); change `/register` route to `methods=["GET", "POST"]`; add POST handler
- `database/db.py` — add `create_user()` and `get_user_by_email()` functions
- `templates/login.html` — add success banner for `?registered=1`

## Files to create
None

## New dependencies
No new dependencies.

## Rules for implementation
- No SQLAlchemy or ORMs — use raw `sqlite3` with parameterised queries only
- Passwords hashed with `werkzeug.security.generate_password_hash`; never store plaintext
- `app.secret_key` must be set (needed for Flask flash / future sessions); read from `SECRET_KEY` env var, fall back to a hard-coded dev string
- Validate all three fields server-side: name non-empty, valid email format (basic check), password ≥ 8 characters
- On any validation failure or duplicate email, re-render `register.html` passing `error=<message>`; preserve the submitted name and email in the template so the user does not have to retype them
- On success, redirect to `/login?registered=1` — do **not** auto-login the user (session / login is Step 3)
- Use CSS variables — never hardcode hex values in any new CSS
- All templates extend `base.html`

## Definition of done
- [ ] Submitting the form with valid, unique data inserts a row into `users` with a hashed password (confirm via `sqlite3 spendly.db "SELECT email, password_hash FROM users"`)
- [ ] The new user's `password_hash` starts with `scrypt:` or `pbkdf2:` — no plaintext stored
- [ ] Submitting with an email already in the database re-renders the form with an error message and does **not** create a duplicate row
- [ ] Submitting with a password shorter than 8 characters re-renders the form with a validation error
- [ ] Submitting with an empty name re-renders the form with a validation error
- [ ] After a successful registration the browser lands on `/login?registered=1`
- [ ] The login page shows a visible success notice (e.g. "Account created — please sign in") when `?registered=1` is present
- [ ] The `/register` `GET` route still works normally (no regression)
