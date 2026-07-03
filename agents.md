# AGENTS.md

This file provides crucial instructions and project context for AI coding agents operating in this repository.

## Setup & Environment
- **Virtual Environment:** Ensure the Python virtual environment is activated before running any Python scripts. 
  ```bash
  source .venv/bin/activate
  ```

## Execution Workflow (CRITICAL)
Before running the scrapers, you **MUST** ensure the following services are active in this exact order:

1. **VPN Connection:**
   The scraper requires an active NordVPN connection.
   - Command to connect: `nordvpn connect`
   - Command to verify: `nordvpn status`
   *Wait until the status says "Connected" before proceeding.*

2. **Proxy Manager:**
   The `Auto_proxy_manager` handles scraping proxies using the open-source `proxy-scraper-checker`. It manages the proxy pool, monitors its health, and listens on port `9595`.
   - Working Directory: `/home/dorking/OSINT/proxies/Auto_proxy_manager`
   - Command to run: `cargo run --release`
   *Ensure this helper is running in the background before starting the Python scripts.*

3. **Core Services (two standalone FastAPI apps):**

   **URL discovery** — `dorking_master/` on port `8000`:
   ```bash
   python dorking_master/create_session_pool.py   # Google sessions (once)
   python dorking_master/dork_scraper.py
   ```

   **Content extraction** — `content_scraper/` on port `8001`:
   ```bash
   python content_scraper/create_tiktok_session.py   # TikTok cookies (once, if needed)
   python content_scraper/content_scraper.py
   ```

   **TikTok burner session:** Playwright login often gets TikTok `error_code 7` (bot detection). **Log in via real Chrome**, then import cookies:
   ```bash
   google-chrome --remote-debugging-port=9222 --user-data-dir="$HOME/.osint-tiktok-chrome" https://www.tiktok.com
   # log in + 2FA + open comments in that window, then:
   cd content_scraper
   TIKTOK_SESSION_FILE=sessions/tiktok_burner.json OSINT_USE_PROXY=0 python import_tiktok_session_from_chrome.py
   ```
   Alternative: `OSINT_USE_PROXY=0 python create_tiktok_burner_session.py` (persistent Chrome profile, strips `--enable-automation`).
   Scrape: `TIKTOK_SESSION_FILE=sessions/tiktok_burner.json OSINT_USE_PROXY=0 python content_scraper.py`

   **TikTok comment debug screenshots:** API tasks save step PNGs to `content_scraper/Results/screenshots/{task_id}/{aweme_id}/` (enabled by default). Paths are listed in each result's `comment_debug.screenshots`. Set `OSINT_DEBUG_SCREENSHOTS=0` to disable.

   **TikTok VPN-only (skip proxy):** If the local proxy pool triggers TikTok Verification, run with `OSINT_USE_PROXY=0` so Playwright uses NordVPN directly. Check `comment_debug.fetch_source: playwright_direct`.

   **TikTok slider captcha:** Comment scrapes auto-try local slider solving (`OSINT_TIKTOK_CAPTCHA_SOLVE=1`). For reliable solves, set `CAPSOLVER_API_KEY` (CapSolver AntiTikTokTask). Check `comment_debug.captcha_solve_method` (`slider_bruteforce`, `capsolver`, or `failed`).

   *(Note: `script.py`, `script1.py`, and `script2.py` inside `dorking_master/legacy_models/` are superseded versions).*

## Project Architecture
- **`dorking_master/`**: Google dork URL discovery. Saves `Results/results_{task_id}.json` + `tasks.db` on port **8000**.
- **`content_scraper/`**: Playwright content extraction (profiles, posts, comments). Saves `Results/content_{task_id}.json` + `tasks.db` on port **8001**. Accepts explicit `urls` or a dork `dork_task_id` (reads `dorking_master/Results/results_{id}.json`).
- **`proxies/`**: Proxy pool manager (port `9595`).
- **`tools/`**: Experimental third-party OSINT tools (sandbox).
- **`dorks.txt`**: Active Google dorks list.

## Agent Guidelines & Conventions
- **dorking_master changes:** URL discovery, dork logic, `url_classifier.py` for canonical URLs from Google results.
- **content_scraper changes:** Playwright adapters, content parsing, session cookies — do not import from `dorking_master`.
- **Network Requests:** All scraping-related network requests must utilize the proxy manager (`127.0.0.1:9595`).
- **Testing Tools:** Experiment in `tools/` first; migrate proven logic into the appropriate service after user review.