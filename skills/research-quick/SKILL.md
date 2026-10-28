---
name: research-quick
description: Open-ended deep research on one question. Splits it into sub-questions, researches them in parallel with web-researcher subagents, fills gaps in a second round, and writes one cited markdown report. Use for "deep research X" when no item-by-field comparison table is needed.
argument-hint: <question>
allowed-tools: read write find web_search web_fetch agent ask_user
---

# Research Quick - Open-ended Deep Research

## Trigger
`/skill:research-quick <question>`

Question: $ARGUMENTS

Use the `research` skill instead when the user wants many items compared on the same fields.

## Workflow

### Step 1: Scope
If the question has more than one reasonable reading (time range, region, audience, depth), ask once with `ask_user`. Otherwise go on.

### Step 2: Plan Sub-questions
Split the question into 3-6 sub-questions that together answer it. Each must be researchable on its own. Show the list in one short message, then continue without waiting.

### Step 3: Round 1, in Parallel
Spawn one `web-researcher` agent per sub-question with the `agent` tool (`agent: web-researcher`, `notify: quiet`), then `agent wait` for all of them. novacode runs at most `agents.maxConcurrent` (default 4) at once and queues the rest.

**Prompt Template** (only replace variables in {xxx}):
```
## Task
Main question: {question}
Your sub-question: {sub_question}
Current date: {YYYY-MM-DD}

## Strategy Modules
${CLAUDE_SKILL_DIR}/../research/modules

## Output Requirements
Return results directly (do not write files):

### Findings
- One claim per bullet, each with its source number [n]. Mark a claim with [uncertain] when only one weak source supports it.

### Conflicts
- Where sources disagree, and which one is more credible and why.

### Gaps
- What you could not find or verify.

### Sources
1. [Title](url) - date if known
```

### Step 4: Gap Check and Round 2
Read every full report with `agent read {id}`: reports that reach you through `agent wait` are cut at 6000 characters, and the cut part often holds the sources. List the gaps and conflicts that matter for the main question. If there are any, run one more round (same template, the gap as the sub-question, at most 4 agents). Never run more than 2 rounds in total.

### Step 5: Write the Report
Write `./{question_slug}/report.md`:
- `# {question}` and the date
- `## Summary`: the answer in 3-6 sentences
- One `##` section per sub-question, claims with citation numbers [n]
- `## Conflicts and Uncertainty`: disagreements and [uncertain] claims
- `## Open Questions`: gaps still left
- `## Sources`: one merged, numbered, deduplicated list; renumber the citations to match

Every claim in the report cites at least one source from the agents' reports. Every source has its URL. Never add a source or a claim no agent reported.

### Step 6: Reply
Give the report path, the summary, and the number of sources and rounds.
