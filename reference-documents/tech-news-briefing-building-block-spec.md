# AI Building Block Spec: BPG Tech News Briefing

## Execution Pattern

**Recommended Pattern:** Skill-Powered Prompt (multi-cadence)

**Reasoning:**
- Tool use required (WebSearch, WebFetch, file system, git, SMTP) → rules out plain Prompt
- All steps are deterministic — scoring formula, routing tables, formatting rules — no human judgment needed → doesn't need human-in-the-loop
- 5 reusable skills with complex internal logic (research, curation, formatting, synthesis, podcasts) → Skill pattern fits
- Single expertise domain (news curation) → doesn't need Multi-Agent
- Sequential pipeline with no review gates → shell script orchestrator handles flow
- Runs autonomously via launchd at 6:00 AM MST daily → fully automated

**Three cadences share the same skill infrastructure:**

| Cadence | Trigger | Skills Used | Output |
|---------|---------|-------------|--------|
| Daily | launchd (6:00 AM MST) or manual | Research → Curation → Formatting | 3-tab briefing (AI News, Breakthroughs, Cyber Intel) |
| Weekly | Manual (typically Sundays) | Synthesis → Podcasts → Formatting | 4-tab recap (Week in AI, Breakthroughs Recap, Cyber Weekly, Podcasts) |
| Monthly | Manual (1st of month) | Synthesis → Formatting | Long-form retrospective (1,500-3,000 words) |

---

## Scenario Summary

| Field | Value |
|-------|-------|
| **Workflow Name** | BPG Tech News Briefing |
| **Description** | Research, curate, score, and deliver tech news briefings across AI, breakthroughs, and cybersecurity at daily, weekly, and monthly cadences |
| **Process Outcome** | A scored, tiered markdown briefing saved to file system, committed to GitHub, and delivered via email |
| **Trigger** | launchd daily at 6:00 AM MST; manual for weekly/monthly |
| **Type** | Automated |
| **Business Process** | Personal Intelligence & Competitive Awareness |

---

## Step-by-Step Decomposition (Daily Cadence)

| Step | Name | Autonomy Level | Building Block | Skill Candidate | Tools / Connectors | HITL Gate |
|------|------|---------------|----------------|-----------------|-------------------|-----------|
| 0 | Pre-fetch OSINT | AI-Deterministic | Python script | N/A | `fetch-osint.py` (CISA KEV API, NVD API, RSS) | None |
| 1 | Build Prompt | AI-Deterministic | Shell script | N/A | `run-briefing.sh` (file I/O, sed, awk) | None |
| 2 | Research | AI-Deterministic | **Skill: Research** (v2.0.0) | Yes | WebSearch, WebFetch | None |
| 3 | Curate | AI-Deterministic | **Skill: Curation** (v2.0.0) | Yes | File Read (previous briefing) | None |
| 4 | Format | AI-Deterministic | **Skill: Formatting** (v2.0.0) | Yes | None (text transformation) | None |
| 5 | Save to File System | AI-Deterministic | Prompt instruction | N/A | File Write | None |
| 6 | Git Commit & Push | AI-Deterministic | Shell script | N/A | git CLI | None |
| 7 | Email Delivery | AI-Deterministic | Python script | N/A | `send-email.py` (Google Workspace SMTP) | None |

## Autonomy Spectrum Summary

```
|--Deterministic----------------------------------------------|
   Steps 0, 1, 2, 3, 4, 5, 6, 7
```

- **8 of 8 steps are deterministic** — every step follows fixed rules, scoring formulas, templates, and routing tables.
- **0 autonomous steps** — no LLM judgment calls. Research queries are pre-defined. Scoring uses a weighted formula. Routing uses threshold-based rules.
- **0 human-in-the-loop gates** — the pipeline runs end-to-end without intervention.

---

## Skill Candidates

### Skill 1: Research (v2.0.0)

**Purpose:** Systematically query 23 sources across 3 domains to gather raw news stories.

**Inputs:**

| Input | Required | Description |
|-------|----------|-------------|
| Current date | Yes | Determines search window and file naming |
| OSINT pre-fetch data | No | `~/.config/tech-news-briefing/prefetch/osint-YYYY-MM-DD.json` — CISA KEV entries, NVD CVEs (CVSS ≥8.0), RSS feeds |

**Outputs:**

| Output | Format | Description |
|--------|--------|-------------|
| Raw story list | Structured data | `{title, url, source, summary, date, category}` per story, tagged `ai-news`, `breakthroughs`, or `cyber-intel` |

**Source Groups (23 total):**

| Group | Category Tag | Sources (10) |
|-------|-------------|-------------|
| AI News | `ai-news` | Anthropic Blog, OpenAI Blog, Ben's Bites, Hacker News (AI), Ars Technica, The Verge, TechCrunch, Microsoft AI, Meta AI, Twitter/X (AI) |
| Breakthroughs | `breakthroughs` | arXiv, Product Hunt, GitHub Trending, Reddit AI, Dev.to |
| Cybersecurity | `cyber-intel` | BleepingComputer, Dark Reading, SecurityWeek, Krebs on Security, The Record, Hacker News (Security), r/netsec, Twitter/X (InfoSec) |

**URL Validation Rules:**
- REJECT all homepage/section URLs (e.g., `https://thehackernews.com/`)
- ONLY accept direct article page URLs
- If article URL not found: search exact headline → search without site restriction → DROP story

**Failure Modes:**

| Failure | Handling |
|---------|----------|
| Source unreachable (timeout, 403) | Log unreachable source, continue with remaining sources |
| WebFetch fails | Fall back to WebSearch for that source |
| Zero results for a category | Broaden query (remove date constraint), log if still zero |
| OSINT pre-fetch missing | Non-fatal — research proceeds without structured OSINT data |

---

### Skill 2: Curation (v2.0.0)

**Purpose:** Deduplicate, score, filter, and route raw stories into a 3-tab tiered briefing.

**Inputs:**

| Input | Required | Description |
|-------|----------|-------------|
| Raw story list | Yes | Output from Research skill |
| Previous day's briefing | No | For deduplication and update detection |

**Outputs:**

| Output | Format | Description |
|--------|--------|-------------|
| Routed story list | Structured data | Stories assigned to tab + tier/section, ordered by composite score |

**Processing Pipeline:**

1. **Deduplicate** — Same event = duplicate. Keep highest-authority source. Combine summaries. Stories in 3+ sources get virality boost.

   Authority hierarchy: Official blog > Major tech pub > Security pub > Newsletter > Community source

2. **Compare to previous briefing** — Remove exact repeats. Keep stories with significant updates (mark "**Update:**"). Remove follow-ups with no new info.

3. **Quality filter** — Reject: stale (>48h, >72h for breakthroughs), paywalled, clickbait, listicles, promotional, trivial.

4. **Score** — Composite formula:

   ```
   Composite = (Impact × 0.30) + (Novelty × 0.25) + (Relevance × 0.25) + (Authority × 0.20)
   ```

   All dimensions 0-10 scale. Virality boost: +0.5 if 3+ independent sources (capped at 10.0).

   | Dimension | Weight | 9-10 Example | 3-4 Example |
   |-----------|--------|-------------|-------------|
   | Impact | 0.30 | Major product launch, critical CVE (CVSS 9.0+) | Minor feature, niche tool |
   | Novelty | 0.25 | First report, breaking news | Rehash with minor additions |
   | Relevance | 0.25 | Core AI dev, critical infrastructure vuln | Adjacent tech, consumer product |
   | Authority | 0.20 | Official blog, CISA advisory, NVD | Unknown blog, unverified social |

5. **Route to tabs:**

   | Tab | Category | Routing Rule |
   |-----|----------|-------------|
   | AI News | `ai-news` | Must-Read (≥7.5), Worth Knowing (5.0-7.4), On the Radar (3.0-4.9), Drop (<3.0) |
   | Breakthroughs | `breakthroughs` | Include if composite ≥8.0 OR 3+ sources. Flat list by score. |
   | Cyber Intel | `cyber-intel` | 4 sections: AI x Cyber, Breaches & Incidents, Active Threats, OSINT Signal |

6. **Order** — Within each tier/section by composite score (highest first). Tie-breaker: more recent > higher authority > broader impact.

**Failure Modes:**

| Failure | Handling |
|---------|----------|
| No previous briefing available | Skip comparison step |
| Ambiguous deduplication | Err on keeping distinct stories |
| All stories filtered out | Output empty briefing fallback message |

---

### Skill 3: Formatting (v2.0.0)

**Purpose:** Transform curated, routed stories into a tabbed markdown briefing document.

**Inputs:**

| Input | Required | Description |
|-------|----------|-------------|
| Routed story list | Yes | Output from Curation skill |
| Date/time | Yes | For header |
| Source list | Yes | For footer |
| Unreachable sources | No | For error note |

**Outputs:**

| Output | Format | Description |
|--------|--------|-------------|
| Markdown briefing | `.md` file | Header + 3 tabbed sections + footer, parseable by dashboard |

**Per-Item Format (strict):**
```
- **[Story Title](url)** — 1-2 sentence summary. *Source: SourceName*
```

**Tab Markers (exact):**
```
<!-- tab: AI News -->
<!-- tab: AI Breakthroughs & Viral -->
<!-- tab: Cyber Intel -->
```

**Rules:**
- Active voice summaries with specific metrics (CVE IDs, CVSS scores, dollar amounts)
- No emoji anywhere
- Omit empty tiers/sections/tabs entirely
- No homepage URLs — final quality check before output

**Failure Modes:**

| Failure | Handling |
|---------|----------|
| Story without valid URL | Remove story |
| Summary exceeds 2 sentences | Truncate |
| Zero stories total | Output fallback: "No significant tech news stories found for today." |

---

### Skill 4: Synthesis (v1.0.0)

**Purpose:** Analyze a week or month of daily briefings to extract persistent stories, emerging themes, and quantifiable metrics.

**Used by:** Weekly and monthly cadences only.

**Inputs:** All daily briefing `.md` files from the target period.

**Outputs:** Persistent stories (ranked by day count and tier), emerging themes (named with analysis), story arcs, and "By the Numbers" metrics.

---

### Skill 5: Podcasts (v1.0.0)

**Purpose:** Match podcast episodes to the week's news stories, flag charting shows, and recommend a discovery pick.

**Used by:** Weekly cadence only.

**Inputs:** Pre-fetch podcast data (`podcasts-YYYY-MM-DD.json`) + week's stories from Synthesis.

**Outputs:** Three sections — News-Connected Episodes (3-8), Trending in Tech Podcasts (2-4), Discovery Pick (1).

---

## Step Sequence and Dependencies

### Daily Pipeline
```
Step 0: Pre-fetch OSINT (Python — non-fatal)
    │
    ▼
Step 1: Build self-contained prompt (Shell — inline command + skills + template)
    │
    ▼
Step 2: Research (Skill — 23 sources, 3 categories)
    │
    ▼
Step 3: Curate (Skill — deduplicate, score, filter, route)
    │
    ▼
Step 4: Format (Skill — tabbed markdown with strict rules)
    │
    ▼
Step 5: Save to file system (BPG_Tech-News/YYYY/week-WW/YYYY-MM-DD.md)
    │
    ▼
Step 6: Git commit & push (non-fatal)
    │
    ▼
Step 7: Email delivery (non-fatal)
```

### Dependency Map

| Step | Depends On | Produces |
|------|-----------|----------|
| 0: Pre-fetch | System trigger (launchd or manual) | OSINT JSON file |
| 1: Build Prompt | Step 0 output (optional) + skill files + template | Self-contained prompt file |
| 2: Research | Step 1 prompt | Raw story list (title, url, source, summary, date, category) |
| 3: Curate | Step 2 output + previous briefing (optional) | Routed, scored, tiered story list |
| 4: Format | Step 3 output + date/time | Markdown briefing document |
| 5: Save | Step 4 output | Persisted .md file |
| 6: Git Push | Step 5 output | Remote commit |
| 7: Email | Step 4 output | Delivered briefing in inbox |

**All steps are sequential.** No parallelism. Steps 6-7 are non-fatal — the briefing is saved (Step 5) before distribution is attempted.

---

## Orchestration Infrastructure

| Component | Location | Purpose |
|-----------|----------|---------|
| `run-briefing.sh` | `~/.config/tech-news-briefing/run-briefing.sh` | Shell orchestrator (205 lines) — handles pre-fetch, prompt assembly, Claude invocation, logging |
| `fetch-osint.py` | `bengio-marketplace/plugins/tech-news-briefing/scripts/` | Pulls CISA KEV (last 48h), NVD CVEs (CVSS ≥8.0, last 48h), RSS feeds |
| `fetch-podcasts.py` | `bengio-marketplace/plugins/tech-news-briefing/scripts/` | Spotify episodes + Apple Charts top 25 |
| `send-email.py` | `bengio-marketplace/plugins/tech-news-briefing/scripts/` | Google Workspace SMTP delivery (HTML + plain text) |
| launchd plist | `~/Library/LaunchAgents/com.bengio.tech-news-briefing.plist` | Fires at 6:00 AM MST daily |
| Templates | `bengio-marketplace/plugins/tech-news-briefing/templates/` | `daily.md`, `weekly.md`, `monthly.md` |

---

## Prerequisites

- Mac with launchd (awake or wakes before 6 AM)
- `claude` CLI installed and authenticated
- Anthropic API key with sufficient quota
- Google Workspace SMTP credentials in macOS Keychain
- Git configured with push access to `bengio777/BPG_Tech-News_Catalog`
- Python 3 with `requests` library (for pre-fetch scripts)
- Internet connection for web searches and email

---

## Context Inventory

| Artifact | Type | Used By | Status | Key Contents |
|----------|------|---------|--------|-------------|
| Research Skill (v2.0.0) | Skill | Step 2 | Complete | 23 source definitions, search strategy, URL validation rules |
| Curation Skill (v2.0.0) | Skill | Step 3 | Complete | Dedup rules, quality filters, scoring formula, tab routing thresholds |
| Formatting Skill (v2.0.0) | Skill | Step 4 | Complete | Markdown template, tab markers, summary writing rules, quality checks |
| Synthesis Skill (v1.0.0) | Skill | Weekly/Monthly | Complete | Theme detection, persistence ranking, story arc tracking |
| Podcasts Skill (v1.0.0) | Skill | Weekly | Complete | Episode matching, chart flagging, discovery pick criteria |
| Daily Template | Template | Step 4 | Complete | Markdown structure with tab markers |
| Weekly Template | Template | Weekly formatting | Complete | 4-tab structure with podcasts |
| Monthly Template | Template | Monthly formatting | Complete | Long-form retrospective structure |
| OSINT Pre-fetch Config | Script | Step 0 | Complete | CISA KEV API, NVD API, RSS feed URLs |
| Daily Briefing Pipeline SOP | SOP | Operations | Complete | 6-step procedure, verification steps, troubleshooting |

---

## Tools and Connectors

| Tool | Purpose | Used By Step | Integration |
|------|---------|-------------|-------------|
| **WebSearch** | Query news sources | Step 2 (Research) | Built into Claude |
| **WebFetch** | Retrieve specific URLs (blogs, feeds) | Step 2 (Research) | Built into Claude |
| **File Read** | Read previous briefing for dedup | Step 3 (Curation) | Built into Claude |
| **File Write** | Save briefing to file system | Step 5 (Save) | Built into Claude |
| **git CLI** | Commit and push to GitHub | Step 6 (Git Push) | Configured in shell |
| **CISA KEV API** | Active vulnerability data | Step 0 (Pre-fetch) | Python `requests` |
| **NVD API** | CVE data (CVSS ≥8.0) | Step 0 (Pre-fetch) | Python `requests` |
| **Google Workspace SMTP** | Email delivery | Step 7 (Email) | Python `smtplib` + Keychain |
| **Spotify API** | Podcast episode data | Pre-fetch (weekly) | Python `requests` |
| **Apple Podcasts Charts** | Top 25 tech podcasts | Pre-fetch (weekly) | Python `requests` |

---

## Where to Run

| Cadence | Trigger | Command |
|---------|---------|---------|
| Daily | launchd (6:00 AM MST) | Automatic — `com.bengio.tech-news-briefing` plist |
| Daily (manual) | Terminal | `bash ~/.config/tech-news-briefing/run-briefing.sh daily` |
| Weekly | Terminal | `bash ~/.config/tech-news-briefing/run-briefing.sh weekly` |
| Monthly | Terminal | `bash ~/.config/tech-news-briefing/run-briefing.sh monthly` |

---

## Deterministic Properties

- **Same date → same OSINT data** — CISA KEV and NVD APIs return deterministic results for a given date window
- **Same stories + scoring formula → same curation** — composite scoring uses fixed weights and thresholds
- **Same template → same markdown structure** — tab markers, heading hierarchy, and item format are invariant
- **Error isolation** — pre-fetch non-fatal, git push non-fatal, email non-fatal — the briefing is always produced and saved locally before any distribution step
