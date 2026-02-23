# BPG Tech News Briefing — Requirements Summary

**Assignment:** Build a Deterministic Automated Workflow | **Module:** Design Prompt Workflows

---

## Lesson Objectives

| Objective | Status | How |
|---|---|---|
| Multi-step deterministic workflow with consistent outputs | **Met** | 8-step pipeline, all 8 steps deterministic. Same date → same OSINT data, same scoring formula → same curation, same template → same markdown structure. Composite scoring uses fixed weights: `(Impact×0.30) + (Novelty×0.25) + (Relevance×0.25) + (Authority×0.20)`. Tier routing uses fixed thresholds (Must-Read ≥7.5, Worth Knowing 5.0-7.4, On the Radar 3.0-4.9). |
| Sequenced steps with clear input/output definitions | **Met** | Full dependency map: Pre-fetch → Build Prompt → Research (raw stories) → Curate (scored, routed stories) → Format (markdown briefing) → Save → Git Push → Email. Each step's output is the next step's input. |
| Error handling and edge case logic | **Met** | Non-fatal isolation at 3 levels: pre-fetch failure doesn't stop research, git push failure doesn't prevent email, email failure doesn't lose the briefing (already saved locally). Per-skill failure modes: URL validation rejects homepage links, WebFetch failures fall back to WebSearch, empty categories get omitted, zero-story briefings get fallback message. |
| Documented SOP saved to registry | **Met** | 4 SOPs (daily, weekly, monthly pipelines + troubleshooting runbook) in `BPG_Tech-News/docs/sops/`. Registered in Notion AI Building Blocks. Version-controlled on GitHub. |

---

## Deliverables

| # | Deliverable | File | Status |
|---|---|---|---|
| 1 | AI Building Block Spec | `reference-documents/tech-news-briefing-building-block-spec.md` (360 lines) — Execution pattern: Skill-Powered Prompt (multi-cadence). 8 steps, all deterministic, zero human gates. 5 skills specified with inputs, outputs, scoring formulas, routing rules, and failure modes. Orchestration infrastructure documented (shell script, launchd, 3 Python scripts). | **Complete** |
| 2 | Baseline Workflow Prompt + Skills | 5 skills in `bengio-marketplace/plugins/tech-news-briefing/skills/`: Research (v2.0.0, 173 lines — 23 sources, URL validation), Curation (v2.0.0, 163 lines — composite scoring, dedup, tier routing), Formatting (v2.0.0, 176 lines — markdown template, summary rules), Synthesis (v1.0.0, 81 lines — theme detection, persistence ranking), Podcasts (v1.0.0, 84 lines — episode matching, chart flagging). 3 cadence commands: `briefing.md`, `briefing-weekly.md`, `briefing-monthly.md`. | **Complete** |
| 3 | Test run output | **In daily production.** launchd fires at 6:00 AM MST. Logs at `~/.config/tech-news-briefing/logs/` show automated runs with exit code 0. Briefing output at `BPG_Tech-News/2026/` — 2 daily briefings + 1 weekly recap across Feb 21-23, 2026. Pipeline iterated through v2 multi-cadence restructure (documented in build journal). Email delivery to 3 recipients confirmed in logs. | **Complete** |

---

## Review Criteria

| Criterion | Evidence |
|---|---|
| Execution pattern makes sense, steps classified, blocks mapped | Skill-Powered Prompt — 5 reusable skills with complex internal logic, all deterministic, orchestrated by a shell script with launchd scheduling. Each of the 8 steps has autonomy level (all AI-Deterministic), building block assignment, tools/connectors, and skill candidate mapped. |
| Prompt instructions clear, context requirements listed | Each skill has explicit rules: Research has 23 named sources with search strategy per group and URL rejection criteria. Curation has a 4-dimension weighted scoring formula with example scores at each level. Formatting has exact markdown templates, tab markers, and a pre-formatting quality checklist. |
| Skills specific enough for AI to execute without ambiguity | Scoring formula uses fixed weights and thresholds. Deduplication uses a 5-level authority hierarchy. Quality filters list 6 specific rejection criteria (stale, paywalled, clickbait, listicle, promotional, trivial). Formatting rules specify exact markdown syntax, tab markers, and per-item format strings. |
| Test run: run on real scenario, output usable | In daily production since Feb 21. Most recent run (Feb 23) produced 12 stories, committed to GitHub, emailed to 3 recipients, exited cleanly. Weekly recap also produced. System iterated from v1 (single-cadence) to v2 (multi-cadence with podcasts, synthesis, and email delivery). |

---

## Deterministic Workflow Fit

The assignment calls for **"fixed steps, predictable inputs, consistent outputs"** and lists **recurring reports** and **data processing pipelines** as good candidates.

BPG Tech News Briefing is both — a recurring daily report powered by a data processing pipeline. Every step follows fixed rules: 23 pre-defined sources, a weighted composite scoring formula, threshold-based tier routing, and a strict markdown template. The pipeline runs autonomously via launchd at 6:00 AM with zero human intervention. The only variability is the news itself — the processing is entirely deterministic.

---

## Bonus

| Bonus Item | Status | Evidence |
|---|---|---|
| Register building blocks in AI Operations Registry | **Done** | Skills registered in Notion AI Building Blocks database |
| Link back to workflow entry | **Done** | Workflow Definition at `reference-documents/claude-news-briefing-definition.md`, cross-referenced in `Projects/workflow-definitions/` |
| Commit `.md` files to GitHub | **Done** | All artifacts at `github.com/bengio777/BPG_Tech-News_Catalog` (public) |

---

## Links

| Resource | URL |
|---|---|
| GitHub Repo (public) | https://github.com/bengio777/BPG_Tech-News_Catalog |
| Building Block Spec | https://github.com/bengio777/BPG_Tech-News_Catalog/blob/main/reference-documents/tech-news-briefing-building-block-spec.md |
| Workflow Definition | https://github.com/bengio777/BPG_Tech-News_Catalog/blob/main/reference-documents/claude-news-briefing-definition.md |
| Daily Briefing Example | https://github.com/bengio777/BPG_Tech-News_Catalog/tree/main/2026 |
| SOPs | https://github.com/bengio777/BPG_Tech-News_Catalog/tree/main/docs/sops |
