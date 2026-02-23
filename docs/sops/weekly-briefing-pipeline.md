# Weekly Briefing Pipeline SOP

## When
- **Schedule**: Sundays (no launchd plist yet — manual trigger only)
- **Best time**: After the last daily briefing of the week has been generated

## Prerequisites
- At least 1 daily briefing exists in this week's folder (`BPG_Tech-News/YYYY/week-WW/`)
- Fewer than 3 dailies triggers a warning but the pipeline still runs
- Mac is awake, `claude` CLI authenticated, internet connection

## What It Does
1. **Pre-fetch** — Runs `fetch-osint.py` (CISA KEV, NVD, RSS) and `fetch-podcasts.py` (Spotify episodes, Apple Charts top 25)
2. **Build prompt** — Assembles the weekly command + synthesis/podcasts/formatting skills + weekly template
3. **Run Claude** — Reads all daily briefings from the week, synthesizes cross-day patterns, matches podcast episodes, formats 4-tab recap
4. **Save** — Writes to `BPG_Tech-News/YYYY/week-WW/week-WW-recap.md`
5. **Git push** — Commits and pushes to `bengio777/BPG_Tech-News_Catalog`
6. **Email** — Sends with subject "Weekly Tech Briefing -- week-WW-recap"

## Run It
```bash
bash ~/.config/tech-news-briefing/run-briefing.sh weekly
```

Typical runtime: ~3-5 minutes.

## Verify It Worked
1. **Check the log**:
   ```bash
   cat ~/.config/tech-news-briefing/logs/$(date +%Y-%m-%d)-weekly.log
   ```
   Look for "Exit code: 0" and the pipeline report (dailies read, stories synthesized, etc.).

2. **Check the file exists**:
   ```bash
   ls -la ~/BPG_Tech-News/2026/week-$(date +%V)/week-$(date +%V)-recap.md
   ```

3. **Check the dashboard**: Look for the "W" badge in the sidebar at http://localhost:3000

4. **Check email**: Look for "Weekly Tech Briefing" in inbox

## Output Structure
The weekly recap has 4 tabs:
- **Week in AI** — Top persistent stories + emerging themes with analysis paragraphs
- **Breakthroughs Recap** — Week's standout breakthroughs with context
- **Cyber Weekly** — Persistent threats, incidents, By the Numbers
- **Podcasts** — News-Connected, Trending, Discovery Pick

## If It Fails
See [Troubleshooting Runbook](troubleshooting-runbook.md).

Note: If Spotify credentials aren't set up, the Podcasts tab will only show Apple Charts data and RSS-sourced episodes. This is expected until Keychain credentials are configured.
