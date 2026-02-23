# Daily Briefing Pipeline SOP

## When
- **Automated**: launchd fires at 6:00 AM MST daily
- **Manual**: Run anytime from terminal
- launchd will fire late if the Mac was asleep at 6 AM — it runs when you open the lid

## Prerequisites
- Mac is awake (or will wake before you need the briefing)
- `claude` CLI is authenticated (`claude auth` to check)
- SMTP credentials in Keychain (`security find-generic-password -s tech-news-briefing-smtp`)
- Internet connection for web searches and email delivery

## What It Does
1. **Pre-fetch** — Runs `fetch-osint.py` to pull CISA KEV, NVD CVEs, and RSS feeds into `~/.config/tech-news-briefing/prefetch/osint-YYYY-MM-DD.json`
2. **Build prompt** — Assembles the daily command + research/curation/formatting skills + daily template into one self-contained prompt
3. **Run Claude** — `claude -p --model sonnet --dangerously-skip-permissions` with the assembled prompt
4. **Save** — Writes briefing to `/Users/benjamingiordano/BPG_Tech-News/YYYY/week-WW/YYYY-MM-DD.md`
5. **Git push** — Commits and pushes to `bengio777/BPG_Tech-News_Catalog`
6. **Email** — Sends HTML + plain text email via Google Workspace SMTP

## Manual Re-run
```bash
bash ~/.config/tech-news-briefing/run-briefing.sh daily
```

## Verify It Worked
1. **Check the log**:
   ```bash
   cat ~/.config/tech-news-briefing/logs/$(date +%Y-%m-%d)-daily.log
   ```
   Look for "Exit code: 0" at the bottom.

2. **Check the file exists**:
   ```bash
   ls -la ~/BPG_Tech-News/2026/week-$(date +%V)/$(date +%Y-%m-%d).md
   ```

3. **Check the dashboard**: Open http://localhost:3000 (if dev server is running)

4. **Check email**: Look for "Daily Tech Briefing" in inbox

## If It Fails
See [Troubleshooting Runbook](troubleshooting-runbook.md).

Common issues:
- Log shows header but nothing else → `claude -p` likely hung. Check for stuck process and kill it.
- Pre-fetch WARN lines → Non-fatal. Briefing still generates, just without OSINT data.
- "Exit code: 1" → Check the log for the specific error above the exit line.
