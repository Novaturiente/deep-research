---
name: research-deep
description: Read the research outline (outline.yaml, fields.yaml) and launch a web-researcher subagent per item batch to fill a validated JSON file for each item. Second step of the research-* workflow.
allowed-tools: read write find bash web_search agent
---

# Research Deep - Deep Research

## Trigger
`/skill:research-deep`

## Workflow

### Step 1: Auto-locate Outline
Find `*/outline.yaml` file in current working directory, read items list, execution config (including items_per_agent).

### Step 2: Resume Check
- Check completed JSON files in output_dir
- Skip completed items

### Step 3: Batch Execution
- Batch by batch_size (need user approval before next batch)
- Each agent handles items_per_agent items
- Spawn `web-researcher` agents with the `agent` tool (`agent: web-researcher`, `notify: quiet`): their reports stay out of the conversation, because each agent writes its own output file

**Parameter Retrieval**:
- `{topic}`: topic field from outline.yaml
- `{item_name}`: item's name field
- `{item_related_info}`: item's complete yaml content (name + category + description etc.)
- `{output_dir}`: execution.output_dir from outline.yaml (default: ./results)
- `{fields_path}`: absolute path to {topic}/fields.yaml
- `{output_path}`: absolute path to {output_dir}/{item_name_slug}.json (slugify item_name: replace spaces with _, remove special chars)
- `{modules_dir}`: `${CLAUDE_SKILL_DIR}/../research/modules`
- `{validator}`: `${CLAUDE_SKILL_DIR}/../research/validate_json.py`

**Hard Constraint**: The following prompt must be strictly reproduced, only replacing variables in {xxx}, do not modify structure or wording.

**Prompt Template**:
```python
prompt = f"""## Task
Research {item_related_info}, output structured JSON to {output_path}

## Field Definitions
Read {fields_path} to get all field definitions

## Strategy Modules
{modules_dir}

## Output Requirements
1. Output JSON according to fields defined in fields.yaml
2. Mark uncertain field values with [uncertain]
3. Add uncertain array at the end of JSON, listing all uncertain field names
4. All field values must be in English

## Output Path
{output_path}

## Validation
After completing JSON output, run validation script to ensure complete field coverage:
python3 {validator} -f {fields_path} -j {output_path}
Task is complete only after validation passes.
"""
```

**One-shot Example** (assuming researching GitHub Copilot):
```
## Task
Research name: GitHub Copilot
category: International Product
description: Developed by Microsoft/GitHub, first mainstream AI coding assistant, ~40% market share, output structured JSON to {project_dir}/results/GitHub_Copilot.json

## Field Definitions
Read {project_dir}/fields.yaml to get all field definitions

## Strategy Modules
{modules_dir}

## Output Requirements
1. Output JSON according to fields defined in fields.yaml
2. Mark uncertain field values with [uncertain]
3. Add uncertain array at the end of JSON, listing all uncertain field names
4. All field values must be in English

## Output Path
{project_dir}/results/GitHub_Copilot.json

## Validation
After completing JSON output, run validation script to ensure complete field coverage:
python3 {validator} -f {project_dir}/fields.yaml -j {project_dir}/results/GitHub_Copilot.json
Task is complete only after validation passes.
```

### Step 4: Wait and Monitor
- `agent wait` for the current batch
- Check each output file exists and passes the validator; re-spawn an agent once for an item that failed
- Launch next batch
- Display progress

### Step 5: Summary Report
After all complete, output:
- Completion count
- Failed/uncertain marked items
- Output directory

## Agent Config
- Background execution: Yes (`notify: quiet`, then `agent wait`)
- Task Output: Disabled (agent has explicit output file when complete)
- Resume support: Yes (Step 2)

## Follow-up Commands
- `/skill:research-report` - Generate the markdown report
