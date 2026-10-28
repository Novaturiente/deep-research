---
name: research-add-items
description: Add items (research objects) to an existing research outline.yaml, from the user or from a web search.
allowed-tools: read write edit find web_search agent ask_user
---

# Research Add Items - Supplement Research Objects

## Trigger
`/skill:research-add-items`

## Workflow

### Step 1: Auto-locate Outline
Find `*/outline.yaml` file in current working directory, auto-read.

### Step 2: Get Supplement Sources in Parallel
Simultaneously (one `ask_user` call with two questions):
- **A. Ask user**: What items to supplement? Any specific names?
- **B. Ask if Web Search needed**: Spawn a `web-researcher` agent to search for more items? If yes, give it the topic, the existing items list and `## Strategy Modules` `${CLAUDE_SKILL_DIR}/../research/modules`, and ask it to return new items with a one-line reason and a source each (no files). Get its full report with `agent read {id}`.

### Step 3: Merge and Update
- Append new items to outline.yaml
- Display to user for confirmation
- Avoid duplicates
- Save updated outline

## Output
Updated `{topic}/outline.yaml` file (in-place modification)
