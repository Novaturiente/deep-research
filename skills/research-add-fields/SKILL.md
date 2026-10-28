---
name: research-add-fields
description: Add field definitions to an existing research fields.yaml, from the user or from a web search.
allowed-tools: read write edit find web_search agent ask_user
---

# Research Add Fields - Supplement Research Fields

## Trigger
`/skill:research-add-fields`

## Workflow

### Step 1: Auto-locate Fields File
Find `*/fields.yaml` file in current working directory, auto-read existing fields definitions.

### Step 2: Get Supplement Source
Use `ask_user` to let the user choose:
- **A. User direct input**: User provides field names and descriptions
- **B. Web Search**: Spawn a `web-researcher` agent to search common fields in this domain. Give it the topic, the existing fields and `## Strategy Modules` `${CLAUDE_SKILL_DIR}/../research/modules`, and ask it to return suggested fields with a description and a source each (no files). Get its full report with `agent read {id}`.

### Step 3: Display and Confirm
- Display suggested new fields list
- User confirms which fields to add (`ask_user` with `multi: true`)
- User specifies field category and detail_level

### Step 4: Save Update
Append confirmed fields to fields.yaml, save file. Keep the `fields: {<category>: [{name, description, detail_level}]}` shape; `validate_json.py` rejects any other.

## Output
Updated `{topic}/fields.yaml` file (in-place modification, requires user confirmation)
