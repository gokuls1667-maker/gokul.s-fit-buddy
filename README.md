# FitBuddy

A complete local FastAPI application based on **FitBuddy – AI Fitness Plan Generator using Gemini Models.pdf**. It creates seven-day workout plans, generates nutrition/recovery tips, refines plans from feedback, and stores every version in SQLite.

The frontend is HTML/CSS/JavaScript rendered through Jinja2. There is no separate frontend server, Node build, GPU requirement, or database installation.

## What is included

- Responsive plan builder, seven-day plan viewer, nutrition tips, and feedback forms.
- FastAPI JSON endpoints and interactive Swagger documentation at `/docs`.
- Google Gen AI SDK integration with schema-validated model output, timeouts, and bounded retries.
- A clearly labeled offline demo mode that works without a key or internet after installation.
- SQLite/SQLAlchemy persistence with an immutable original plan and full revision history.
- A protected coach dashboard to view, compare, and delete users and their plans.
- Browser session ownership, per-plan API access tokens, CSRF checks on forms, and escaped HTML.
- JSON export and browser Print / Save PDF for all seven days.
- Automated application, schema, and mocked Gemini SDK tests; VS Code debugging configuration.

## 1. Open the project in VS Code (Windows)

1. Extract the ZIP. Open the **FitBuddy** folder, the one containing `README.md` and `requirements.txt`, with **File > Open Folder**.
2. Install Python 3.12 and the Microsoft **Python** extension in VS Code. Python 3.12 is the tested version; the code requires Python 3.11 or newer.
3. Open **Terminal > New Terminal**. Use PowerShell in the FitBuddy folder.
4. Run these commands individually:

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements-dev.txt
.\.venv\Scripts\python.exe scripts/setup.py
```

The direct interpreter path avoids PowerShell activation-policy problems. No execution-policy change is needed. If `py` is unavailable but `python --version` shows the correct Python version, use `python -m venv .venv` for the first command.

`setup.py` creates a private `.env` with a unique coach password and session secret. It leaves an existing `.env` unchanged. Dependencies need internet during installation; subsequent demo operation is offline.

## 2. Run immediately in offline demo mode

```powershell
.\.venv\Scripts\python.exe -m uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
```

Open [http://127.0.0.1:8000](http://127.0.0.1:8000). Keep the terminal running. Stop the server with **Ctrl+C**.

The default `.env` has `AI_MODE=demo`. Demo mode uses deterministic templates; it does **not** use Gemini or claim to perform open-ended AI reasoning. It supports these feedback phrases: `more rest days`, `more cardio`, `add yoga`, and `make it easier`. Other feedback returns a clear explanation without saving a fake revision.

Use **127.0.0.1** consistently. Switching to `localhost` uses a different browser cookie and can make a saved plan appear inaccessible. Use Restore access with its token if needed.

## 3. Enable real Gemini generation

1. Create an API key in [Google AI Studio](https://aistudio.google.com/apikey).
2. Open `.env` in VS Code and change:

```dotenv
AI_MODE=gemini
GEMINI_API_KEY=paste_your_own_key_here
GEMINI_WORKOUT_MODEL=gemini-3.8-flash
GEMINI_TIP_MODEL=gemini-3.5-flash-lite
```

3. Save `.env`. Stop and restart Uvicorn; changing `.env` does not reliably trigger its reload watcher.
4. Validate the key, model access, and real generation from the terminal:

```powershell
.\.venv\Scripts\python.exe scripts/check_gemini.py
```

This diagnostic lists available models, checks the two configured IDs, and calls real workout generation, feedback revision, and nutrition generation. It uses your Google quota and may incur charges under your account. It does not store a test profile. An API key, enabled model access, sufficient quota, and network access to Google are required. A ChatGPT subscription does not supply a Gemini key.

Google's model catalog and access policies change. Both model names are configurable; if the check reports a missing model, use an appropriate text model shown for your key. There is no automatic switch to another model or to offline demo after a live error.

The PDF uses the old `google-generativeai` SDK and Gemini 1.5 references. This implementation uses `google-genai` and current configurable model IDs. The model catalog and deprecation pages were checked on 2026-09-25; see [requirements analysis](docs/REQUIREMENTS_ANALYSIS.md).

## 4. Use the complete application

1. Enter a name, a unique user ID, age, weight, goal, intensity, experience, equipment, and session duration. Select **Build my 7-day plan**.
2. Copy the **plan access token** shown once after creation. It restores access in another browser and authenticates API requests. Never share it publicly.
3. Select each day to see warm-up, exercises, sets/repetitions or duration, rest intervals, and cooldown.
4. Submit feedback. The new plan appears as the next version; the original and all prior versions remain accessible.
5. Use **Version history** to compare old and new plans, **Export JSON** for the entire history, or **Print / Save PDF** to print the selected version with all seven days.
6. Return to the home page to see plans created in this browser. The signed browser session remembers up to 20 profiles for 30 days. The database retains all profiles until explicitly deleted.
7. Open **Coach dashboard**. Enter `ADMIN_USERNAME` and `ADMIN_PASSWORD` from your local `.env` in the browser authentication prompt. The default username is `admin`; the generated password is unique.
8. The coach can inspect original/latest plans and remove a user, which deletes all of that user's plan versions. The browser asks for deletion confirmation.

The home page also has a standalone nutrition/recovery tip form. A fresh standalone tip is displayed separately; it does not silently replace the tip attached to a saved revision.

## 5. Test

With the development dependencies installed, run:

```powershell
.\.venv\Scripts\python.exe -m pytest -q
.\.venv\Scripts\python.exe -m pip check
```

The automated suite uses temporary SQLite databases and mocked HTTP responses for the real Google SDK. It requires no API key, uses no Google quota, and does not modify your application database.

To exercise the running server end to end, keep Uvicorn open and run this in a second terminal:

```powershell
.\.venv\Scripts\python.exe scripts/smoke_test.py
```

This creates a `smoke-...` profile, generates a plan, submits feedback, verifies the original was preserved, fetches history, and requests a tip. Delete that test profile through the coach dashboard afterward. In Gemini mode, this smoke test uses live API quota.

Optional real-browser automation:

```powershell
.\.venv\Scripts\python.exe -m pip install -r requirements-browser.txt
.\.venv\Scripts\python.exe -m playwright install chromium
.\.venv\Scripts\python.exe scripts/browser_check.py
```

The browser check starts its own temporary demo server and database; screenshots are saved under `test-results/`. See [the test report](docs/TEST_REPORT.md) for what was verified in the supplied build and what still requires your key.

## 6. VS Code debugging

Use **Ctrl+Shift+P > Python: Select Interpreter** and select the project's `.venv` interpreter. Then press **F5** and choose **FitBuddy: FastAPI**. Stop an existing Uvicorn process first so port 8000 is available. The launch configuration supports Python and Jinja debugging. The built-in test task runs `pytest` using the selected interpreter.

## macOS / Linux setup

```bash
python3 -m venv .venv
.venv/bin/python -m pip install -r requirements-dev.txt
.venv/bin/python scripts/setup.py
.venv/bin/python -m uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
```

## Configuration

| Setting | Meaning |
| --- | --- |
| `AI_MODE` | `demo` or `gemini`; demo is the default. |
| `GEMINI_API_KEY` | Your server-side Google key. `GOOGLE_API_KEY` is accepted as an alternative. |
| `GEMINI_WORKOUT_MODEL` | Model used for complete workout generation and feedback revisions. |
| `GEMINI_TIP_MODEL` | Model used for nutrition/recovery tips. |
| `AI_TIMEOUT_SECONDS` | Per-attempt timeout, 5–180 seconds; default 60. Transient provider retries are bounded to two attempts. A plan request normally involves two model calls. |
| `DATABASE_URL` | Blank uses the absolute project path `data/fitbuddy.db`. A supplied SQLite URL is used as written. |
| `SESSION_SECRET` | At least 32 characters; `setup.py` generates it. If blank, the app persists a random secret in `data/session.key`. |
| `ADMIN_USERNAME` / `ADMIN_PASSWORD` | Coach HTTP Basic credentials; coach routes are unavailable when the password is blank. |
| `COOKIE_SECURE` | `false` for local HTTP. Use `true` only with correctly configured HTTPS. |

Real environment variables override `.env`. Credentials, `.env`, browser sessions, and database contents are excluded from version control and the delivered ZIP. In Gemini mode, training details and feedback go to Google; the application excludes names, user IDs, tokens, and credentials from prompts. Avoid putting identifying information into free-text feedback.

## Troubleshooting

| Symptom | Action |
| --- | --- |
| `No module named ...` | Install requirements using the same `.venv` interpreter used to run Uvicorn. |
| `Could not import module app.main` | Open the project folder containing `app/`, then use exactly `python -m uvicorn app.main:app`. |
| Port 8000 is occupied | Stop the existing process or add `--port 8001` and visit that port. |
| Gemini credentials rejected | Correct the key/project access in AI Studio, save `.env`, and restart. |
| Model unavailable / 404 | Run `scripts/check_gemini.py`, update the configured IDs, then restart. |
| Quota / 429 | Check AI Studio quota and billing. Retry later or explicitly use demo mode. |
| Timeout / 504 | Check connectivity. Retry later; saved plans are unchanged. |
| Invalid generated plan / 502 | The output failed schema or time-limit validation. Retry; nothing partial was saved. |
| User ID already exists / 409 | Open its saved plan or choose a new ID. IDs are case insensitive. |
| Newer plan exists / 409 | Reload the current plan before submitting feedback again. |
| Form expired / 403 | Reload to receive a fresh CSRF token; verify cookies are enabled. |
| Coach login unavailable / 503 | Run `scripts/setup.py` before first use. If `.env` already exists, set a strong `ADMIN_PASSWORD` there and restart. |
| Lost browser access | Restore with the user ID and token. A coach can still view the plan with coach credentials. |
| Script execution is disabled | Use the direct `.venv\Scripts\python.exe` commands above; activation is unnecessary. |

Data is in `data/fitbuddy.db`. Stop the app before copying the data folder for a consistent backup. Preserve `.env`/session secrets separately if you want existing browser sessions to continue working. Database schema changes require a migration; do not delete an existing database to resolve a development error.

This delivery targets local coursework and local demonstrations, as requested in the PDF. Keep the default loopback binding. Public hosting would require a separate deployment design with HTTPS, appropriate user authentication, rate limiting, and operational controls. The generated plans are general education, not medical advice or individualized injury rehabilitation.

## Source map

| Location | Responsibility |
| --- | --- |
| `app/main.py` | App factory, lifespan, middleware, error handling, static mount. |
| `app/routes.py` | HTML forms, JSON APIs, coach pages. |
| `app/config.py` | Environment settings, stable filesystem paths. |
| `app/schemas.py` | Profile, feedback, workout, tip, and response validation. |
| `app/models.py`, `app/database.py` | SQLite schema, ORM, sessions, database initialization. |
| `app/services.py` | Atomic creation, versioned updates, optimistic concurrency, deletion. |
| `app/security.py` | Coach authentication, CSRF, browser ownership, token hashing. |
| `app/ai_provider.py` | Google SDK, prompts, structured generation, provider errors. |
| `app/demo_provider.py` | Explicit offline example provider. |
| `app/gemini_generator.py`, `app/gemini_flash_generator.py`, `app/updated_plan.py` | The documented generation/tip/revision entry points. |
| `templates/` | Complete Jinja pages, including the three pages named in the PDF. |
| `static/` | CSS, JavaScript, local SVG brand asset. |
| `scripts/` | Setup, live Gemini check, running-server smoke test, browser check. |
| `tests/` | Isolated automated tests. |
| `.vscode/` | Interpreter, debugging, and test-task settings. |
| `docs/` | Requirements mapping, architecture, API examples, verification report. |

Detailed [architecture and requirements mapping](docs/REQUIREMENTS_ANALYSIS.md) and [API examples](docs/API.md) are included.
