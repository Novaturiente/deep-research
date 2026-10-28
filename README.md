# deep-research

Deep research skills for [novacode](https://github.com/Novaturiente). Ported from [Weizhena/Deep-Research-skills](https://github.com/Weizhena/Deep-Research-skills) (MIT, commit `6ce38f6`), which was inspired by [RhinoInsight](https://arxiv.org/abs/2511.18743).

Two ways to research:

- **Structured** (`research` → `research-deep` → `research-report`): pick the items to study and the fields to collect, then research every item in parallel and turn the results into one report. Good for comparisons, surveys and tool selection.
- **Open-ended** (`research-quick`): one question in, one cited report out.

## Install

```sh
novacode install https://github.com/Novaturiente/deep-research.git
```

Or link a local clone, so your edits apply at once:

```sh
git clone https://github.com/Novaturiente/deep-research.git ~/personal/deep-research
novacode install ~/personal/deep-research
```

Requirements:

- A working `web_search` in novacode (`novacode login antigravity`, `GEMINI_API_KEY`, `BRAVE_API_KEY`, or the setting `web.searxngUrl`).
- `python3` with PyYAML (`pip install pyyaml`) for the validator and the report script.

## Commands

| Command | What it does |
|---|---|
| `/skill:research <topic>` | Make `outline.yaml` (items) and `fields.yaml` (fields) |
| `/skill:research-add-items` | Add items to the outline |
| `/skill:research-add-fields` | Add fields to the outline |
| `/skill:research-deep` | Research each item in parallel, one JSON per item |
| `/skill:research-report` | Turn the JSON results into `report.md` |
| `/skill:research-quick <question>` | Open-ended research, one cited `report.md` |

The model can also start them by itself when you ask for research.

## What is in the repo

```
agents/web-researcher.md         subagent that does the searching
skills/research/                 outline skill
  validate_json.py               checks each JSON covers every field
  modules/                       search strategies per source type
skills/research-*/               the other skills
tests/                           validator regression test
```

## Differences from upstream

- Tool names are novacode's: `agent` (not `Task`), `ask_user` (not `AskUserQuestion`), `web_search`, `web_fetch`.
- Paths come from the skill's own folder (`${CLAUDE_SKILL_DIR}`), not `~/.claude/...`. The strategy modules folder is passed to the subagent in each task.
- `research-quick` is new.
- English only; the Chinese, OpenCode and Codex variants are not ported.

## Test

```sh
python3 tests/test_validate_json_schemas.py
```

## License

MIT. See `LICENSE`.
