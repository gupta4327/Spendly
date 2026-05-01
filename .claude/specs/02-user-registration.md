# Spec: Registration

## Overview
This step wires up the registration form so new users can create an account. It adds a `POST /register` handler that validates input, inserts a hashed-password row into `users`, stores the new user's id in the Flask session, and redirects to `/profile`. It also adds the `create_user()` DB helper and configures a `SECRET_KEY` so Flask sessions work.

## Depends on
Step 01 — Database Setup (users table must exist, `get_db()` must be working).

## Routes
- `POST /register` — validate form fields, create user, set session, redirect to `/profile` — public

The existing `GET /register` route is modified to accept both methods (no logic change for GET).

## Database changes
No new tables or columns. A new helper function is added to `database/db.py`:

- `create_user(name, email, password)` — hashes the password and inserts a row into `users`; returns the new row's `id`; raises `sqlite3.IntegrityError` if the email is already taken.

## Templates
- **Modify:** `templates/register.html`
  - Fix form action from hardcoded `/register` to `{{ url_for('register') }}`
  - No other changes (error display via `{{ error }}` is already in place)

## Files to change
- `app.py` — add `SECRET_KEY`; extend imports (`request`, `redirect`, `session`); change `register()` to handle GET + POST
- `database/db.py` — add `create_user()` function
- `templates/register.html` — fix hardcoded form action URL

## Files to create
None.

## New dependencies
No new dependencies.

## Rules for implementation
- No SQLAlchemy or ORMs
- Parameterised queries only — never f-strings in SQL
- Passwords hashed with `werkzeug.security.generate_password_hash` — never store plaintext
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`
- `SECRET_KEY` must be set before any session use; use a hard-coded dev string (e.g. `"dev-secret-change-me"`) — note in a comment that this must be replaced in production
- After successful registration, set `session["user_id"]` and `session["user_name"]`, then `redirect(url_for("profile"))`
- On validation failure, re-render `register.html` with an `error=` kwarg — do **not** redirect
- Validate: name non-empty, valid email format (basic — `@` present), password ≥ 8 characters; catch `IntegrityError` for duplicate email

## Definition of done
- [ ] Submitting the form with valid data creates a new row in `users` with a hashed password (verify with `sqlite3 spendly.db "SELECT email, password_hash FROM users;"`)
- [ ] After successful registration, the browser lands on `/profile` (stub response is fine)
- [ ] Submitting with a duplicate email shows an error message on the register page without crashing
- [ ] Submitting with a password shorter than 8 characters shows a validation error on the register page
- [ ] Submitting with an empty name shows a validation error on the register page
- [ ] `session["user_id"]` is set after registration (verify with Flask debug toolbar or by adding a temp print)
- [ ] The app starts without errors and `GET /register` still renders the form
