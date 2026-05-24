---
layout: "post"
title: "Reflection: 2026-05-22"
date: "2026-05-22"
domains:
  - "behavioral science"
  - "astronomy"
  - "mycology"
---
### What the run produced

Three messages delivered in short format (scheduled mode):

1. **Jolt** (behavioral science): MIT Sloan integrative experiment design paper showing punishment in public goods games swings from +43% to -44% depending on context. Published in Science. 2,500 prior papers couldn't resolve this because they studied factors in isolation.
2. **Lift** (astronomy): JWST discovery of galaxy XMM-VID1-2075, a non-rotating galaxy less than 2 billion years old with more stars than the Milky Way. Published in Nature Astronomy.
3. **Prompt** (mycology): SPUN's global assessment finding 70% of Earth's ecosystems have zero DNA sequencing data for arbuscular mycorrhizal fungi, despite these networks moving ~1 billion tons of carbon annually. Published in FEMS Microbiology Letters.

Token cost: ~27 tool calls across Raindrop MCP, WebSearch, web_fetch, bash, and file tools. High context consumption, particularly from raw HTML in web_fetch results.

### Skill execution analysis

#### Done well

- **Log handling.** read_log.py and append_log.py were used correctly. Blocklist and variety data were consumed and respected.
- **Domain variety.** Today's domains (behavioral science, astronomy, mycology) have zero overlap with yesterday's (neuroscience, materials science/chemistry, writing/epistemology). The 2-of-3 variety rule is satisfied cleanly.
- **Source quality.** All three sources are peer-reviewed papers in top-tier journals with specific, falsifiable claims and concrete numbers. Bullshit probability is well below threshold for all three.
- **Format and delivery.** Short format, three slots, saved to inspirations.md, log appended. Mechanics were correct.

#### Done poorly

1. **Reference files largely skipped.** Only 2 of 8 reference files were loaded (message-craft.md and bullshit-detection.md). The skill specifies that four-tier-filter.md, failure-modes.md, source-quality-by-domain.md, mechanisms-and-triggers.md, hedonic-adaptation-countermeasures.md, and examples-pass-fail.md should be loaded at specific workflow steps. The agent treated these as optional. Without four-tier-filter.md, the four-tier filtering step was performed informally in reasoning rather than explicitly, making it unverifiable.

2. **No explicit candidate scoring.** The skill specifies a four-tier filter with a composite score (Tier 2 criteria met + Tier 3 checks passed, minimum threshold 3). No candidate was explicitly scored against this rubric. The agent jumped from "here are candidates" to "here are my picks" with no documented elimination rationale.

3. **Source-type monoculture.** All three final sources are research papers. Yesterday also had 2/3 research papers. This means 5 of the last 6 messages link to journal articles. The hedonic-adaptation-countermeasures.md file (never loaded) likely addresses this, but the check was never performed.

4. **Raindrop lane produced zero final sources.** Three Raindrop bookmark searches (23 total results) yielded no final selections. Four MCP calls (probe + 3 searches) produced nothing delivered. This may be legitimate (all failed quality checks) or may indicate the agent defaulted to web search because it's easier to find impressive papers than to evaluate bookmarked material. Without filtering records, we cannot determine which.

5. **Action prompt slot routing is questionable.** The mycorrhizal fungi source (70% of ecosystems unsequenced, billion-ton carbon network invisible to science) evokes vastness and the unknown more strongly than it prompts action. The slot-specific requirement for the action prompt is "an autonomy-supporting next step grounded in a real exemplar" with a step the reader "could do tomorrow." Message 3's closing sentence ("reads less as a finding and more as an open invitation to look") is a meta-comment about the source rather than content from it. This violates the principle of letting the source do the work. The message lacks a concrete, reachable next step.

6. **Reframe lane was never attempted.** The skill describes three source lanes (Raindrop, external primary, reframe). The reframe lane, which often produces the most distinctive action prompts, was entirely skipped. This contributed to the research-paper monoculture.

### Message quality grades

- **Jolt: A-.** Strong schema disruption, excellent specificity, graspable in one read. Weakness: buries the most striking finding (communication was 3x more influential) at the end. Reordering to lead with that punchline would make the reframe land faster for a morning reader.
- **Lift: A.** The strongest message. Quiet, reverent, leads with vastness, no meta-commentary. The closing mechanism (counter-rotating collision) is a vivid image. No changes needed.
- **Prompt: B+.** Quality contract met. Good specificity (named researcher, named organization, exact numbers). Weakness: the closing sentence is editorial meta-commentary rather than an invitational next step. Should have included a concrete, reachable action (e.g., "SPUN maintains an open map of sampled regions" or a specific question the reader could sit with). The source fits the emotional lift slot better than the action prompt slot.

### Skill file (SKILL.md) improvements

These changes would prevent the execution failures observed in this run.

#### 1. Inline reference file loading at specific steps

Current: A reference table at the bottom says "Load these on demand during the workflow."

Problem: The agent interprets "on demand" as "optional" and skips most of them.

Fix: Replace the reference table with inline directives at each step. For example, Step 5 should say: "Load `references/four-tier-filter.md` now. For each candidate, evaluate against the four tiers in order." Step 4 should say: "Load `references/hedonic-adaptation-countermeasures.md` now. Compare the candidate pool against recent_variety."

#### 2. Explicit source-type variety check

Current: The variety check in Step 4 covers domains but not source types.

Problem: The agent produced three research papers in a row, continuing a multi-day trend.

Fix: Add to Step 4: "Check source_type distribution. No more than 2 of 3 slots may share a source type. If the last 6 entries have 4+ of the same source type, at least one slot today must use a different type." Add source_type to the variety summary output from read_log.py.

#### 3. Require documented filtering output

Current: Step 5 says "For each candidate, evaluate in order."

Problem: Evaluation happens in the agent's reasoning with no external record, making it unverifiable.

Fix: Add to Step 5: "For each candidate entering the filter, record in scratch space: source URL, Tier 1 pass/fail, Tier 2 criteria met, Tier 3 checks passed, Tier 4 pass/fail, composite score, disposition (selected/rejected + one-line reason). This record is not delivered to the reader but is required for skill improvement."

#### 4. Raindrop lane efficiency gate

Current: The skill says to pull 5-8 Raindrop candidates.

Problem: Three separate searches returned 23 results, none were selected, consuming 4 MCP calls.

Fix: Add: "If the first Raindrop search returns zero candidates that plausibly pass Tier 1, stop searching Raindrop and note the degradation. Do not run 3+ speculative searches." Also consider adding a longer-term solution: tag promising bookmarks with "inspiration-candidate" during regular review, so the daily skill queries a curated tag rather than doing open-ended search.

#### 5. Reframe lane rotation requirement

Current: The reframe lane is described as optional.

Problem: It was never attempted, contributing to source-type monoculture and weak action prompt.

Fix: Add: "At least one reframe-lane attempt every 3 runs. If the action prompt slot is unfilled after Raindrop and external primary filtering, attempt a reframe before reporting the slot unfilled. Track reframe attempts in the log."

#### 6. Action prompt slot validation

Current: The action prompt slot has craft guidelines but no validation checkpoint.

Problem: The delivered message lacks a concrete next step, the core requirement of the slot.

Fix: Add a checkpoint after Step 8: "For the action prompt, verify: (a) there is a specific practice or technique described, (b) there is a visible next step the reader could take tomorrow, (c) the framing uses invitational language only. If (a) or (b) fails, rewrite or replace the source."

### Infrastructure and tooling improvements

Ordered by implementation impact.

#### 1. Article extraction script (highest priority)

The single biggest token waste is raw HTML from web_fetch. The UC Davis page returned full site navigation, every academic department, and footer links. The actual article was maybe 10% of content returned.

Create `scripts/fetch_article.py` using `trafilatura` (or `readability-lxml` as fallback):

```
# Usage: python scripts/fetch_article.py <url>
# Returns JSON: {title, author, date, word_count, text_first_1500_words, status}
# Strips nav, footer, sidebars, ads
# Returns status: success | empty | error | timeout
```

Install trafilatura in the workspace. This script replaces raw web_fetch calls for article evaluation. Estimated savings: 50-70% of web_fetch token consumption.

#### 2. Batch search-and-extract script (high priority)

Collapse 8+ WebSearch calls and 3+ web_fetch calls into 1-2 bash calls:

```
# Usage: python scripts/search_sources.py --queries queries.json --blocklist blocklist.json --max-candidates 10
# Input: list of search queries with domain labels
# Process: run searches, deduplicate by domain, fetch top candidates via fetch_article.py, check against blocklist
# Output: JSON array of {url, title, domain, source_type, excerpt_300_words, fetch_status}
```

The LLM provides the search queries and evaluates the compact candidate list. The mechanical search/fetch/dedup work moves to a script. This collapses ~11 tool calls to 1-2 and halves token consumption.

Note: This requires WebSearch to be callable from bash or replaced with a direct search API. If WebSearch is only available as an MCP tool, the script handles the fetch-and-extract portion and the LLM still does the search calls, but now processes clean article text rather than raw HTML.

#### 3. Candidate tracking log (medium priority)

Add `candidate_log.jsonl` with one JSON line per candidate considered per run:

```json
{"date": "2026-05-22", "url": "...", "title": "...", "source_lane": "web_search", "query": "...", "tier1_pass": true, "tier2_criteria": ["schema_disruption", "vivid_specificity"], "score": 5, "disposition": "selected", "slot": "jolt"}
```

Extend append_log.py to accept candidate entries, or create a separate script. After 30 runs this data enables: search query effectiveness analysis, Raindrop lane yield tracking, filter calibration, and source type distribution monitoring.

#### 4. Reference file consolidation (medium priority)

The 8 reference files total perhaps 200-300 lines of actionable rules. The current selective-loading approach fails because the agent doesn't load enough of them. Two options:

- **Short-term:** Create `references/digest.md` that contains the actionable checklists from each file (stripped of background/rationale). Load this single file at the start of filtering. Estimated size: ~150 lines, a fraction of current context cost.
- **Long-term:** Integrate the checklists into the SKILL.md workflow steps directly, so they're loaded when the skill is loaded.

#### 5. Monthly archival for inspirations.md (low priority)

The file grows unboundedly. At month boundaries, move entries to `archive/YYYY-MM.md`. A small bash function at the start of each run handles this:

```bash
# If month changed since last entry, mv older entries to archive
```

#### 6. Quality metrics script (low priority)

Add `scripts/metrics.py` that reads inspiration-log.md and (if it exists) candidate_log.jsonl to report:

- Keeper rate (delivered / considered) per week.
- Domain frequency over rolling 30 days (catches over-representation).
- Source-type distribution over 30 days.
- Source lane yield (Raindrop vs. web vs. reframe).
- Average tool calls per delivery (efficiency trend).

Run on-demand or monthly. No need to automate alerts initially.

#### 7. Additional source integrations (low priority)

- **Semantic Scholar API:** Free, structured, returns abstracts and citation counts. Better than web-searching for academic sources.
- **arXiv API:** Same benefit for preprints.
- Both would be additional lanes in the batch search script.

### Implementation priority

| Priority | Item | Estimated effort | Impact |
|----------|------|-----------------|--------|
| 1 | Inline reference loading in SKILL.md | 30 min edit | Fixes filtering quality |
| 2 | fetch_article.py with trafilatura | 1-2 hours | Halves token waste from web_fetch |
| 3 | Source-type variety check in SKILL.md + read_log.py | 1 hour | Prevents research-paper monoculture |
| 4 | Candidate tracking log | 1 hour | Enables all future optimization |
| 5 | Action prompt validation checkpoint in SKILL.md | 15 min edit | Prevents weak action prompts |
| 6 | Raindrop efficiency gate in SKILL.md | 15 min edit | Stops wasting MCP calls |
| 7 | Batch search_sources.py script | 2-3 hours | Largest efficiency gain |
| 8 | Reference digest consolidation | 1 hour | Simplifies reference loading |
| 9 | Monthly inspirations.md archival | 30 min | Keeps file manageable |
| 10 | Quality metrics script | 2 hours | Detects long-term drift |

---
