# Agent Guidelines for Gemini-API

This file contains repo‑specific guidance to help agents work effectively in this codebase. Every line answers: “Would an agent likely miss this without help?” If not, it’s omitted.

## Development Commands

### Install for development
```bash
pip install -e .
```

Optional browser‑cookie support (for automatic cookie import):
```bash
pip install -e .[browser]
```

### Run tests
Tests are built with `unittest` (not pytest). They require live Gemini credentials; if credentials are not present the suite is skipped.

```bash
# Run all tests (credentials required)
export SECURE_1PSID="your_cookie_value"
export SECURE_1PSIDTS="your_cookie_value"
python -m unittest discover -s tests -v

# Run a single test file
python -m unittest tests.test_cli -v
```

**Important:** Tests use `unittest.IsolatedAsyncioTestCase`. Do not attempt to run them with `pytest` (pytest is disabled in `.vscode/settings.json`).

### Build the package
Version is managed by `setuptools_scm` (from git tags). Build with:
```bash
python -m build
```

### Lint & format
The project uses `black` for code formatting (see badge in README). No pre‑commit hooks or lint config files are present; run black manually if needed.

## Environment & Configuration

### Test credentials
- `SECURE_1PSID`, `SECURE_1PSIDTS` – required for any test that calls the real Gemini API. If unset, tests are skipped with a clear message.
- Credentials are read in `setUpClass` and written to a temporary JSON file that the CLI can consume.

### Cookie persistence
- The library can auto‑refresh cookies in the background. Set `GEMINI_COOKIE_PATH` to a writable directory to persist refreshed cookies across runs (useful in containerized environments).
- The CLI automatically updates the cookie JSON file after each run unless `--no-persist` is given.

## Project Structure

- **Source code**: `src/gemini_webapi/`
  - `client.py` – main `GeminiClient` and `ChatSession`
  - `components/` – mixins for chat, research, gem management
  - `types/` – Pydantic models for API responses
  - `utils/` – helpers (cookies, logging, file upload, etc.)
  - `constants.py` – model enum and other constants
- **CLI**: `cli.py` – standalone command‑line interface
- **Tests**: `tests/` – all live tests require credentials
- **Assets**: `assets/` – sample files used in examples

## CI/CD

- **Publishing**: Triggered on git tags matching `v[0-9]+.[0-9]+.[0-9]+`
  - `pypi-publish.yml` builds the package and publishes to PyPI using trusted publishing.
  - `github-release.yml` creates a GitHub release with release notes.
- **No separate test job** – tests are run locally only (they need live credentials).

## CLI Usage Notes

- The CLI expects a JSON cookie file:
  ```json
  { "__Secure-1PSID": "value", "__Secure-1PSIDTS": "value" }
  ```
- Global options (must appear **before** the subcommand):
  - `--cookies-json PATH` – path to cookie file (required)
  - `--proxy URL` – proxy URL (or uses `HTTPS_PROXY` env)
  - `--model NAME` – model name (see `models` command)
  - `--verbose` – enable debug logging
  - `--no-persist` – do not update cookie file after run
  - `--request-timeout SEC` – HTTP timeout (default 300)
- Subcommands: `ask`, `reply`, `list`, `read`, `models`, `download`, `inspect`, `research send|check|get`

## Code Style & Conventions

- **Logging**: uses `loguru`. Control level with `gemini_webapi.set_log_level()`.
- **Async**: the entire library is async‑first. Use `asyncio.run()` in examples.
- **Type hints**: Pydantic v2 is used for data models; type hints are thorough.
- **Docstrings**: follow standard Python conventions; no special format enforced.

## What’s Not Here

- Generic Python packaging advice (use `pyproject.toml`).
- How to use the Gemini API (see `README.md` for extensive examples).
- How to set up a Google account or obtain cookies (see README “Authentication”).
- Instructions for contributing (no `CONTRIBUTING.md` present).

## Quick Reference for Common Tasks

| Task | Command |
|------|---------|
| Install with browser‑cookie support | `pip install -e .[browser]` |
| Run all tests (with credentials) | `python -m unittest discover -s tests -v` |
| Build distribution | `python -m build` |
| Run CLI (with cookie file) | `python cli.py --cookies-json cookies.json ask "Hello"` |
| Check available models via CLI | `python cli.py --cookies-json cookies.json models` |
