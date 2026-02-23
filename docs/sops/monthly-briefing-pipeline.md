# Monthly Briefing Pipeline SOP

## When
- **Schedule**: 1st of each month (no launchd plist yet — manual trigger only)
- **Dependency**: Weekly recaps for the month must exist first. Run the final weekly recap before running the monthly.

## Prerequisites
- Weekly recaps exist for the target month (at minimum 2-3 weeks of data)
- Mac is awake, `claude` CLI authenticated, internet connection

## What It Does
1. **Build prompt** — Assembles the monthly command + synthesis/formatting skills + monthly template (no pre-fetch needed — monthly draws from existing weekly recaps)
2. **Run Claude** — Reads all weekly recaps for the month, performs deep synthesis into defining stories, trend lines, metrics, podcast highlights, and forward-looking analysis
3. **Save** — Writes to `BPG_Tech-News/YYYY/YYYY-MM-recap.md`
4. **Git push** — Commits and pushes to `bengio777/BPG_Tech-News_Catalog`
5. **Email** — Sends with subject "Monthly Tech Briefing -- YYYY-MM-recap"

## Run It
```bash
bash ~/.config/tech-news-briefing/run-briefing.sh monthly
```

Typical runtime: ~3-5 minutes.

## Verify It Worked
1. **Check the log**:
   ```bash
   cat ~/.config/tech-news-briefing/logs/$(date +%Y-%m-%d)-monthly.log
   ```

2. **Check the file exists**:
   ```bash
   ls -la ~/BPG_Tech-News/2026/$(date +%Y-%m)-recap.md
   ```

3. **Check the dashboard**: Look for the "M" badge in the monthly section at http://localhost:3000

4. **Check email**: Look for "Monthly Tech Briefing" in inbox

## Output Structure
The monthly recap is a long-form document (1500-3000 words, no tabs):
- **Month in Review** — Defining stories and narrative arc
- **Trend Lines** — Themes that strengthened or weakened
- **By the Numbers** — Aggregated metrics (funding, CVEs, launches)
- **Podcast Highlights** — Top episodes from the month
- **What to Watch** — Stories/themes likely to develop next month

## If It Fails
See [Troubleshooting Runbook](troubleshooting-runbook.md).

Common issue: If no weekly recaps exist for the month, Claude will have nothing to synthesize. Ensure weekly recaps are generated first.
