# Troubleshooting Runbook

## Log Locations
All pipeline logs are at:
```
~/.config/tech-news-briefing/logs/YYYY-MM-DD-{cadence}.log
```
The launchd stdout/stderr logs (overwritten each run):
```
~/.config/tech-news-briefing/logs/launchd-stdout.log
~/.config/tech-news-briefing/logs/launchd-stderr.log
```

## Quick Health Check
```bash
# Did today's daily run?
ls -la ~/BPG_Tech-News/2026/week-$(date +%V)/$(date +%Y-%m-%d).md

# What does the log say?
tail -20 ~/.config/tech-news-briefing/logs/$(date +%Y-%m-%d)-daily.log

# Any stuck claude processes?
ps aux | grep "claude" | grep -v grep
```

---

## Issue: Pipeline Hung (No Output After Header)

**Symptoms**: Log shows the header ("Tech News Briefing — ...") but nothing after, or only the pre-fetch phase. Process runs for hours with minimal CPU.

**Cause**: `claude -p` is stuck. This was previously caused by passing slash commands (which `-p` mode can't parse). The current `run-briefing.sh` uses self-contained prompts, so this should be resolved.

**Fix**:
1. Find and kill the stuck process:
   ```bash
   ps aux | grep "claude" | grep -v grep
   kill <PID>
   ```
2. Check that `run-briefing.sh` is using the self-contained prompt approach (should have `strip_frontmatter` and `PROMPT_FILE` in it, NOT `"Run /tech-news-briefing:briefing"`)
3. Re-run manually:
   ```bash
   bash ~/.config/tech-news-briefing/run-briefing.sh daily
   ```

---

## Issue: Pre-fetch Failed

**Symptoms**: Log shows `WARN: OSINT pre-fetch failed (non-fatal), continuing...`

**Impact**: Non-fatal. The briefing generates without OSINT data. Cyber Intel tab may have fewer items.

**Fix**:
1. Test the script standalone:
   ```bash
   python3 ~/bengio-marketplace/plugins/tech-news-briefing/scripts/fetch-osint.py --date $(date +%Y-%m-%d)
   ```
2. Common causes: NVD API rate limiting (wait and retry), network timeout, Python version issue
3. The pipeline continues regardless — only investigate if it fails repeatedly

---

## Issue: Podcast Pre-fetch Shows "No Credentials"

**Symptoms**: Log shows `WARN: Spotify credentials not found in Keychain`

**Impact**: Podcast tab uses only Apple Charts and RSS data. No Spotify episode matching.

**Fix**: Set up Spotify credentials:
```bash
security add-generic-password -s tech-news-briefing-spotify -a <SPOTIFY_CLIENT_ID> -w <SPOTIFY_CLIENT_SECRET>
```
Get credentials from the [Spotify Developer Dashboard](https://developer.spotify.com/dashboard).

---

## Issue: Email Delivery Failed

**Symptoms**: Log shows `Email delivery failed: ...`

**Impact**: Briefing is saved and pushed to GitHub, just not emailed. Non-fatal.

**Fix**:
1. Verify SMTP credentials exist:
   ```bash
   security find-generic-password -s tech-news-briefing-smtp
   security find-generic-password -s tech-news-briefing-to -w
   ```
2. Test email standalone:
   ```bash
   python3 ~/bengio-marketplace/plugins/tech-news-briefing/scripts/send-email.py ~/BPG_Tech-News/2026/week-08/2026-02-21.md
   ```
3. Common causes: App password expired (regenerate in Google Admin), wrong FROM email, network issue

---

## Issue: Git Push Failed

**Symptoms**: Log shows git push error.

**Fix**:
1. Check the remote URL is correct:
   ```bash
   git -C ~/BPG_Tech-News remote -v
   ```
   Should show `https://github.com/bengio777/BPG_Tech-News_Catalog.git`

2. Check for auth issues:
   ```bash
   gh auth status
   ```

3. Manual push:
   ```bash
   git -C ~/BPG_Tech-News push origin main
   ```

4. If there are merge conflicts (someone pushed to remote):
   ```bash
   git -C ~/BPG_Tech-News pull --rebase origin main
   git -C ~/BPG_Tech-News push origin main
   ```

---

## Issue: Stuck launchd Job

**Symptoms**: The daily job from 6 AM is still running hours later.

**Fix**:
1. Find the process:
   ```bash
   ps aux | grep run-briefing | grep -v grep
   ```
2. Kill it:
   ```bash
   kill <PID>
   ```
3. Check the log to see where it got stuck
4. Re-run manually if needed

---

## Issue: Homepage URLs in Briefing

**Symptoms**: Stories link to `https://thehackernews.com/` or other homepage URLs instead of specific article pages.

**Impact**: Reader clicks through to a homepage instead of the article. Poor experience.

**Cause**: The research skill couldn't find the direct article URL and fell back to the homepage. The v2.0.0 URL enforcement rules should prevent this, but it can still happen if the rules were bypassed.

**Fix**: This is a content quality issue, not a pipeline failure. Check that the research skill at `~/bengio-marketplace/plugins/tech-news-briefing/skills/research/SKILL.md` contains the "URL Rules" section with homepage rejection rules. The formatting skill should also have the final check that removes stories with homepage URLs.

---

## Key File Locations

| What | Where |
|------|-------|
| Pipeline script | `~/.config/tech-news-briefing/run-briefing.sh` |
| Pipeline script backup | `~/bengio-marketplace/plugins/tech-news-briefing/config/run-briefing.sh` |
| launchd plist | `~/Library/LaunchAgents/com.bengio.tech-news-briefing.plist` |
| launchd plist backup | `~/bengio-marketplace/plugins/tech-news-briefing/config/com.bengio.tech-news-briefing.plist` |
| Run logs | `~/.config/tech-news-briefing/logs/` |
| Pre-fetch data | `~/.config/tech-news-briefing/prefetch/` |
| Briefing output | `~/BPG_Tech-News/2026/week-WW/` |
| Plugin skills | `~/bengio-marketplace/plugins/tech-news-briefing/skills/` |
| Dashboard app | `~/Projects/tech-news-viewer/` |
