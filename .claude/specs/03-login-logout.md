# Spec: Login and Logout

## Overview
Implement session-based login and logout so registered users can authenticate and access protected areas of Spendly. This step upgrades the existing stub `GET /login` route into a fully functional form that accepts a `POST`, verifies credentials against the `users` table, and establishes a Flask session. A `login_required` decorator is introduced as the access-control primitive used by all authenticated routes going forward. Logout clears the session and redirects the user to the landing page. The navbar in `base.html` is updated to be session-aware, hiding the auth links once a user is signed in.

## Depends on
- Step 1: Database setup — `users` table, `werkzeug` installed
- Step 2: Registration — `get_user_by_email()` exists, `app.secret_key` is set

## Routes
- `GET /login` — render the login form; redirect to `/profile` if the user is already authenticated — public
- `POST /login` — validate credentials, call `check_password_hash`, set `session`, redirect to `/profile` on success — public
- `GET /logout` — clear the session, redirect to `/` — logged-in

## Database changes
No schema changes. One new helper function added to `database/db.py`:
- `get_user_by_id(user_id)` — returns a `Row` or `None`; used by `login_required` to confirm the user still exists

## Templates
- **Modify:** `templates/login.html` — already contains the POST form and `{% if error %}` block; no structural changes needed (the POST handler in `app.py` drives it)
- **Modify:** `templates/base.html` — make the nav session-aware:
  - When **not** authenticated: show existing "Sign in" and "Get started" links (no change)
  - When **authenticated**: replace those links with the user's name (non-clickable or links to `/profile`) and a "Sign out" link to `/logout`

## Files to change
- `app.py` — add `session` and `check_password_hash` to imports; add `login_required` decorator; upgrade `/login` to `methods=["GET", "POST"]` with POST handler; implement `/logout`
- `database/db.py` — add `get_user_by_id(user_id)` function
- `templates/base.html` — update nav to branch on `session.get('user_id')`

## Files to create
None

## New dependencies
No new dependencies. (`flask.session` is built into Flask; `check_password_hash` is already in `werkzeug`.)

## Rules for implementation
- No SQLAlchemy or ORMs — use raw `sqlite3` with parameterised queries only
- Passwords verified with `werkzeug.security.check_password_hash`; never compare plaintext
- Store only `user_id` and `user_name` in the session — do not store the password hash or full user row
- `login_required` must redirect to `url_for('login')` with a `next` query-param set to `request.path` so the user can be sent back after signing in (implement the redirect-after-login in the POST handler)
- On login failure (wrong email or wrong password) render a single generic error — "Invalid email or password." — do not reveal which field was wrong
- On `GET /login` when already authenticated, redirect to `/profile`
- `GET /logout` must work even if the session is already empty (no error)
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`

## Definition of done
- [ ] `POST /login` with correct credentials sets the session and redirects to `/profile`
- [ ] `POST /login` with a wrong password re-renders `login.html` with "Invalid email or password."
- [ ] `POST /login` with an unregistered email re-renders `login.html` with the same generic error
- [ ] `GET /login` while already signed in redirects to `/profile` without re-rendering the form
- [ ] `GET /logout` clears the session — visiting `/login` afterward shows the sign-in form (no auto-redirect)
- [ ] After logout the navbar shows "Sign in" and "Get started" again
- [ ] While logged in the navbar shows the user's name and a "Sign out" link instead of "Sign in"/"Get started"
- [ ] A route decorated with `login_required` (e.g. `/profile`) redirects to `/login?next=/profile` when the user is not authenticated
- [ ] After logging in via that redirect, the user lands on `/profile` (the `next` param is honoured)
- [ ] `GET /logout` with an empty session returns a redirect and does not raise a 500
