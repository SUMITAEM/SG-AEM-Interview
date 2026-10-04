---
name: claude-certification-hard
description: Use when the user wants hard, real-exam-difficulty practice questions for the CCFA exam. All questions are scenario-anchored, multi-step, with four highly plausible distractors. No easy recall questions. Simulates sitting the actual exam.
---

# Claude Certified Architect (Foundations) — Hard Exam Simulation

## Language

**Default language is English.** Respond in English throughout the session.

---

## Exam Overview

| Field | Value |
|---|---|
| Questions | 60 |
| Time limit | 120 minutes |
| Passing score | 720/1,000 (scaled) |
| Format | Multiple choice; one correct answer, three plausible distractors |
| Structure | 4 scenarios drawn from a bank of 6 |

| Domain | Topic | Weight |
|---|---|---|
| 1 | Agentic Architecture & Orchestration | 27% |
| 2 | Tool Design & MCP Integration | 18% |
| 3 | Claude Code Configuration & Workflows | 20% |
| 4 | Prompt Engineering & Structured Output | 20% |
| 5 | Context Management & Reliability | 15% |

---

## The 6 Official Exam Scenarios

All questions are anchored in one of these 6 scenarios. Every question stem MUST name the scenario explicitly.

| # | Scenario | Canonical Tools / Context | Primary Domains |
|---|---|---|---|
| 1 | **Customer Support Resolution Agent** | `get_customer`, `lookup_order`, `process_refund`, `escalate_to_human` | D1, D2, D5 |
| 2 | **Code Generation with Claude Code** | CLAUDE.md configs, slash commands, plan mode | D3, D5 |
| 3 | **Multi-Agent Research System** | coordinator + web search + doc analysis + synthesis + report agents | D1, D2, D5 |
| 4 | **Developer Productivity with Claude** | `Read`, `Write`, `Bash`, `Grep`, `Glob` + MCP servers | D2, D3, D1 |
| 5 | **Claude Code for CI/CD** | `-p` flag, `--output-format json`, automated PR review | D3, D4 |
| 6 | **Structured Data Extraction** | JSON schemas, `tool_use`, validation-retry loops, `custom_id` | D4, D5 |

---

## Session Format

### On Session Start

1. Ask: **"Do you want to practice a specific domain, a specific scenario, or all domains?"**
2. Ask how many questions (default: 15)
3. Begin immediately

### Hard Mode Rules — ALWAYS ACTIVE in this skill

This skill runs in hard mode for every session. These rules cannot be turned off:

- **Zero direct recall questions.** Every question is scenario-based.
- **60% applied scenario, 40% multi-step reasoning.** No easy questions.
- **All four options must be initially plausible.** No obviously-wrong distractors. A test-taker who knows the domain superficially should be able to make a case for at least two options.
- **Every distractor must exploit a named exam trap** from the distractor bank below. Never invent generic wrong answers.
- **At least one distractor per question must be a partially-correct answer** that fails in a specific edge case or under a specific condition.
- **2 of every 15 questions must be cross-domain** — requiring reasoning across two domains simultaneously (e.g., tool_choice from D2 + context propagation from D5 in one scenario).
- **Every question stem must include specific metrics**: percentages, latency numbers, error rates, file counts, dollar amounts. No vague descriptions.
- **Every question must name the exact canonical scenario** in the first sentence.

### Question Format

Present ONE question at a time.

Structure:
1. **Scenario context** (2–4 sentences): name the canonical scenario, describe the production situation with specific metrics
2. **Question stem** (1 sentence): use patterns like "What is the most likely root cause?", "Which change would most effectively address this?", "What should the engineer do first?"
3. **Options A–D**: 1–2 sentences each, concrete — name the exact mechanism, flag, tool, or configuration. Each option must be specific enough that a reader can picture exactly what would be implemented.

Show question number and domain: e.g., **Question 4/15 — Domain 1 × Domain 5 (Scenario: Customer Support Resolution Agent)**

### Answer Distribution Rule

Actively rotate the correct answer letter. Over 15 questions: ~4 A, ~4 B, ~3 C, ~4 D. Never let the same letter be correct more than 2 times in a row. Before generating each question, check which letter you've used least recently.

### Multi-Step Elimination Pattern (use for 40% of questions)

For multi-step questions:
- The scenario describes a production failure or unexpected behaviour
- The question asks for root cause or best fix
- All four options target a different component of the system — each sounds like a credible culprit
- The correct answer requires the test-taker to trace the failure to its origin using the principle: **trace failures upstream, not downstream**
- The three wrong options are components that are downstream of the actual root cause

### After Each Answer

Show:
1. **CORRECT ✅** or **INCORRECT ❌**
2. **Why the correct answer is right** — 2–3 sentences, grounded in the specific concept, naming the exact mechanism
3. **Why each wrong answer is wrong** — one sentence per distractor naming the specific flaw or the exam trap it represents
4. **The exam trap this question tested** — one line naming the trap (e.g., "Trap: downstream agent blamed instead of coordinator decomposition")

### After All Questions

- Final score (X/Y)
- List of incorrectly answered questions grouped by domain
- Identify the two weakest domains
- Offer to re-drill those domains with 5 more hard questions each

---

## Domain Knowledge Reference

### Domain 1: Agentic Architecture & Orchestration (27%)

**Agentic loop termination:**
- Correct: check `stop_reason` — `"end_turn"` = done, `"tool_use"` = continue
- Anti-pattern: `response.content[0].type == "text"` — wrong because model can return text AND tool_use simultaneously
- Anti-pattern: parsing natural language ("I'm done") — ambiguous, unreliable
- Anti-pattern: arbitrary iteration cap as primary mechanism

**Multi-agent orchestration:**
- Hub-and-spoke: coordinator at centre, subagents never communicate directly
- Subagents do NOT inherit coordinator history — everything must be explicit in their prompt
- Narrow decomposition failure → root cause is always the coordinator's decomposition, NOT downstream agents
- Iterative refinement: coordinator evaluates synthesis output for gaps, re-delegates with targeted queries

**Subagent invocation:**
- Coordinator's `allowedTools` must include `"Task"` or subagent spawning fails entirely
- Parallel spawning: emit multiple Task tool calls in ONE coordinator response
- `fork_session`: independent branches from shared baseline — divergent approaches
- Context passing MUST include structured metadata (source URLs, doc names, page numbers)

**Workflow enforcement:**
- Prompt-based guidance = probabilistic (non-zero failure rate)
- Programmatic hooks/gates = deterministic
- Financial, security, compliance → programmatic. Style/formatting → prompt is acceptable.
- Programmatic prerequisite: blocks `process_refund` until `get_customer` returns verified customer ID

**SDK Hooks:**
- `PostToolUse`: fires AFTER tool execution, BEFORE model processes result → data normalisation
- Tool call interception: fires BEFORE execution → block or redirect
- PostToolUse = transform result in-flight; interception = prevent action entirely

**Task decomposition:**
- Fixed sequential (prompt chaining): predetermined steps, reliable, cannot adapt
- Dynamic adaptive: generates subtasks from runtime discoveries, flexible, less predictable
- Attention dilution: too many files in one pass → inconsistent depth → fix with per-file passes + cross-file integration pass

**Session management:**
- `--resume`: prior context still valid, files unchanged
- `fork_session`: divergent approaches from shared baseline
- Fresh start + summary injection: files changed, context stale
- After code changes: inform agent of SPECIFIC file changes, not full re-exploration

---

### Domain 2: Tool Design & MCP Integration (18%)

**Tool descriptions:**
- Descriptions ARE the tool selection mechanism — not supplementary
- Minimal descriptions → misrouting between similar tools
- Fix for misrouting: expand descriptions FIRST — not routing classifiers, not few-shot, not tool consolidation
- Good description: purpose + input format + example queries + edge cases + "use THIS vs THAT" boundaries + "when NOT to use"
- System prompt keyword conflicts can override well-written descriptions

**Error handling:**
- Transient: retry. Validation: fix input. Business: not retryable, alternative workflow. Permission: escalate.
- Three-field structured error: `errorCategory` + `isRetryable` boolean + human-readable description
- `isError: true` in MCP response signals tool-level failure to the agent
- **Critical**: access failure (tool couldn't reach source → consider retry) vs valid empty result (source reached, no matches → do NOT retry — this is a legitimate success)

**Tool distribution:**
- Optimal: 4–5 tools per agent, scoped to role
- `"auto"`: model decides whether to call a tool at all
- `"any"`: model MUST call a tool, chooses which
- `{"type": "tool", "name": "X"}`: force specific tool
- `"any"` ≠ `"auto"` — `"any"` guarantees a tool call; `"auto"` may return text

**MCP configuration:**
- `.mcp.json` in repo root → project-level, version-controlled, team-shared
- `~/.claude.json` → user-level, personal, NOT shared
- `${GITHUB_TOKEN}` syntax keeps credentials out of version control

---

### Domain 3: Claude Code Configuration & Workflows (20%)

**CLAUDE.md hierarchy:**
- User-level (`~/.claude/CLAUDE.md`): only you, not version-controlled, new teammates do NOT get it
- Project-level (`.claude/CLAUDE.md` or root `CLAUDE.md`): everyone, version-controlled
- Directory-level: applies only in that directory
- Exam trap: new team member not getting instructions → user-level config, not project-level

**Custom commands and skills:**
- `.claude/commands/` = project-scoped, shared via git
- `~/.claude/commands/` = personal, not shared
- `context: fork`: isolated sub-agent, verbose output stays contained
- `allowed-tools`: restricts tool access during skill execution
- Skills = on-demand. CLAUDE.md = always-loaded. Never mix these.

**Path-specific rules:**
- `.claude/rules/` files with `paths:` glob patterns in YAML frontmatter
- Key advantage: glob patterns activate across ENTIRE codebase, not just one directory
- Load ONLY when editing matching files → token efficiency

**Plan mode:**
- Use when: large-scale changes, multiple valid approaches, 45+ files, architectural decisions
- Direct execution when: clear single-file bug fix, simple validation
- `Explore` subagent: isolates verbose codebase discovery, prevents context bloat

**CI/CD:**
- `-p` flag: non-interactive mode. Without it, CI job hangs waiting for input.
- Correct flag is `-p`, NOT `--batch`, NOT `CLAUDE_HEADLESS=true`, NOT stdin redirect
- `--output-format json`: machine-parseable findings for automated CI
- Independent review instance (fresh session) catches more than same-session self-review

---

### Domain 4: Prompt Engineering & Structured Output (20%)

**Explicit criteria:**
- Specific categorical criteria beat vague confidence-based instructions
- High false positive rates in one category destroy trust in ALL categories
- Fix: temporarily disable high-FP categories while improving prompts
- Severity calibration requires actual CODE EXAMPLES, not prose

**Few-shot prompting:**
- Most effective technique for consistency — beats more instructions or confidence thresholds
- Use when: inconsistent formatting, inconsistent judgment on ambiguous cases, extraction misses existing info
- 2–4 examples, each showing REASONING for the choice over plausible alternatives

**Structured output with tool_use:**
- `tool_use` with JSON schema eliminates syntax errors
- `tool_use` does NOT prevent: semantic errors, field placement errors, fabrication
- Optional/nullable fields (`{"type": ["string", "null"]}`) prevent fabrication when data may be absent
- `strict: true`: enforces schema exactly — forces model to provide required field values

**Validation-retry loops:**
- Retry with: original document + failed extraction + specific validation error
- Effective for: format mismatches, structural errors, misplaced values
- NOT effective for: information genuinely absent from source
- Max 3 retry attempts

**Batch processing:**
- 50% cost savings, up to 24-hour window, no latency SLA
- Does NOT support multi-turn tool calling
- Synchronous: blocking workflows (pre-merge checks, developer waits)
- Batch: latency-tolerant workflows (overnight reports, weekly audits)

---

### Domain 5: Context Management & Reliability (15%)

**Context preservation:**
- Case facts block: persistent, prepended to every prompt, verbatim — never summarise
- Progressive summarisation trap: compresses `$247.83 for order #8891` → "a refund was discussed"
- Lost in the middle: place key summaries at START, use explicit section headers
- Trim verbose tool results before appending to context

**Escalation triggers:**
- Valid: explicit human request (honour IMMEDIATELY), policy gap, inability to make progress
- Invalid: sentiment/frustration alone, self-reported confidence scores
- Frustration nuance: acknowledge + offer resolution first; only escalate if customer reiterates human preference
- Ambiguous customer match: ask for additional identifiers, never select by heuristic

**Error propagation:**
- Structured error context: failure type + what was attempted + partial results + alternatives
- Anti-pattern 1: silent suppression → prevents recovery
- Anti-pattern 2: workflow termination on single failure → discards partial results

**Codebase exploration:**
- Context degradation symptom: agent references "typical patterns" instead of specific class names
- Fix: scratchpad files, subagent delegation, `/compact`, summary injection

**Human review and confidence:**
- 97% overall accuracy can hide 40% error rate on a specific document type
- Validate by document type AND field segment before automating
- Stratified random sampling of high-confidence extractions detects novel error patterns

**Information provenance:**
- Each finding: claim + source URL + doc name + excerpt + publication date
- Two conflicting credible sources: annotate BOTH — do not arbitrarily select one
- Temporal awareness: different publication dates explain different numbers

---

## Distractor Bank — Use These Patterns

Every distractor must map to one of these named traps:

| Trap Name | Situation | Correct Answer | Wrong Answer (distractor) |
|---|---|---|---|
| downstream-blame | Multi-agent coverage gap | Coordinator decomposition | Web search agent / synthesis agent / retrieval agent |
| loop-termination | Agent terminates prematurely | Check `stop_reason` | Check content type / add iteration cap / parse natural language |
| compliance-prompt | High-stakes compliance failure | Programmatic hook/gate | Enhanced system prompt / few-shot examples / routing classifier |
| misrouting-fix | Tool misrouting | Expand tool descriptions | Add routing classifier / merge tools / add few-shot |
| config-level | New team member not getting instructions | Move to project-level CLAUDE.md | Update user-level / add to skills / use /memory |
| ci-flag | CI pipeline hangs | Add `-p` flag | `--batch` / `CLAUDE_HEADLESS=true` / stdin redirect |
| empty-result | Tool returns empty array, agent retries | Valid empty result — do NOT retry | Retry with backoff / escalate / check permissions |
| attribution-fix | Synthesis report has no attribution | Fix context passing (structured metadata) | Fix synthesis agent prompt / fix web search agent / fix coordinator prompt |
| posttooluse-timing | PostToolUse timing question | Fires AFTER execution, BEFORE model processes | Fires before execution / fires after model responds / fires at loop end |
| any-vs-auto | tool_choice "any" vs "auto" | `"any"` forces a tool call; `"auto"` may return text | `"auto"` also forces a tool call |
| escalate-immediately | Customer explicitly requests human | Escalate immediately, no investigation | Investigate first / assess confidence / check sentiment |
| batch-blocking | Batch API for pre-merge check | Keep synchronous | Use batch with polling / add timeout fallback |
| batch-multiturn | Batch API multi-turn tool calling | Not supported | Fully supported / requires special flag |
| mcp-level | `.mcp.json` vs `~/.claude.json` | `.mcp.json` = project (shared); `~/.claude.json` = personal | Reversed / wrong filename |
| lost-in-middle | Long context reliability | Key summary at START + section headers | Larger context window / split into chunks / streaming |
| partial-correct | Any scenario | The mechanism that works in the normal case | The same mechanism that fails under the specific edge case described |

---

## Hard Question Bank

Reference these pre-written questions as templates for difficulty calibration. Generate new questions at the same difficulty level.

---

### D1-H1 — PostToolUse timing (Domain 1 × Domain 2)

**Scenario: Customer Support Resolution Agent.** The agent processes refunds across three regional MCP servers: EU, US-East, and US-West. Each server returns timestamps in a different format (ISO 8601, Unix epoch, and a locale string respectively). After deploying a `PostToolUse` hook to normalise all timestamps to ISO 8601, the team notices the normalisation is working for EU and US-East but the model is still receiving raw locale strings from US-West. Logs confirm the hook is firing for all three servers. The US-West MCP server has not been modified.

**What is the most likely root cause?**

A. The `PostToolUse` hook fires before the US-West tool executes, so the raw response bypasses normalisation and is written directly to context.

B. The hook is normalising US-West timestamps correctly, but the model is reading a cached version of the tool result from a previous turn before the hook ran.

C. The hook fires after tool execution but before the model processes the result; the US-West server is returning the locale string inside a nested field that the hook's field selector does not cover.

D. The hook is overwriting the normalised US-West value with the raw value because the hook runs twice per tool call — once for the result and once for the metadata envelope.

**Correct: C** — PostToolUse fires after execution and before model processing (correct timing), but a nested field the selector does not target passes through unchanged. Trap: option A inverts the hook's timing (fires-before-execution distractor). Option B introduces a non-existent caching mechanism. Option D invents a double-fire behaviour.

---

### D1-H2 — Coordinator decomposition failure (Domain 1)

**Scenario: Multi-Agent Research System.** A coordinator decomposes a query about "the regulatory landscape for autonomous vehicles" and assigns subtopics to four subagents: federal highway regulations, state-level legislation, EU framework directives, and insurance liability law. The synthesis agent produces a 12-page report. The client flags that the report is missing entirely: FAA drone-vehicle integration rules, NHTSA software safety standards, and UNECE WP.29 international harmonisation. All four subagents completed successfully with no errors.

**What is the root cause?**

A. The synthesis agent did not request a second pass from the web search subagent to fill coverage gaps identified during report assembly.

B. The web search subagent used keyword-based queries that did not surface regulatory documents from specialised agencies, returning only general news articles.

C. The coordinator's initial decomposition defined subtopics too narrowly, omitting the regulatory bodies (FAA, NHTSA, UNECE) whose scope overlaps with autonomous vehicles.

D. The document analysis subagent filtered out documents from non-legislative sources, excluding agency technical standards from its output.

**Correct: C** — Downstream agents executed correctly within their assigned scope; the gap is in what the coordinator chose to decompose into. Trap: downstream-blame applied to all three wrong options — each blames a subagent rather than the upstream decomposition.

---

### D2-H1 — Access failure vs valid empty result (Domain 2)

**Scenario: Customer Support Resolution Agent.** A `lookup_order` tool queries the order database for orders placed in the last 90 days. For customer #4471, it returns an empty array. The agent retries the call three times with exponential backoff, then escalates to a human agent with the note "unable to retrieve order history." The human agent checks manually and confirms the customer has no orders in the last 90 days — they last ordered 14 months ago.

**Which change would most directly prevent this incorrect escalation in future?**

A. Increase the `lookup_order` tool's retry limit from 3 to 5 attempts before escalating, giving transient failures more recovery opportunities.

B. Add a secondary `lookup_order_extended` tool that queries the full order history when the 90-day window returns empty, so the agent has more data before deciding.

C. Update the `lookup_order` tool to return a structured response that distinguishes between an access failure (`isRetryable: true`) and a valid empty result (`isRetryable: false`, `resultType: "no_records_found"`), and update the agent to not retry on `no_records_found`.

D. Implement a `PostToolUse` hook that inspects empty array results and automatically retries with an extended date range before the model processes the response.

**Correct: C** — The tool must signal the difference between a retrieval failure and a legitimate empty result; the agent's retry logic depends on this distinction. Trap: options A and D both increase retrying (wrong direction). Option B adds a second tool but doesn't fix the agent's inability to distinguish failure from empty.

---

### D3-H1 — CLAUDE.md hierarchy (Domain 3)

**Scenario: Developer Productivity with Claude.** A team of six engineers uses Claude Code daily on a shared monorepo. The tech lead added API naming conventions, test file structure requirements, and PR description templates to their personal `~/.claude/CLAUDE.md` nine months ago. Three engineers who joined in the last four months consistently produce code that violates these conventions. The three original engineers follow them perfectly. All six engineers are working from the same repo on the same branch.

**What is the root cause and the correct fix?**

A. The new engineers have not run `/memory` to load the team's CLAUDE.md into their session context; the fix is to add `/memory` to the team onboarding documentation.

B. The conventions are in the tech lead's user-level `~/.claude/CLAUDE.md`, which is not version-controlled and not shared via git; the fix is to move them to a project-level `.claude/CLAUDE.md` committed to the repo.

C. The new engineers' local Claude Code installations have a different default model than the original engineers, causing different instruction-following behaviour; the fix is to pin the model version in a project-level config file.

D. The conventions conflict with the new engineers' personal `~/.claude/CLAUDE.md` files, which are overriding the team instructions; the fix is to ask each engineer to delete their personal CLAUDE.md.

**Correct: B** — User-level config is personal and never shared via git; the fix is always to move team standards to project-level. Trap: option A correctly names `/memory` but applies it to the wrong problem. Option D diagnoses a conflict that isn't described in the scenario.

---

### D3-H2 — `-p` flag in CI (Domain 3)

**Scenario: Claude Code for CI/CD.** A team integrates Claude Code into their GitHub Actions pipeline to automatically review pull requests. The workflow runs `claude --output-format json "Review this PR for security vulnerabilities"` as a pipeline step. In testing, the step runs successfully on the engineer's laptop. In CI, the step hangs indefinitely and the pipeline times out after 6 hours without producing output.

**What is the correct fix?**

A. Add `CLAUDE_HEADLESS=true` as an environment variable in the GitHub Actions workflow configuration to suppress interactive prompts.

B. Redirect stdin from `/dev/null` in the command: `claude --output-format json "..." < /dev/null` to signal that no interactive input is available.

C. Add the `-p` flag to the command: `claude -p --output-format json "..."` to run Claude Code in non-interactive mode.

D. Add `--batch` to the command: `claude --batch --output-format json "..."` to enable headless batch processing mode.

**Correct: C** — `-p` is the correct flag for non-interactive mode. Without it, Claude Code waits for interactive input. Trap: options A, B, and D are the three specific wrong-flag distractors the exam uses — all sound plausible to someone who hasn't memorised the correct flag.

---

### D4-H1 — What tool_use does NOT prevent (Domain 4)

**Scenario: Structured Data Extraction.** An engineering team builds an invoice extraction pipeline using `tool_use` with a strict JSON schema. The schema defines `invoice_total`, `line_items` (array), and `tax_rate` as required fields. After processing 500 invoices, they audit a sample and find: (1) zero JSON syntax errors, (2) 23 invoices where `line_items` sum to a different value than `invoice_total`, and (3) 11 invoices where the model populated `invoice_total` with the subtotal figure from the invoice rather than the grand total.

**Which statement correctly characterises what tool_use with a strict schema guarantees and what it does not?**

A. Tool_use with a strict schema guarantees syntactic validity and semantic correctness; the errors in findings (2) and (3) indicate the schema definition is missing validation constraints that should enforce sum consistency.

B. Tool_use with a strict schema guarantees only that the output is syntactically valid JSON matching the schema structure; it does not prevent semantic errors (finding 2) or field placement errors where a value is extracted from the wrong location in the source (finding 3).

C. Tool_use with a strict schema guarantees syntactic validity; finding (2) is a schema design flaw (missing sum constraint), while finding (3) is a prompt engineering failure that few-shot examples would resolve.

D. Tool_use with a strict schema prevents all three finding categories when `strict: true` is set in the tool definition; the presence of errors (2) and (3) indicates `strict: true` was not applied.

**Correct: B** — tool_use eliminates syntax errors but cannot enforce semantic consistency or prevent the model from reading the wrong field. Trap: option A incorrectly claims tool_use guarantees semantic correctness. Option D misattributes `strict: true` as preventing semantic and placement errors (it only enforces required fields). Option C is partially correct but mischaracterises finding (3) as a prompt issue when it's a semantic extraction error tool_use cannot catch.

---

### D5-H1 — Escalation triggers (Domain 5)

**Scenario: Customer Support Resolution Agent.** A customer contacts support about an incorrect charge of $34.99 on their account. The agent identifies this as a known billing system error affecting accounts created before March 2024, and the refund policy covers it directly — no exceptions needed. The customer's messages become increasingly frustrated over three exchanges. In the fourth message, the customer writes: "I've been dealing with this for 20 minutes. Can you please just fix this?" The agent escalates to a human.

**Was this escalation correct, and what is the right behaviour?**

A. The escalation was correct because the customer's sustained frustration across four exchanges indicates the agent is failing to make meaningful progress, which is a valid escalation trigger.

B. The escalation was incorrect; the customer asked the agent to "fix this," which is a request to resolve the issue, not an explicit request for a human agent. The agent should acknowledge the frustration and immediately process the $34.99 refund under the applicable policy.

C. The escalation was correct because the agent's confidence in diagnosing the billing error without human verification is a self-reported confidence score, which is not a reliable escalation trigger.

D. The escalation was incorrect; however, the agent should ask a clarifying question ("Would you prefer I process the refund now, or would you like to speak with a human agent?") before taking action.

**Correct: B** — "Can you please just fix this" is a request to resolve, not an explicit request for a human. The issue is within policy and solvable. The agent should acknowledge frustration and resolve. Trap: option A misclassifies frustration as "inability to make progress" — the agent CAN resolve it. Option C inverts the confidence score trap (confidence scores are unreliable triggers; absence of confidence is not a trigger either). Option D adds unnecessary friction when the path forward is clear.

---

### D1×D5-H1 — Cross-domain: context passing + case facts (Domain 1 × Domain 5)

**Scenario: Multi-Agent Research System.** A coordinator assigns a web search subagent to find statistics on electric vehicle adoption in Southeast Asia. The subagent returns a 4,200-word result containing market figures, country breakdowns, and analyst quotes. The coordinator appends the full result to the synthesis agent's prompt. After three more research phases, the synthesis agent's context is 87% full. In the final report, the synthesis agent attributes two of the Southeast Asia statistics to a European market report from a different research phase.

**What is the primary architectural failure and the correct fix?**

A. The synthesis agent's context is too large; the fix is to increase the context window size or switch to a model with a larger context budget.

B. The coordinator appended the full 4,200-word subagent result instead of a structured summary of key facts with source metadata; trimming verbose tool results to relevant fields with claim-source mappings before appending would preserve attribution and reduce context load.

C. The web search subagent returned results without source URLs attached to each statistic; the fix is to update the subagent's system prompt to require inline citation after every figure.

D. The synthesis agent should run a second cross-reference pass using a `verify_fact` tool before finalising the report; the missing tool in the synthesis agent's `allowedTools` is the root cause of the attribution errors.

**Correct: B** — Appending 4,200 words fills context and dilutes attribution. The fix is trimming to structured key facts with metadata before appending (D5: tool result trimming + D1: structured context passing). Trap: option A treats context exhaustion as the root cause rather than a symptom of not trimming. Option C is partially correct (subagent should output structured data) but misplaces the fix in the subagent prompt rather than the coordinator's handling of the result. Option D adds a tool that doesn't address the root cause.
