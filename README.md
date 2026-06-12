# Starlink Usage Scraper v2

Extracts daily data usage from your Starlink account and displays it on a local dashboard. Data can be viewed as a table and exported as CSV.

---

## Project Structure

```
starlink-scraper-v2/
├── app.py                  # Flask web server
├── scraper.py              # Selenium-based scraper logic
├── requirements.txt        # Python dependencies
├── templates/
│   └── index.html          # Frontend dashboard
└── output/
    └── starlink_usage.csv  # Generated CSV output
```

> **Note:** The `templates/` and `output/` folders must exist before running. If missing, create them:
> ```bash
> mkdir templates output
> ```

---

## How It Works

| File | Role |
|------|------|
| `scraper.py` | Opens Chrome via `undetected-chromedriver`, prompts you to log in manually, then navigates to the usage page and pulls daily GB data from the bar chart. Loops through all monthly tabs and saves to CSV. |
| `app.py` | Flask server running at `http://localhost:8080`. Exposes `/` (dashboard), `POST /api/scrape` (trigger scrape), and `GET /api/download-csv` (download CSV). |
| `templates/index.html` | Dashboard UI — shows usage data and provides scrape/export buttons. |

---

## Setup & Installation

### 1. Find your Python executable

> **Important — read this before anything else.**
>
> On Windows, `py` and `python` can point to **different Python installations**. If you install
> packages with one and run the app with the other, you'll get `ModuleNotFoundError` even though
> the install succeeded. Use the **same executable** for both steps.
>
> Check what you have:
> ```powershell
> py --version
> python --version
> where python
> ```
> If `where python` returns nothing, use `py` for everything. If both exist, prefer `py` — it's
> the Windows Python Launcher and picks the correct version.

### 2. Install dependencies

```powershell
py -m pip install -r requirements.txt
```

**Requirements:** `Flask`, `undetected-chromedriver`, `pandas`, `setuptools`

> You also need **Google Chrome** installed on your machine.

### 3. Set your Starlink URL

Open `scraper.py` and replace the placeholder with your own service-line URL:

```python
TARGET_URL = "https://starlink.com/account/service-line/AST-XXXXXXX-XXXXX-XX?..."
```

To find it: log in to [starlink.com](https://starlink.com), go to your account dashboard, and copy the URL from the address bar while on your service line page.

### 4. Start the server

```powershell
py app.py
```

Opens at `http://localhost:8080`.

---

## Running a Scrape

1. Go to `http://localhost:8080` in your browser
2. Click **"Run Sync"**
3. A Chrome window will open — **log in to your Starlink account manually**
4. After logging in, inside the Chrome window:
   - Click the **☰ three-line hamburger menu** on the side of the page
   - When the side menu opens, click **"Sign In"**
5. The scraper takes over from here — it navigates to your usage page and collects data across all monthly tabs
6. When done, data appears on the dashboard and is saved to `output/starlink_usage.csv`

> ⏱️ You have **45 seconds** after the Chrome window opens to complete login and click ☰ → Sign In.
> Need more time? Increase the `time.sleep(45)` value in `scraper.py`.

---

## Output Format

The CSV is saved to `output/starlink_usage.csv`:

```
Date,Usage_GB
2025-11-17,17.0
2025-11-18,13.0
```

Download it from the dashboard or via `GET /api/download-csv`.

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `ModuleNotFoundError: No module named 'flask'` | Use `py app.py` — `py` and `python` may point to different installs. Always use the same executable for both `pip install` and running the app. |
| "No bars found with any selector" | Refresh the page or wait for the chart to fully load |
| Chrome won't open | Update Chrome or check your `undetected-chromedriver` version |
| "CSV not found" error | Run a scrape first before attempting to download |
| No data after login | Make sure `TARGET_URL` matches your own Starlink account URL |
| Scraper runs before you finish logging in | Increase `time.sleep(45)` in `scraper.py` |
| "Could not find hamburger menu" | Wait for the page to fully load before clicking ☰; try scrolling up |
| "Sign In not found in menu" | The menu may not have fully rendered — wait a moment and try again |

---

## Notes

- **Manual login is intentional** — automating login increases bot detection risk, so it's left to you.
- **The hamburger menu step is required** — clicking ☰ → Sign In is what signals the scraper to start collecting.
- **Chrome version fallback** — the scraper tries `auto-detect → 148 → 147` if there are compatibility issues.
- **If the chart loads blank** — wait a moment and try the scrape again.
