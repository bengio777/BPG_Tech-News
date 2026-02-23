# BPG Tech News Briefing — Assignment Requirements Mapping

**Assignment:** Build a Deterministic Automated Workflow
**Module:** Design Prompt Workflows
**Project:** BPG Tech News Briefing

---

## Lesson Objectives → Evidence

### 1. "Build a multi-step deterministic workflow using prompt chaining that produces consistent outputs from the same inputs"

**Met.** The workflow is an 8-step pipeline where all 8 steps are deterministic. The Research skill queries 23 pre-defined sources. The Curation skill scores stories using a fixed composite formula: `(Impact × 0.30) + (Novelty × 0.25) + (Relevance × 0.25) + (Authority × 0.20)`. The Formatting skill applies a strict markdown template with exact tab markers, item format, and summary rules. Same date input with same available sources produces the same briefing.

**Evidence:**
- `reference-documents/tech-news-briefing-building-block-spec.md` — Autonomy Spectrum shows 8/8 steps deterministic, 0 autonomous, 0 human-in-the-loop
- `skills/curation/SKILL.md` — Composite scoring formula with fixed weights, tier thresholds (Must-Read ≥7.5, Worth Knowing 5.0-7.4, On the Radar 3.0-4.9)
- `skills/formatting/SKILL.md` — Exact markdown template, tab markers, per-item format string

### 2. "Sequence workflow steps with clear input/output definitions between each stage"

**Met.** Every step has defined inputs and outputs. The dependency chain is explicit: Pre-fetch → Build Prompt → Research → Curate → Format → Save → Git Push → Email.

**Evidence:**
- `reference-documents/tech-news-briefing-building-block-spec.md` — Dependency Map table:

  | Step | Depends On | Produces |
  |------|-----------|----------|
  | 0: Pre-fetch | System trigger | OSINT JSON file |
  | 1: Build Prompt | Step 0 output + skill files + template | Self-contained prompt file |
  | 2: Research | Step 1 prompt | Raw story list (title, url, source, summary, date, category) |
  | 3: Curate | Step 2 output + previous briefing | Routed, scored, tiered story list |
  | 4: Format | Step 3 output + date/time | Markdown briefing document |
  | 5: Save | Step 4 output | Persisted .md file |
  | 6: Git Push | Step 5 output | Remote commit |
  | 7: Email | Step 4 output | Delivered briefing in inbox |

- `~/.config/tech-news-briefing/run-briefing.sh` — 205-line orchestrator that sequences pre-fetch → prompt assembly → Claude invocation

### 3. "Implement error handling and edge case logic to ensure reliable execution"

**Met.** Error handling is implemented at three levels: orchestration (non-fatal steps), per-skill (failure modes with recovery), and per-story (URL validation, quality filters).

**Evidence:**
- `run-briefing.sh` — Pre-fetch uses `|| { echo "WARN: ... (non-fatal), continuing..."; }` pattern. Git push and email are attempted after the briefing is already saved locally.
- `skills/research/SKILL.md` — 4 failure modes: source unreachable (continue with remaining), WebFetch fails (fall back to WebSearch), zero results (broaden query), OSINT pre-fetch missing (non-fatal)
- `skills/curation/SKILL.md` — 3 failure modes: no previous briefing (skip comparison), ambiguous dedup (keep distinct), all filtered (fallback message)
- `skills/formatting/SKILL.md` — 4 failure modes: invalid URL (remove story), long summary (truncate), empty tab (omit), zero stories (fallback message)
- `~/.config/tech-news-briefing/logs/2026-02-23-daily.log` — Real-world error handling demonstrated: NVD API timed out with `ConnectionResetError`, handled as non-fatal, pipeline continued and completed successfully (exit code 0)

### 4. "Document the workflow SOP and save it to your registry"

**Met.** Four SOPs cover all cadences plus troubleshooting. Registered in AI Operations Registry and version-controlled on GitHub.

**Evidence:**
- `docs/sops/daily-briefing-pipeline.md` — 6-step automated procedure with verification steps
- `docs/sops/weekly-briefing-pipeline.md` — Weekly cadence with synthesis + podcasts
- `docs/sops/monthly-briefing-pipeline.md` — Monthly retrospective cadence
- `docs/sops/troubleshooting-runbook.md` — Diagnostic flowchart for common failures
- Registered in Notion AI Building Blocks database
- Version-controlled at `github.com/bengio777/BPG_Tech-News_Catalog` (public)

---

## Assignment Instructions → Evidence

### Part 1: Design ("Upload your Workflow Definition and let the AI recommend an execution pattern, classify each step, and map building blocks")

**Completed.** Output saved as `reference-documents/tech-news-briefing-building-block-spec.md`.

| Design Requirement | Where It Appears |
|---|---|
| Execution pattern recommendation | Building Block Spec — "Recommended Pattern: Skill-Powered Prompt (multi-cadence)" with reasoning: 5 reusable skills, all deterministic, single domain, shell script orchestration |
| Each step classified | Building Block Spec — Step-by-Step Decomposition table with Autonomy Level column (all AI-Deterministic), plus Autonomy Spectrum Summary showing 8/8 deterministic |
| Building blocks mapped | Building Block Spec — Each step maps to specific building blocks (Skill, Python script, Shell script, Prompt instruction), tools/connectors (WebSearch, WebFetch, git CLI, SMTP, CISA API, NVD API), and 5 skill candidates with full input/output/scoring/failure specs |

### Part 2: Construct ("Follow the build path for your recommended execution pattern... Save the prompt output. If your pattern is Skill-Powered Prompt, also save any skill files you built.")

**Completed.** 5 skills + 3 cadence commands + 3 templates built and deployed.

| Construct Requirement | Where It Appears |
|---|---|
| Baseline Workflow Prompt | 3 cadence commands at `plugins/tech-news-briefing/commands/`: `briefing.md` (daily), `briefing-weekly.md`, `briefing-monthly.md`. Each command references specific skills and templates for its cadence. |
| Skill files | 5 skills at `plugins/tech-news-briefing/skills/`: Research (v2.0.0, 173 lines), Curation (v2.0.0, 163 lines), Formatting (v2.0.0, 176 lines), Synthesis (v1.0.0, 81 lines), Podcasts (v1.0.0, 84 lines) |
| Supporting infrastructure | `run-briefing.sh` (205 lines — orchestrator), `fetch-osint.py` (OSINT pre-fetch), `fetch-podcasts.py` (podcast pre-fetch), `send-email.py` (email delivery), 3 templates (`daily.md`, `weekly.md`, `monthly.md`) |

### Part 3: Run ("Run your workflow on a real scenario, evaluate the output, and iterate. Run at least twice.")

**Completed.** The workflow is in automated daily production via launchd.

| Run Requirement | Evidence |
|---|---|
| Run on a real scenario | Pipeline runs daily at 6:00 AM MST via launchd. Briefings produced at `BPG_Tech-News/2026/`: Feb 21 (daily), Feb 22 (weekly recap), Feb 23 (daily). Committed to GitHub and emailed to 3 recipients. |
| Run at least twice | The system has run on multiple consecutive days. Logs at `~/.config/tech-news-briefing/logs/` show successful runs on Feb 22 and Feb 23 with exit code 0. |
| Output is usable without heavy editing | Briefings are consumed directly — no manual editing. They render in a custom-built web dashboard at `localhost:3000`, arrive formatted in email, and are archived in GitHub. |
| Iterate and verify adjustments | The system was iterated from v1 (single-cadence, Claude-only) to v2 (multi-cadence with weekly/monthly synthesis, podcast integration, OSINT pre-fetch, email delivery, and web dashboard). Build journal documents the restructure at `docs/build-journal/2026-02-22-v2-multi-cadence-restructure.md`. |

---

## Review and Refine Criteria → Evidence

### "AI Building Block Spec: Execution pattern makes sense, each step is classified, building blocks are mapped"

| Criterion | Status | Evidence |
|---|---|---|
| Execution pattern makes sense | **Yes** | Skill-Powered Prompt — 5 reusable skills with complex scoring/routing logic, all deterministic, orchestrated by shell script + launchd. No human gates, no autonomous judgment. (`tech-news-briefing-building-block-spec.md`, Execution Pattern section) |
| Each step is classified | **Yes** | 8 steps, all classified as AI-Deterministic. Autonomy spectrum: 8/8 deterministic, 0 semi-autonomous, 0 autonomous, 0 human. (`tech-news-briefing-building-block-spec.md`, Step-by-Step Decomposition table) |
| Building blocks are mapped | **Yes** | Each step maps to: building block type (Skill, Python script, Shell script), specific tools/connectors (WebSearch, WebFetch, CISA API, NVD API, git CLI, SMTP), and skill candidates with full specs. Orchestration infrastructure documented (shell script, launchd plist, 3 Python scripts, 3 templates). (`tech-news-briefing-building-block-spec.md`, Skill Candidates + Orchestration Infrastructure sections) |

### "Baseline Workflow Prompt: Instructions are clear, context requirements are listed, output is consistent across different inputs"

| Criterion | Status | Evidence |
|---|---|---|
| Instructions are clear | **Yes** | Research skill names all 23 sources with search strategy per group. Curation skill specifies exact scoring formula with weights, example scores at each level, and tier thresholds. Formatting skill provides exact markdown template, tab markers, per-item format string, and pre-formatting quality checklist. |
| Context requirements are listed | **Yes** | Building Block Spec — Context Inventory table lists all 10 artifacts (5 skills, 3 templates, OSINT config, daily SOP) with type, used-by step, and status. |
| Output is consistent across inputs | **Yes** | Fixed scoring formula + fixed thresholds + fixed template = consistent output structure. Same stories always score the same, route to the same tiers, and format identically. Tab markers, heading hierarchy, and item format are invariant. |

### "Skills (if applicable): Instructions are specific enough for the AI to execute without ambiguity"

| Criterion | Status | Evidence |
|---|---|---|
| Specific enough for AI execution | **Yes** | Research: 23 named sources in 3 groups with specific search strategies, URL rejection rules (no homepage URLs ever), fallback chains (WebFetch → WebSearch → broaden → drop). Curation: 4-dimension scoring with exact weights (0.30/0.25/0.25/0.20), score examples at 9-10 and 3-4 levels for each dimension, virality boost rule (+0.5 for 3+ sources), 6 quality filter rejection criteria, 5-level dedup authority hierarchy. Formatting: exact tab marker syntax, exact per-item format string, active voice rule, no-emoji rule, empty-section omission rule, pre-formatting quality checklist with 8 items. |

### "Test run: You've run the workflow on at least one real scenario and the output is usable"

| Criterion | Status | Evidence |
|---|---|---|
| Run on real scenario | **Yes** | Automated daily production. launchd fires at 6:00 AM MST. Logs show successful runs on Feb 22 and Feb 23 (exit code 0). Briefings at `BPG_Tech-News/2026/week-08/` and `week-09/`. |
| Output is usable | **Yes** | Briefings consumed directly with zero manual editing: rendered in web dashboard (`localhost:3000`), delivered via email (HTML + plain text), archived in GitHub. Weekly recap also produced. System iterated from v1 to v2 with multi-cadence support. |

---

## Deliverables Checklist

| # | Deliverable | Required Format | File | Status |
|---|---|---|---|---|
| 1 | AI Building Block Spec | `[name]-building-block-spec.md` | `reference-documents/tech-news-briefing-building-block-spec.md` (360 lines) | **Complete** |
| 2 | Baseline Workflow Prompt | `[name]-prompt.md` + skill files | 3 cadence commands + 5 skills (Research 173L, Curation 163L, Formatting 176L, Synthesis 81L, Podcasts 84L) + 3 templates | **Complete** |
| 3 | Test run output | Evidence of real execution | Automated daily production: launchd-triggered, 3 briefings produced (2 daily + 1 weekly), logs with exit code 0, email delivery confirmed, GitHub commits | **Complete** |

---

## Bonus (Optional) → Evidence

| Bonus Item | Status | Evidence |
|---|---|---|
| Register building blocks in AI Operations Registry | **Done** | Skills registered in Notion AI Building Blocks database |
| Link back to workflow entry | **Done** | Workflow Definition at `reference-documents/claude-news-briefing-definition.md` |
| Commit `.md` files to GitHub | **Done** | All artifacts at `github.com/bengio777/BPG_Tech-News_Catalog` (public) |

---

## Deterministic Workflow Fit

The assignment specifies: *"Choose one that has predictable, fixed steps — a deterministic workflow where the AI follows rules and templates rather than making open-ended judgments."*

**Good candidates listed:** **recurring reports**, prospecting or research workflows, template-driven content, **data extraction and formatting**, intake processing.

BPG Tech News Briefing is a **recurring report** powered by a **data extraction and formatting** pipeline — two of the exact use cases the assignment calls out. The pipeline follows fixed rules at every stage: pre-defined source lists, a weighted composite scoring formula, threshold-based tier routing, a 5-level deduplication authority hierarchy, 6 quality filter rejection criteria, and a strict markdown template. It runs autonomously via launchd at 6:00 AM daily with zero human intervention. The only variability is the news itself — the processing is entirely deterministic.

---

## File Inventory

| File | Location | Purpose |
|---|---|---|
| `tech-news-briefing-building-block-spec.md` | `reference-documents/` | Deliverable 1: AI Building Block Spec |
| `claude-news-briefing-definition.md` | `reference-documents/` | Supporting: Workflow Definition |
| `research/SKILL.md` | `plugins/.../skills/` | Deliverable 2: Research skill (v2.0.0) |
| `curation/SKILL.md` | `plugins/.../skills/` | Deliverable 2: Curation skill (v2.0.0) |
| `formatting/SKILL.md` | `plugins/.../skills/` | Deliverable 2: Formatting skill (v2.0.0) |
| `synthesis/SKILL.md` | `plugins/.../skills/` | Deliverable 2: Synthesis skill (v1.0.0) |
| `podcasts/SKILL.md` | `plugins/.../skills/` | Deliverable 2: Podcasts skill (v1.0.0) |
| `run-briefing.sh` | `~/.config/tech-news-briefing/` | Orchestrator (205 lines) |
| `daily-briefing-pipeline.md` | `docs/sops/` | SOP: Daily cadence |
| `weekly-briefing-pipeline.md` | `docs/sops/` | SOP: Weekly cadence |
| `monthly-briefing-pipeline.md` | `docs/sops/` | SOP: Monthly cadence |
| `troubleshooting-runbook.md` | `docs/sops/` | SOP: Diagnostics |
| `2026/week-08/2026-02-21.md` | `BPG_Tech-News/` | Test run: Daily briefing |
| `2026/week-08/week-08-recap.md` | `BPG_Tech-News/` | Test run: Weekly recap |
| `2026/week-09/2026-02-23.md` | `BPG_Tech-News/` | Test run: Daily briefing |
