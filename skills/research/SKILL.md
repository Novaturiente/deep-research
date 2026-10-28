---
name: research
description: Conduct preliminary research on a topic and generate a research outline (items to study plus fields to collect for each). For academic surveys, benchmark reviews, technology selection, competitor analysis. First step of the research-* workflow.
argument-hint: <topic>
allowed-tools: read write find web_search agent ask_user
---

# Research Skill - Preliminary Research

## Trigger
`/skill:research <topic>`

Topic: $ARGUMENTS

## Workflow

### Step 1: Generate Initial Framework from Model Knowledge
Based on topic, use model's existing knowledge to generate:
- Main research objects/items list in this domain
- Suggested research field framework

Output {step1_output}, use `ask_user` to confirm:
- Need to add/remove items?
- Does field framework meet requirements?

### Step 2: Web Search Supplement
Use `ask_user` to ask for time range (e.g., last 6 months, since 2024, unlimited).

**Parameter Retrieval**:
- `{topic}`: User input research topic
- `{YYYY-MM-DD}`: Current date
- `{step1_output}`: Complete output from Step 1
- `{time_range}`: User specified time range
- `{modules_dir}`: `${CLAUDE_SKILL_DIR}/modules`

**Hard Constraint**: The following prompt must be strictly reproduced, only replacing variables in {xxx}, do not modify structure or wording.

Spawn 1 `web-researcher` agent with the `agent` tool (`action: spawn`, `agent: web-researcher`), then `agent wait` for it and get the full report with `agent read {id}` (`agent wait` cuts reports at 6000 characters). **Prompt Template** (the `task`):
```python
prompt = f"""## Task
Research topic: {topic}
Current date: {YYYY-MM-DD}

Based on the following initial framework, supplement latest items and recommended research fields.

## Existing Framework
{step1_output}

## Strategy Modules
{modules_dir}

## Goals
1. Verify if existing items are missing important objects
2. Supplement items based on missing objects
3. Continue searching for {topic} related items within {time_range} and supplement
4. Supplement new fields

## Output Requirements
Return structured results directly (do not write files):

### Supplementary Items
- item_name: Brief explanation (why it should be added)
...

### Recommended Supplementary Fields
- field_name: Field description (why this dimension is needed)
...

### Sources
- [Source1](url1)
- [Source2](url2)
"""
```

**One-shot Example** (assuming researching AI Coding History):
```
## Task
Research topic: AI Coding History
Current date: 2025-12-30

Based on the following initial framework, supplement latest items and recommended research fields.

## Existing Framework
### Items List
1. GitHub Copilot: Developed by Microsoft/GitHub, first mainstream AI coding assistant
2. Cursor: AI-first IDE, based on VSCode
...

### Field Framework
- Basic Info: name, release_date, company
- Technical Features: underlying_model, context_window
...

## Strategy Modules
/home/user/.novacode/packages/deep-research/skills/research/modules

## Goals
1. Verify if existing items are missing important objects
2. Supplement items based on missing objects
3. Continue searching for AI Coding History related items within since 2024 and supplement
4. Supplement new fields

## Output Requirements
Return structured results directly (do not write files):

### Supplementary Items
- item_name: Brief explanation (why it should be added)
...

### Recommended Supplementary Fields
- field_name: Field description (why this dimension is needed)
...

### Sources
- [Source1](url1)
- [Source2](url2)
```

### Step 3: Ask User for Existing Fields
Use `ask_user` to ask if user has existing field definition file, if so read and merge.

### Step 4: Generate Outline (Separate Files)
Merge {step1_output}, {step2_output} and user's existing fields, generate two files:

**outline.yaml** (items + config):
- topic: Research topic
- items: Research objects list
- execution:
  - batch_size: Number of parallel agents (confirm with `ask_user`; novacode runs at most `agents.maxConcurrent`, default 4, at once and queues the rest)
  - items_per_agent: Items per agent (confirm with `ask_user`)
  - output_dir: Results output directory (default: ./results)

**fields.yaml** (field definitions). Use exactly this shape; `validate_json.py` rejects any other:
```yaml
fields:
  <category>:
    - {name: <field_name>, description: <what to collect>, detail_level: brief|moderate|detailed}
uncertain: []
```
- detail_level hierarchy: brief -> moderate -> detailed
- uncertain: Uncertain fields list (reserved field, auto-filled in deep phase)

### Step 5: Output and Confirm
- Create directory: `./{topic_slug}/`
- Save: `outline.yaml` and `fields.yaml`
- Show to user for confirmation

## Output Path
```
{current_working_directory}/{topic_slug}/
  ├── outline.yaml    # items list + execution config
  └── fields.yaml     # field definitions
```

## Follow-up Commands
- `/skill:research-add-items` - Supplement items
- `/skill:research-add-fields` - Supplement fields
- `/skill:research-deep` - Start deep research
