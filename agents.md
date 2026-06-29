# AGENTS.md

This file provides crucial instructions and project context for AI coding agents operating in this repository.

## Setup & Environment
- **Virtual Environment:** Ensure the Python virtual environment is activated before running any Python scripts. 
  ```bash
  source .venv/bin/activate
  ```

## Execution Workflow (CRITICAL)
Before running the main scraper, you **MUST** ensure the following services are active in this exact order:

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

3. **Core Scraper Scripts:**
   The main logic resides in `dorking_master/`. Always run them in this sequence:
   - First, run `python dorking_master/create_session_pool.py` to create Playwright browser sessions.
   - Second, run `python dorking_master/dork_scraper.py`. This is the final, most refined script that utilizes the sessions to perform the actual dorking.
   
   *(Note: `script.py`, `script1.py`, and `script2.py` inside `dorking_master/` are legacy/previous versions and should generally be ignored).*

## Project Architecture
- **`dorking_master/`**: **(The Main Brain)** This is the core intelligence engine containing the primary logic for executing Google dorks, automating browser sessions, and managing scraping workflows.
- **`proxies/`**: **(Proxy Helper)** Dedicated support system managing proxy health and rotation. 
- **`tools/`**: **(Experimental / Temporary)** A sandbox directory used for testing third-party open-source tools (e.g., Pagodo). It also houses legacy directories like `frontend/`, `backend/`, and `Reports/` that were associated with these external tools and are no longer actively used in the core workflow.
- **`dorks.txt`**: A list of active Google dorks used by the scraper.

## Agent Guidelines & Conventions
- **Modifying Core Logic:** All new scraping logic, playbooks, or browser automation strategies should be integrated into `dorking_master/dork_scraper.py` or its supporting modular files.
- **Network Requests:** All scraping-related network requests must utilize the proxy manager to guarantee anonymity and avoid rate limits.
- **Testing Tools:** When working in the `tools/` directory, thoroughly test the open-source logic before migrating the "useful brains" into the `dorking_master` component. Ask the user to review the temporary experiments before cleaning them up.
