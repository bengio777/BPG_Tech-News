# Workflow Definition: Claude & Anthropic News Research Briefing

## Overview

| Field | Value |
|-------|-------|
| **Workflow Name** | Claude & Anthropic News Research Briefing |
| **Description** | Research and compile the latest Claude Code, Anthropic, and Claude Cowork news into a curated, linked markdown briefing |
| **Trigger** | On-demand (daily or as-needed) |
| **Outcome** | A saved markdown file with a bulleted list of top stories and links from the last 3 days |
| **Type** | Research & Curation |
| **Business Objective** | Stay current on Claude ecosystem developments without manual searching |
| **Current Owner** | James Gray |

## Steps

### Step 1: Search the web for recent news
- **Action:** Search for top news stories about Claude Code, Anthropic, and Claude Cowork from the last 3 days
- **Sources to check:**
  - claude.ai (product updates, announcements)
  - anthropic.com (blog, press releases, research)
  - Anthropic X.com accounts (@AnthropicAI, @alexalbert__)
  - claude.com/blog 
  - General web search for coverage and commentary
- **Input:** Search queries targeting Claude Code, Anthropic, Claude Cowork
- **Output:** Raw list of news stories, announcements, and updates

### Step 2: Curate a bulleted list with links
- **Action:** Filter and organize the most relevant stories into a clean bulleted list
- **Each item includes:** Story title/headline, brief summary, and source link
- **Input:** Raw search results from Step 1
- **Output:** Curated, formatted bulleted list

### Step 3: Save to markdown file
- **Action:** Write the curated list to a markdown file in the `outputs/` directory
- **File naming:** `outputs/claude-news-YYYY-MM-DD.md`
- **Input:** Curated bulleted list from Step 2
- **Output:** Saved `.md` file ready for reference
