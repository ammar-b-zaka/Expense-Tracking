# Spec: Profile Page Design

## Overview
Implement the authenticated `/profile` page so logged-in users can see their account details and a summary of their spending activity. This step replaces the current stub string response with a real rendered template that shows the user's name, email, member-since date, and three at-a-glance stats: total amount spent, number of expenses recorded, and their top spending category. A helper function is added to `database/db.py` to compute these stats in a single query. The page is protected by the `login_required` decorator introduced in Step 3.

## Depends on
- Step 1: Database setup — `users` and `expenses` tables, `get_db()` exists
- Step 2: Registration — `users` table populated, `get_user_by_id()` exists
- Step 3: Login and Logout — `login_required` decorator exists, `session` holds `user_id`

## Routes
- `GET /profile` — render the profile page with user info and spending stats — logged-in only

## Database changes
No schema changes. One new helper function added to `database/db.py`:
- `get_user_stats(user_id)` — returns a dict with keys `total_spent` (REAL), `expense_count` (INTEGER), and `top_category` (TEXT or None); uses a single parameterised query with aggregation and a subquery/GROUP BY for top category

## Templates
- **Create:** `templates/profile.html` — extends `base.html`; displays:
  - A page header with the user's name and a subline showing their email
  - A "Member since" date formatted as Month DD, YYYY (e.g. "January 05, 2026")
  - Three stat cards in a row: Total Spent, No. of Expenses, Top Category
  - A call-to-action button linking to `/expenses/add` for users with no expenses yet
- **Modify:** none

## Files to change
- `app.py` — import `get_user_stats`; add `@login_required` to the `/profile` route; pass `user` and `stats` to `render_template`
- `database/db.py` — add `get_user_stats(user_id)` function
- `static/css/style.css` — add styles for `.profile-header`, `.profile-meta`, `.stats-grid`, `.stat-card`, `.stat-value`, `.stat-label`; use CSS variables throughout

## Files to create
- `templates/profile.html`

## New dependencies
No new dependencies.

## Rules for implementation
- No SQLAlchemy or ORMs — use raw `sqlite3` with parameterised queries only
- `get_user_stats` must use a single DB connection; close it before returning
- The `/profile` route must be decorated with `@login_required`
- `total_spent` should be formatted as a currency string in the template (e.g. `$358.24`) — do not store formatted strings in the DB
- If the user has no expenses yet, `expense_count` is 0, `total_spent` is 0.0, and `top_category` is None — the template must handle the None case gracefully (show "—" or "No expenses yet")
- Member-since date is stored as `TEXT` in SQLite; parse it with `datetime.strptime` in the route and pass a formatted string to the template
- Use CSS variables — never hardcode hex values in any new CSS
- All templates extend `base.html`
- Stat card layout must use CSS Grid (`stats-grid`) — no inline styles, no Bootstrap

## Definition of done
- [ ] `GET /profile` while not authenticated redirects to `/login?next=/profile`
- [ ] `GET /profile` while authenticated renders the profile page (HTTP 200, no exceptions)
- [ ] The page shows the logged-in user's name as the page heading
- [ ] The page shows the user's email below the heading
- [ ] The "Member since" date is formatted as "Month DD, YYYY" and matches the user's `created_at` in the DB
- [ ] The "Total Spent" stat card shows the sum of all the user's expenses formatted as a dollar amount
- [ ] The "No. of Expenses" stat card shows the correct integer count
- [ ] The "Top Category" stat card shows the category with the highest total spend for that user
- [ ] When the demo user is logged in all three stat cards show non-zero / non-None values (seed data covers this)
- [ ] When `top_category` is None (new user with no expenses) the template renders "—" rather than crashing
- [ ] The stats grid is responsive: three cards side-by-side on desktop, stacked on narrow viewports
- [ ] All new CSS uses variables from `:root` — no hardcoded hex colours
