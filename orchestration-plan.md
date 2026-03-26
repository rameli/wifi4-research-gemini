# Orchestration Plan

Detailed subagent orchestration strategy for the 802.11n long-distance parameter research. This file is used by the orchestrator (main conversation) only. See `CLAUDE.md` for general rules and constraints that apply to all agents.

---

## Data schemas

All agents must use these exact formats for their output files. This ensures consistent, parseable data flow between phases.

### Phase 1 agent output schema (all W, P-Deep, K agents)

Every Phase 1 agent writes its output file as a markdown list of parameter entries. Each entry uses this format:

```markdown
### {Parameter Name}
- **Layer**: {PHY / MAC / MAC management / Aggregation / MIMO-MCS / Regulatory}
- **Criticality**: {Mandatory / Optimization / TBD}
- **Default value**: {value with units, or TBD}
- **Long-distance value**: {value or formula, or TBD}
- **Description**: {1-2 sentence functional description}
- **Usage context**: {when/how this parameter is used in WiFi operation}
- **Distance impact**: {why this matters for long-distance links}
- **Channel width notes**: {20 vs 40 MHz differences, or "Same for both"}
- **MIMO/MCS relevance**: {interaction with spatial streams or MCS, or "None"}
- **IEEE section/page**: {section and page if known from PDF, or "TBD"}
- **Source URLs**: {URLs if from web search, or "N/A"}
- **Notes**: {any additional context, formulas, caveats}
```

Agents should list every parameter they find, one entry per parameter. Partial data is fine — use "TBD" for unknown fields.

### PDF scan index schema (P-Scan-1, P-Scan-2)

The scan agents write their index as a markdown table:

```markdown
# PDF Index: {document name}

Total pages: {N}

## Section Index
| Section | Title | Page Range | Relevance | Topics |
|---------|-------|------------|-----------|--------|
| 9.2     | MAC sublayer functional description | 45-120 | HIGH | frame exchange, timing |
| 19      | HT PHY specification | 200-280 | HIGH | OFDM, guard interval, MCS |
| Annex C | MIB | 350-380 | HIGH | parameter definitions |

## Parameter Name Index
| Parameter Name | Page(s) | Section |
|----------------|---------|---------|
| aSlotTime      | 67, 210 | 9.2, 19 |
| dot11RTSThreshold | 350  | Annex C |
```

The "Relevance" column uses: HIGH (directly contains parameter definitions or timing values), MEDIUM (contextual — frame exchange procedures, protocol descriptions), LOW (tangential).

**Note on Parameter Name Index**: P-Scan-1 (IEEE standard) should produce a detailed parameter name index since the standard's back-of-book index and MIB sections use formal parameter names (e.g., `aSlotTime`, `dot11RTSThreshold`). P-Scan-2 (O'Reilly book) should produce this on a **best-effort basis** — the book likely uses informal names (e.g., "slot time", "RTS threshold") rather than IEEE identifiers. Map informally named entries to their likely IEEE equivalents where obvious.

### Phase 2 accumulator schema

Each entry in `log/phase2-accumulator.md`:

```markdown
### {Parameter Name}
- **Source agents**: {comma-separated list of agent IDs that found this, e.g., "W1, P-Deep-1, K1"}
- **Layer**: {best determination from sources}
- **Criticality**: {best determination from sources}
- **Default value**: {best determination, with source}
- **Long-distance value**: {best determination, with source}
- **Description**: {merged from best source}
- **Usage context**: {merged}
- **IEEE section/page**: {from PDF agents if available, else TBD}
- **Has Phase 1B page ref**: {YES / NO — used by Phase 3 Track B to decide fresh lookup vs verify-only}
- **Conflicts**: {note any disagreements between sources}
```

### Phase 3 Track A input

The orchestrator embeds in each V-A agent's prompt:
- The list of parameter names and layers for the agent's layer group
- For each parameter: the current criticality, default value, description, and any noted conflicts between sources
- The specific verification questions: "Does this parameter exist in 802.11n? Is the criticality correct?"

### Phase 3 Track A output schema

```markdown
### {Parameter Name}
- **802.11n confirmed**: {YES / NO / PARTIAL — e.g., parameter exists but only in HT greenfield mode}
- **Criticality verified**: {Mandatory / Optimization — with justification if changed}
- **Conflicts resolved**: {resolution if sources disagreed, or "No conflicts"}
- **Verification source**: {URL or reference used to verify}
- **Notes**: {any caveats, e.g., "Only applies to 5 GHz band"}
```

### Phase 3 Track B input

The orchestrator embeds in each V-B agent's prompt:
- The list of parameter names for the agent's layer group
- For each parameter: a flag `FRESH_LOOKUP` or `VERIFY_ONLY`
  - `FRESH_LOOKUP`: no page reference from Phase 1B — agent must find it in the standard
  - `VERIFY_ONLY`: existing page reference provided — agent reads just that page to confirm official name and description
- The PDF scan index from P-Scan-1 (for `FRESH_LOOKUP` parameters)
- The existing page references (for `VERIFY_ONLY` parameters)

### Phase 3 Track B output schema

```markdown
### {Parameter Name}
- **Official IEEE name**: {exact name from standard, e.g., `aSlotTime`}
- **Official IEEE description**: {quoted or closely paraphrased from standard}
- **Section**: {e.g., "9.4.2.28"}
- **Page(s)**: {page numbers in the PDF}
- **Table/Figure**: {e.g., "Table 9-21", or "N/A"}
- **HT context confirmed**: {YES — found in HT/802.11n context / NO — only in legacy context / NOT FOUND}
- **Lookup type**: {FRESH_LOOKUP / VERIFY_ONLY}
- **Notes**: {any discrepancies from Phase 1B data, additional context}
```

### Phase 3 accumulator schema

Each entry in `log/phase3-accumulator.md` merges Track A and Track B results for one parameter. The orchestrator matches entries across tracks by parameter name.

```markdown
### {Parameter Name}
- **Official IEEE name**: {from Track B}
- **Official IEEE description**: {from Track B}
- **Section**: {from Track B}
- **Page(s)**: {from Track B}
- **Table/Figure**: {from Track B}
- **HT context confirmed**: {from Track B}
- **802.11n confirmed**: {from Track A}
- **Criticality verified**: {from Track A}
- **Conflicts resolved**: {from Track A}
- **Verification sources**: {URLs from Track A, lookup type from Track B}
- **Notes**: {merged notes from both tracks}
```

**Merge procedure**: Process Track A outputs first — create one accumulator entry per parameter with Track A fields filled, Track B fields set to "PENDING". Then process Track B outputs — for each parameter, find its existing accumulator entry by name and fill in the Track B fields. If a Track B parameter has no Track A entry (or vice versa), create a new entry with the missing track's fields set to "NOT VERIFIED".

### Orchestrator layer split preparation for Phase 3

Track A and Track B use different layer groupings. The orchestrator must prepare separate parameter sublists for each:

**Track A groupings:**
- V-A1: PHY + Regulatory parameters
- V-A2: MAC + MAC management parameters
- V-A3: Aggregation + MIMO-MCS parameters

**Track B groupings:**
- V-B1: PHY parameters
- V-B2: MAC parameters
- V-B3: MAC management + Aggregation parameters
- V-B4: MIMO-MCS + Regulatory parameters

A single parameter appears in exactly one agent per track, but may be in differently-grouped agents across tracks. The orchestrator must prepare 7 distinct parameter sublists (3 for Track A + 4 for Track B).

---

## Phase 1 — Parallel research (all 3 sources simultaneously)

Phase 1A (web), Phase 1B Stage 1 (PDF scanning), and Phase 1C (training knowledge) all launch in parallel. Phase 1B Stage 2 (PDF deep-dive) launches after Stage 1 completes.

### Phase 1A — Web search subagents (5 agents, parallel)

Launch all 5 simultaneously using `Agent` tool with `subagent_type: "general-purpose"`.

**Tool permissions**: `WebSearch`, `WebFetch`, and `Write` (for `log/` files only). No file reading.

**Agent W1: PHY layer timing**
- Search for: 802.11n slot time, SIFS, DIFS, EIFS, guard intervals (short/long GI), preamble durations, OFDM symbol timing
- Include 20 MHz vs 40 MHz timing differences
- Search for how these parameters affect or are affected by propagation delay

**Agent W2: MAC layer parameters**
- Search for: ACK timeout, CTS timeout, RTS/CTS thresholds, fragmentation thresholds, NAV duration limits
- Include ACK timeout calculation formulas and distance dependency
- Search for coverage class and its relationship to distance

**Agent W3: Long-distance WiFi deployments**
- Search for: real-world long-range 802.11n deployments, WISP configurations, point-to-point links over tens/hundreds of km
- Focus on which parameters operators actually changed and to what values
- Include both infrastructure and IBSS/ad-hoc mode experiences
- Capture specific distance records and the configurations used

**Agent W4: MAC management and aggregation**
- Search for: beacon interval, DTIM period, probe request/response timers, association timers, scan dwell times
- Also cover: A-MPDU/A-MSDU aggregation limits, block ACK window sizes, ADDBA timeout
- Include IBSS-specific management parameters (ATIM window, beacon generation)

**Agent W5: MIMO, MCS, and regulatory**
- Search for: MCS rate viability at long distance, spatial stream behavior at range, MIMO performance vs distance
- Also cover: EIRP limits, maximum transmit power regulations, antenna gain considerations for long-range links
- Include frequency band differences (2.4 GHz vs 5 GHz) for long distance

---

### Phase 1B — PDF analysis subagents (5-7 agents, two-stage)

#### Stage 1: PDF scanning (2 agents, parallel — launch with Phase 1A and 1C)

These agents perform a quick structural scan of each PDF to build a page index. They do NOT deep-dive into content.

**Agent P-Scan-1: IEEE 802.11-2020 standard scanner**
- **Tool permissions**: `Read` (with page ranges) and `Write` (for `log/` files only)
- **Task**: Scan the IEEE standard to build a structural index
- **Procedure**:
  1. Read the table of contents (first ~10-20 pages) to identify all major sections and their page ranges
  2. Read the index at the end of the PDF (last ~20-30 pages) to identify where specific parameter names appear
  3. Identify sections most relevant to 802.11n long-distance parameters:
     - PHY layer sections (OFDM, HT PHY)
     - MAC layer sections (frame exchange, timing, acknowledgment)
     - Management frame sections
     - MIB (Management Information Base) sections — these contain parameter definitions
     - HT (High Throughput) specific sections — the 802.11n content
  4. Return a structured index: section name, page range, relevance to long-distance parameters
- **Output**: Written to `log/phase1b-P-Scan-1-output.md`. This file is critical — the orchestrator reads it to construct deep-dive agent prompts.
- **Critical**: Do NOT read the entire PDF. Read only TOC, index, and section headers. Budget: ~60-80 pages total across all reads.

**Agent P-Scan-2: O'Reilly book scanner**
- **Tool permissions**: `Read` (with page ranges) and `Write` (for `log/` files only)
- **Task**: Scan the O'Reilly book to build a structural index
- **Procedure**:
  1. Read the table of contents (first ~10-15 pages) to identify structure and total page count
  2. Read the index at the end (last ~15-20 pages — adjust based on discovered page count)
  3. Identify chapters/sections covering:
     - PHY layer timing and parameters
     - MAC layer operations (ACK, RTS/CTS, IFS)
     - Management frames and parameters
     - Long-range or outdoor deployment considerations
  4. Return a structured index: chapter/section name, page range, relevance
- **Adaptive budgeting**: After reading the TOC, adjust budget based on total page count:
  - If <300 pages: budget ~40 pages total
  - If 300-600 pages: budget ~60 pages total
  - If >600 pages: budget ~80 pages total
- **Note**: This book predates 802.11n but its coverage of 802.11a/b/g fundamentals directly applies since 802.11n inherits these base parameters.
- **Output**: Written to `log/phase1b-P-Scan-2-output.md`.

#### Stage 1 → Stage 2 handoff (orchestrator responsibility)

Deep-dive agents can launch as soon as their scanner finishes — they do NOT need to wait for both scanners:

**When P-Scan-1 (IEEE standard) completes:**
1. Read `log/phase1b-P-Scan-1-output.md` (IEEE standard index)
2. For each IEEE deep-dive agent (P-Deep-1, P-Deep-2, P-Deep-3), extract the relevant page ranges from the index for that agent's topic area
3. Construct the agent prompts, **embedding the relevant portions of the index directly in each prompt** (not a file reference — the agent must have the data in its prompt so it can target its reads)
4. Launch P-Deep-1, P-Deep-2, P-Deep-3 in parallel
5. Update `research-state.md` checkpoint

**When P-Scan-2 (O'Reilly book) completes:**
1. Read `log/phase1b-P-Scan-2-output.md` (O'Reilly book index)
2. Extract the relevant page ranges for P-Deep-4
3. Construct the agent prompt with embedded index
4. Launch P-Deep-4
5. Update `research-state.md` checkpoint

This staggered launch maximizes parallelism — if P-Scan-1 finishes before P-Scan-2, the IEEE deep-dive agents start immediately without waiting for the book scanner.

#### Stage 2: PDF deep-dive (3-5 agents, parallel, launched as scanners complete)

These agents receive the page index from Stage 1 **embedded in their prompt** and read specific sections in detail.

**Agent P-Deep-1: IEEE standard — PHY and timing parameters**
- **Tool permissions**: `Read` (with page ranges) and `Write` (for `log/` files only)
- **Input**: Relevant sections of the page index from P-Scan-1 (embedded in prompt by orchestrator)
- **Task**: Read the identified PHY/HT-PHY sections of the IEEE standard
- **Extract**: All timing parameters (slot time, SIFS, DIFS, EIFS, guard intervals, OFDM symbol time, preamble durations), their definitions, default values, and any distance-related notes
- **Budget**: ~80-100 pages total across all reads. Use targeted page ranges from the index.

**Agent P-Deep-2: IEEE standard — MAC parameters and MIB**
- **Tool permissions**: `Read` (with page ranges) and `Write` (for `log/` files only)
- **Input**: Relevant sections of the page index from P-Scan-1 (embedded in prompt by orchestrator)
- **Task**: Read the MAC frame exchange and MIB sections
- **Extract**: ACK timeout, CTS timeout, RTS threshold, fragmentation threshold, NAV limits, coverage class, and all MIB parameter definitions with their default values
- **Budget**: ~80-100 pages total

**Agent P-Deep-3: IEEE standard — Management, aggregation, and HT-specific**
- **Tool permissions**: `Read` (with page ranges) and `Write` (for `log/` files only)
- **Input**: Relevant sections of the page index from P-Scan-1 (embedded in prompt by orchestrator)
- **Task**: Read management frame sections, A-MPDU/A-MSDU sections, block ACK sections, and HT-specific parameter sections
- **Extract**: Beacon interval, DTIM, probe/association timers, aggregation limits, block ACK window, ADDBA timeout, IBSS parameters (ATIM window)
- **Budget**: ~80-100 pages total

**Agent P-Deep-4: O'Reilly book — All relevant sections**
- **Tool permissions**: `Read` (with page ranges) and `Write` (for `log/` files only)
- **Input**: Relevant sections of the page index from P-Scan-2 (embedded in prompt by orchestrator)
- **Task**: Read the identified relevant chapters of the O'Reilly book
- **Extract**: Parameter explanations, practical configuration guidance, timing relationships, any long-range or outdoor deployment discussion
- **Budget**: ~80-100 pages total
- **Note**: Map pre-802.11n parameter names to their 802.11n equivalents where possible

#### PDF agent context overflow prevention

To prevent context overflow when analyzing large PDFs:

1. **Never read more than 20 pages in a single `Read` call** (tool limit)
2. **Budget total pages per agent**: ~100 pages max across all Read calls
3. **Use the index**: Always consult the page index to target reads. Do not scan sequentially.
4. **Extract and summarize**: After each Read call, immediately extract relevant parameter information into structured notes. Do not accumulate raw PDF text.
5. **Write incrementally**: After extracting parameters from a chunk, append them to the output file on disk. Do not hold all findings in context until the end.
6. **Skip irrelevant content**: If a page range turns out to be irrelevant, note it and move on.
7. **Prioritize MIB sections**: The MIB sections contain formal parameter definitions — most valuable for this research.

---

### Phase 1C — Training knowledge subagents (3 agents, parallel — launch with Phase 1A and 1B Stage 1)

Launch all 3 simultaneously. These agents use ONLY their built-in training knowledge for research, but MAY use `Write` to save their log and output files.

**Tool permissions**: `Write` ONLY (for `log/` files). **Explicitly forbidden** from using `WebSearch`, `WebFetch`, `Read`, `Glob`, `Grep`, `Bash`, or any other tool that accesses the internet or reads files.

**Prompt preamble for all Phase 1C agents**:
> You are researching 802.11n parameters that need modification for long-distance WiFi links (up to hundreds of km). You must answer ENTIRELY from your training knowledge. Do NOT use WebSearch, WebFetch, Read, Glob, Grep, Bash, or any tool that accesses the internet or reads files. The ONLY tool you may use is `Write` — to save your task log to `log/phase1c-{agent_id}-task.md` and your research output to `log/phase1c-{agent_id}-output.md`. You may ONLY write files under `/home/rameli/work/wifi4-research-gemini/`. Do NOT read, write, or reference any path outside this directory.

**Agent K1: PHY and MAC timing parameters**
- From training knowledge, list all 802.11n PHY timing parameters and MAC timing parameters
- Include default values, formulas, and distance dependencies
- Cover: slot time, SIFS, DIFS, EIFS, guard intervals, ACK timeout, CTS timeout, coverage class

**Agent K2: MAC management, aggregation, and IBSS**
- From training knowledge, list all 802.11n management parameters and aggregation parameters
- Include: beacon interval, DTIM, probe timers, A-MPDU/A-MSDU limits, block ACK, ADDBA timeout
- Cover IBSS-specific parameters: ATIM window, beacon generation rules

**Agent K3: MIMO/MCS, regulatory, and deployment practice**
- From training knowledge, describe MCS rate viability at long distance
- Cover MIMO spatial stream behavior at extreme range
- Include regulatory constraints (EIRP, power, antenna gain) for long-range links
- Describe known long-distance 802.11n deployment practices and configurations

---

## Phase 2 — Parameter extraction (main conversation)

No subagents. The main conversation merges all Phase 1 outputs using the context overflow prevention strategy.

**File processing order**: Process PDF deep-dive outputs first (P-Deep-1 through P-Deep-4) since they contain IEEE parameter names and page references that aid normalization. Then process web outputs (W1-W5). Then knowledge outputs (K1-K3). Within each group, order doesn't matter.

**Procedure:**

1. **Process output files one at a time in the order above** — for each `log/phase1*-output.md` file:
   a. Read the file
   b. Extract every distinct parameter using the Phase 1 agent output schema fields
   c. Append extracted parameters to `log/phase2-accumulator.md` on disk using the Phase 2 accumulator schema
   d. When a parameter already exists in the accumulator (from a prior file), update the existing entry: add the new agent ID to "Source agents", merge any new data into existing fields, note conflicts
   e. Note a brief summary of what was found (parameter count, names) before moving to the next file
2. After processing all output files, read `log/phase2-accumulator.md`
3. Deduplicate and normalize parameter names (use official IEEE names from PDF agents where available)
4. Verify layer and criticality tags — resolve any conflicts between sources
5. Note which of the 3 source types (web/PDF/knowledge) mentioned each parameter
6. Write `phase2-candidate-parameters.md` using the Phase 2 accumulator schema (NOT the full `parameter-template.md` — the full template is only used in Phase 4)
7. Generate `phase2-candidate-parameters.xlsx` using `.venv/bin/python`
8. Update `research-state.md` checkpoint

---

## Phase 3 — Parallel verification (two tracks, ~6-7 agents total)

### Track A: Cross-source verification (3 agents, parallel, split by layer)

**Tool permissions**: `WebSearch`, `WebFetch`, and `Write` (for `log/` files)

**Input**: Each agent receives (embedded in prompt by orchestrator):
- The candidate parameter list for its layer group (extracted from `phase2-candidate-parameters.md`)
- For each parameter: name, current criticality, default value, description, source agents, and any noted conflicts
- See "Phase 3 Track A input" in Data schemas section for exact format

**Agent V-A1: PHY + regulatory verification**
- Confirm each PHY parameter exists in 802.11n (not only later amendments)
- Verify regulatory parameters apply to 802.11n frequency bands
- Resolve any conflicts between web, PDF, and training knowledge sources

**Agent V-A2: MAC + MAC management verification**
- Confirm MAC parameters are part of 802.11n
- Verify IBSS-specific parameters exist in 802.11n
- Verify criticality classifications

**Agent V-A3: Aggregation + MIMO/MCS verification**
- Confirm aggregation parameters and limits are 802.11n-specific
- Verify MCS/MIMO claims against 802.11n capabilities (not 802.11ac/ax features)

### Track B: IEEE standard verification (3-4 agents, parallel, split by layer)

**Tool permissions**: `Read` (with page ranges) and `Write` (for `log/` files)

**Optimization**: Track B agents only do **fresh PDF lookups** for parameters that **lack** page references from Phase 1B. For parameters that already have section/page references from Phase 1B deep-dive agents, Track B agents **verify and augment** the existing references (confirm the official name and description are correct) without re-reading the full section — a targeted read of just the specific page(s) is sufficient.

These agents look up each candidate parameter in `sources/802.11-2020-reissue-2022-01.pdf` to:
1. Find the **official IEEE parameter name** as it appears in the standard
2. Copy or closely paraphrase the **official description** from the standard
3. Record the **section number**, **page number(s)**, and **table/figure references**
4. Confirm the parameter exists in the 802.11n (HT) context of the standard

**Input**: Each agent receives (embedded in prompt by orchestrator):
- The candidate parameter list for its layer group (from `phase2-candidate-parameters.md`)
- The PDF page index from Phase 1B Stage 1 (from `log/phase1b-P-Scan-1-output.md`)
- Any existing page references already found in Phase 1B (flagged so the agent knows which parameters need fresh lookup vs. verification only)

**Agent V-B1: IEEE verification — PHY parameters**
- Look up all PHY layer candidate parameters in the standard
- Use the index to find exact pages
- Budget: ~60-80 pages total (less if most parameters already have references)

**Agent V-B2: IEEE verification — MAC parameters**
- Look up all MAC layer candidate parameters
- Budget: ~60-80 pages total

**Agent V-B3: IEEE verification — Management and aggregation parameters**
- Look up management, aggregation, block ACK, and IBSS parameters
- Budget: ~60-80 pages total

**Agent V-B4: IEEE verification — MIMO/MCS and regulatory parameters**
- Look up MIMO, MCS, and regulatory-related parameters
- Budget: ~60-80 pages total

### Verification output merge

After both tracks complete, the orchestrator merges results using the context overflow prevention strategy:

1. Process Track A output files one at a time, extracting verification results to `log/phase3-accumulator.md`
2. Process Track B output files one at a time, augmenting the accumulator with IEEE references
3. Read the accumulator and produce `phase3-verified-parameters.md`
4. Generate `phase3-verified-parameters.xlsx`
5. Update `research-state.md` checkpoint

For each parameter, the verified output must include:
- Official IEEE parameter name (from Track B)
- Official IEEE description (from Track B)
- Section/page/table reference in the standard (from Track B)
- Confirmation that it applies to 802.11n (from Track A)
- Verified criticality (from Track A)
- Resolution notes for any conflicts between sources

---

## Phase 4 — Final report (main conversation or single agent)

Produce `parameters.md` with this structure:

### 1. Title and introduction
- Project objective
- Methodology summary (3 sources, multi-phase approach)

### 2. Summary of findings
- Total number of parameters identified
- Breakdown by layer (PHY, MAC, MAC management, Aggregation, MIMO/MCS, Regulatory)
- Breakdown by criticality (Mandatory vs Optimization)
- Key insights and notable findings

### 3. Parameter tables by layer
One summary table per layer with columns:
| Official IEEE Name | Common Name | Criticality | Default Value | Long-Distance Value | Standard Section | Page |

### 4. Detailed parameter sections by layer
For each layer, a subsection for every parameter containing the full metadata from `parameter-template.md`.

### 5. Conclusion
- Synthesis of findings
- Minimum viable parameter set for long-distance 802.11n
- Recommendations for deployment
- Identified gaps, uncertainties, or areas needing further research
- List of any skipped agents and their impact on completeness

Also produce `parameters.xlsx` with all parameters and full metadata.
