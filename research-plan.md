# 802.11n Long-Distance Parameter Research

## Objective

Identify all 802.11n parameters that must be modified from their default standard values to support reliable connections between stations and access points at distances up to hundreds of kilometers — far beyond the standard's design assumptions.

## Scope

### Network types
- Infrastructure mode (AP + STA)
- IBSS mode (ad-hoc, peer-to-peer)

### Protocol layers (all must be covered)
- PHY (Physical layer) — timing, guard intervals, preamble, modulation constraints
- MAC (Medium Access Control) — ACK/CTS timeouts, slot times, IFS values, RTS/CTS thresholds
- MAC management — beacon intervals, DTIM, probe/association timers, scanning parameters
- Aggregation — A-MPDU/A-MSDU limits, block ACK windows, ADDBA timeouts
- MIMO/MCS — spatial stream constraints, MCS rate viability at distance
- Regulatory — EIRP limits, transmit power, antenna gain

### Channel width
802.11n supports 20 MHz and 40 MHz channel widths. Some timing parameters differ by channel width, and long-distance links may favor one over the other. Both must be considered and differences noted.

### MIMO and spatial streams
802.11n introduced MIMO with up to 4 spatial streams. At extreme distances, not all MCS rates remain viable. Research must cover MCS selection constraints and any MIMO-related parameters that interact with distance.

### Regulatory and power constraints
EIRP limits and antenna gain considerations for long-distance links. Note where regulatory constraints interact with or constrain parameter choices for long-distance operation.

### Parameter criticality
Each parameter must be classified by criticality:
- **Mandatory** — must be changed or the link will not establish / function at all (e.g., ACK timeout)
- **Optimization** — improves performance or reliability but is not strictly required for basic operation (e.g., aggregation tuning)

### What qualifies as a "parameter"
A parameter is included if:
1. It has a defined default or recommended value in 802.11n (amendment or base standard)
2. Its default value assumes short propagation delay (typically <1 km)
3. Modifying it is necessary or beneficial for long-distance operation (up to hundreds of km)
4. It is configurable in practice (driver/firmware level)

## Parameter metadata

Every parameter uses the template defined in `parameter-template.md`. This template includes:
- Official IEEE name and description from the 802.11-2020 standard
- Page references to `sources/802.11-2020-reissue-2022-01.pdf`
- Usage context (how and when the parameter is used)
- Default and long-distance values
- Channel width, MIMO/MCS, and regulatory notes
- Source attribution (web, PDF, or training knowledge)

## Three research sources

All research draws from three distinct, independent sources:

1. **Web search** — Comprehensive internet research using `WebSearch` and `WebFetch` tools. Covers academic papers, deployment reports, vendor documentation, forum discussions, and regulatory databases.

2. **PDF analysis** — Deep analysis of the two reference documents in `sources/`:
   - `802.11-2020-reissue-2022-01.pdf` — IEEE 802.11-2020 standard (authoritative specification)
   - `802.11® Wireless Networks The Definitive Guide 2nd Edition.pdf` — O'Reilly reference book (covers pre-802.11n concepts that carry forward)

3. **Model training knowledge** — Opus 4.6's built-in knowledge from training data, used without internet or file access. Captures well-known parameters and configurations that may not appear in search results or the specific PDF pages examined.

Each source is researched independently by isolated subagents. Results are merged in the main conversation during Phase 2.

## Research phases

### Phase 1 — Parallel research (all 3 sources simultaneously)

Three groups of subagents run in parallel, one group per source. See `orchestration-plan.md` for detailed subagent specifications.

**Phase 1A: Web search subagents** (~5 agents)
- Comprehensive web search covering all parameter categories
- Each agent focuses on a specific topic area
- Agents use `WebSearch`, `WebFetch`, and `Write` (for log files) tools

**Phase 1B: PDF analysis subagents** (~5-7 agents)
- First, orchestrator agents scan each PDF to build a section/page index
- Then, deep-dive agents analyze specific sections for parameter details
- Agents use `Read` tool with page ranges only
- See `orchestration-plan.md` for PDF handling strategy to avoid context overflow

**Phase 1C: Training knowledge subagents** (~3 agents)
- Agents rely exclusively on Opus 4.6's training knowledge
- Explicitly forbidden from using `WebSearch`, `WebFetch`, `Read`, or any file/internet tools — but MAY use `Write` for log files
- Each agent covers a different parameter category

**Output:** `phase1-research-results.md` — merged summary of all findings from all sources, with source attribution. Also `phase1-research-results.xlsx` with the same data in tabular form.

### Phase 2 — Parameter extraction (main conversation)

Merge all Phase 1 results (process PDF outputs first for IEEE names, then web, then knowledge):
1. Read agent output files one at a time, extracting parameters to a disk accumulator
2. Deduplicate and normalize parameter names (use IEEE names from PDF agents)
3. Assign preliminary layer and criticality tags
4. Note which source types mentioned each parameter
5. Output uses the accumulator schema (not the full `parameter-template.md` — the full template is only populated in Phase 4)

**Output:** `phase2-candidate-parameters.md` and `phase2-candidate-parameters.xlsx` — candidate list with source attribution and preliminary metadata.

### Phase 3 — Verification (parallel subagents)

Two verification tracks run in parallel:

**Track A: Cross-source verification subagents** (~3 agents, split by layer)
- Confirm each parameter applies to 802.11n specifically (not only 802.11ac/ax/be)
- Cross-reference findings across the 3 sources
- Verify criticality classification (mandatory vs. optimization)
- Use web search to resolve conflicts between sources

**Track B: IEEE standard verification subagents** (~3-4 agents, split by layer)
- Look up each candidate parameter in `sources/802.11-2020-reissue-2022-01.pdf`
- Record the official IEEE parameter name as it appears in the standard
- Record the official description from the standard
- Record the exact section number, page number(s), and table/figure references
- Use the PDF index built during Phase 1B to locate parameters efficiently
- Agents use `Read` tool with targeted page ranges only

**Output:** `phase3-verified-parameters.md` and `phase3-verified-parameters.xlsx` — verified list with complete metadata including official IEEE names, descriptions, and page references. Notes on what was confirmed/rejected and why.

### Phase 4 — Final report (main conversation or single agent)

Produce `parameters.md` — the final deliverable, structured as follows:

1. **Title and introduction** — project objective and methodology summary
2. **Summary of findings** — high-level overview of how many parameters were found, breakdown by layer and criticality, key insights
3. **Parameter tables by layer** — one summary table per layer (PHY, MAC, MAC management, Aggregation, MIMO/MCS, Regulatory) with key metadata columns
4. **Detailed parameter sections by layer** — for each layer, a subsection for every parameter with the full metadata from `parameter-template.md`
5. **Conclusion** — synthesis of findings, recommendations for long-distance 802.11n deployment, identified gaps or uncertainties, list of any skipped agents and their impact

Also produce `parameters.xlsx` with all parameters and their full metadata in tabular form.

## Output convention

- **All files must be within the project directory** — no access to any path outside `/home/rameli/work/wifi4-research-gemini/`
- Every research phase writes its results to disk as both markdown (`.md`) and Excel (`.xlsx`) files before proceeding to the next phase
- All phase output files are written to the project root
- Phase outputs: `phase1-research-results.{md,xlsx}`, `phase2-candidate-parameters.{md,xlsx}`, `phase3-verified-parameters.{md,xlsx}`
- Final deliverables: `parameters.md`, `parameters.xlsx`
- Each phase must complete and write its output before the next phase begins
- The parameter metadata template is in `parameter-template.md`
- Python operations use the project venv at `.venv/`

## Subagent logging and tracking

- Every subagent writes a task log at start: `log/{phase}-{agent_id}-task.md`
- Every subagent writes its output to: `log/{phase}-{agent_id}-output.md`
- The orchestrator maintains timing data in: `log/orchestrator-timing.md`
- File names use the agent ID (e.g., W1, P-Scan-1, K2, V-B3) to avoid conflicts
- All agents have `Write` access for `log/` files in addition to their source-specific tools
- See `orchestration-plan.md` for detailed formats and data schemas

## Error handling

- If a subagent fails, the orchestrator logs the failure and retries once
- If the retry also fails, the agent is skipped — this is recorded in `log/orchestrator-timing.md` and `research-state.md`
- Skipped agents are noted in the final report's conclusion as potential information gaps
- The research continues without blocking on failed agents

## Context overflow prevention

- Merging phases (Phase 2, Phase 3) process agent output files one at a time, not all at once
- Extracted data is written to disk accumulators (`log/{phase}-accumulator.md`) — disk is the authoritative merge state, not conversation context
- Partial results are written to disk as soon as they are ready
- See `orchestration-plan.md` for detailed procedures

## Resumability

A checkpoint file `research-state.md` is maintained in the project root. It records:
- Current phase and overall status
- Completed agents and their output file paths
- Skipped agents with failure reasons
- Pending agents with full specifications for relaunch
- Next steps to resume after an interruption

If the research is interrupted (e.g., usage limits), the checkpoint file allows a new conversation to pick up exactly where work stopped, skipping completed agents and relaunching only what remains.

## Prerequisites

Before starting research, verify:
1. `poppler-utils` is installed (`which pdftotext`) — required for PDF reading
2. Project venv exists at `.venv/` with `openpyxl` installed
3. `log/` directory exists
