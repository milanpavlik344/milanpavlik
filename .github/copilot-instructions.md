# Copilot instructions for this repository

## Project overview

This repository is a small FastAPI app for Mergington High School: students can browse extracurricular activities and submit sign-up requests. The app is intentionally simple and uses in-memory state rather than a database.

Important runtime pieces:

- `src/app.py` contains the FastAPI application, route handlers, and the in-memory `activities` store.
- `src/static/index.html` and `src/static/app.js` provide the browser UI and client-side API calls.
- `src/static/styles.css` contains the page styling.
- `requirements.txt` contains the Python dependencies; there is no package manager wrapper or build system beyond Python itself.

## Commands

Run the app locally from the repository root:

```bash
python -m uvicorn src.app:app --reload
```

The repo also includes a VS Code debug configuration in `.vscode/launch.json` that launches the same app via `uvicorn src.app:app --reload --reload-include src/static/*`.

Run the test suite:

```bash
python -m pytest -q
```

There is no dedicated lint command in the repo. This project currently relies on pytest for validation; if you add Python checks, prefer commands that fit the existing minimal setup rather than introducing a new toolchain unless the repo already expects it.

## Architecture and data flow

The app is centered around a single route module and a single in-memory dictionary:

- `GET /activities` returns the `activities` dict.
- `POST /activities/{activity_name}/signup` validates the activity, then appends the submitted email to `activity["participants"]`.
- The root route redirects to `/static/index.html`, so the frontend is served from the static directory.
- The browser fetches `/activities` on page load and posts signups via `fetch()` to the API.

Because the data is in memory, state resets whenever the server process restarts. This is intentional for the exercise and should be preserved unless a large repo-wide change is being made.

## Conventions specific to this codebase

- The app treats activity names as dictionary keys and uses the key as the primary identifier (`activities[activity_name]`).
- Student signups are represented as email strings in the `participants` list; the frontend encodes user input with `encodeURIComponent()` when creating the signup request URL.
- API validation is done with `HTTPException` rather than custom wrappers; invalid activity names should raise 404 and duplicate registrations should be rejected with a 400-style error when applicable.
- Frontend code expects the API response body to be JSON and displays the returned `detail` text for errors.
- Static assets are referenced under `/static/...` and are mounted in `app.py` with `FastAPI`’s `StaticFiles` integration.
- The project is intentionally small and flat: most app logic lives in `src/app.py`, not in separate service or model modules.

## Working style for this repo

- Keep changes scoped to the FastAPI app and its static assets unless the task clearly requires more structure.
- Prefer minimal, direct edits over introducing new abstraction layers or persistence systems.
- When changing API behavior, confirm the frontend still calls the same URL shape and expected JSON fields.
- Tests should remain lightweight and focused on the request/response behavior of the FastAPI app; there is no existing test harness beyond pytest.
