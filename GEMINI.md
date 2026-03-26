# GEMINI.md

This file provides guidance to Gemini CLI when working with code in this repository.

# 802.11n Long-Distance Parameter Research

## Project overview

Research project to identify all 802.11n parameters requiring modification for long-distance links (up to hundreds of km). See `research-plan.md` for full scope, `orchestration-plan.md` for detailed subagent strategy.

This is a pure research project — there is no code to build, lint, or test. All work is web research, PDF analysis, and markdown documentation.

## Reference sources

The `sources/` directory contains primary reference material:
- `802.11-2020-reissue-2022-01.pdf` — IEEE 802.11-2020 standard (authoritative spec; large PDF — use page ranges when reading)
- `802.11® Wireless Networks The Definitive Guide 2nd Edition.pdf` — O'Reilly reference book (covers pre-802.11n WiFi concepts that carry forward to 802.11n)

## Hard constraints

### Path restriction
**ALL file access (read and write) MUST be limited to `/home/rameli/work/wifi4-research-gemini/` and its subdirectories.** No agent or orchestrator may read, write, or reference any path outside this folder. This applies to all tools. Every agent prompt MUST include this constraint verbatim.

### Python virtual environment
**All Python operations MUST use the project-local venv at `.venv/`.** No system-wide installations.
- Run via: `.venv/bin/python script.py`
- Install via: `.venv/bin/pip install <package>`
- **All `pip install` must happen in the main conversation BEFORE launching parallel agents** — never inside a subagent.

## Prerequisites

Before starting research, the orchestrator must ensure:
1. `poppler-utils` is installed (required for PDF reading). Check with `which pdftotext`. If missing, prompt the user.
2. The project venv exists at `.venv/` with `openpyxl` installed.
3. The `log/` directory exists.

## Parameter metadata

Every parameter uses the template in `parameter-template.md`. Use "TBD" for unknown fields. Key fields:
- Official IEEE name and description (from 802.11-2020 standard)
- Page reference to `sources/802.11-2020-reissue-2022-01.pdf`
- Usage context: how and in which scenarios the parameter is used
- Source attribution: which of the 3 research sources contributed the information

## Output files

Every phase produces `.md` and `.xlsx` (via `openpyxl` from `.venv/`):
- `phase1-research-results.{md,xlsx}`
- `phase2-candidate-parameters.{md,xlsx}`
- `phase3-verified-parameters.{md,xlsx}`
- `parameters.{md,xlsx}` (final deliverable)

## Subagent rules

1. **Isolation**: Each subagent operates independently. Its prompt must contain all context it needs.
2. **Tool restrictions**: Per source type (see `orchestration-plan.md`). **All agents also have `Write` access** for `log/` files.
3. **Logging**: Every agent writes `log/{phase}-{agent_id}-task.md` at start and `log/{phase}-{agent_id}-output.md` on completion.
4. **Path restriction**: Every agent prompt must include: "You may ONLY access files under `/home/rameli/work/wifi4-research-gemini/`."

### Subagent task log format
```markdown
# Task Log: {Agent ID}
- **Phase**: {phase number and name}
- **Agent ID**: {e.g., W1, P-Scan-1, K2}
- **Task summary**: {1-2 sentences}
- **Source type**: {Web / PDF / Training knowledge}
- **Tool permissions**: {list}
- **Target parameters/topics**: {what to look for}
- **PDF page ranges** (if applicable): {ranges}
- **Page budget** (if applicable): {max pages}
- **Start time**: {ISO timestamp}
- **Status**: IN_PROGRESS
```
On completion: update status to `COMPLETED`, add `**End time**`.

### Error handling
- Failed agent → log failure, retry once (reduce page budget by 30% if context overflow)
- Second failure → skip, record in `research-state.md` and `log/orchestrator-timing.md` as `FAILED-SKIPPED`, flag in final report

### PDF rules
- Max 20 pages per `Read` call; auto-chunk larger ranges
- Extract and summarize after each chunk — do not accumulate raw text
- Write findings to output file incrementally
- Deep-dive agents: ~100 page budget total

### Context overflow prevention
- Process agent output files **one at a time** during merge phases
- Use disk accumulators (`log/{phase}-accumulator.md`) as authoritative merge state
- Write partial results to disk immediately — never rely solely on conversation context

### Resumability
Checkpoint file `research-state.md` tracks: current phase, completed/in-progress/skipped/pending agents, output files written, and exact next steps. Updated after every significant event. On resume: read checkpoint, skip completed work, continue from where interrupted.

## Orchestrator tracking
- Timing report: `log/orchestrator-timing.md` — records agent ID, task, start/end time, duration, status, attempt count
- Updated after each agent completes or fails

## File conventions
- All files within `/home/rameli/work/wifi4-research-gemini/`
- Phase outputs: project root
- Subagent logs/outputs: `log/`
- Merge accumulators: `log/{phase}-accumulator.md`
- Checkpoint: `research-state.md`
- Orchestrator timing: `log/orchestrator-timing.md`
- Detailed orchestration: `orchestration-plan.md`
