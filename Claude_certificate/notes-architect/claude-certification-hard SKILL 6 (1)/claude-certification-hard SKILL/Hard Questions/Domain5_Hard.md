You are an expert instructor for Domain 5 (Context Management & Reliability) of the CCFA exam — hard mode. This domain is worth 15% of the exam. All questions must be at real exam difficulty: scenario-anchored, four plausible distractors, no direct recall.

EXAM CONTEXT
Scenario-based multiple choice. Concepts in this domain cascade into Domains 1, 2, and 4. This domain appears across nearly all scenarios, particularly: Customer Support Resolution Agent, Multi-Agent Research System, Structured Data Extraction.

TASK STATEMENT REFERENCE

5.1 CONTEXT PRESERVATION
- Case facts block: persistent, prepended to every prompt, verbatim — NEVER summarise it
- Progressive summarisation trap: `$247.83 for order #8891` → "a refund was discussed"
- Lost in the middle: models process beginning and end reliably; middle may be missed
- Fix: place key summaries at START, use explicit section headers
- Trim verbose tool results to relevant fields BEFORE appending to context

5.2 ESCALATION AND AMBIGUITY
- Valid triggers: explicit human request (honour IMMEDIATELY), policy gap, inability to make progress
- Invalid triggers: sentiment/frustration alone, self-reported confidence scores
- Frustration nuance: acknowledge + offer resolution; only escalate if customer reiterates human preference
- Ambiguous customer match: ask for additional identifiers, never select by heuristic

5.3 ERROR PROPAGATION
- Structured error context: failure type + what was attempted + partial results + alternatives
- Anti-pattern 1: silent suppression → prevents any recovery
- Anti-pattern 2: workflow termination on single failure → discards partial results
- Access failure vs valid empty result: retry on access failure; do NOT retry on valid empty result

5.4 CODEBASE EXPLORATION
- Context degradation: agent references "typical patterns" instead of specific class names
- Fix: scratchpad files, subagent delegation, summary injection, `/compact`
- Crash recovery: each agent exports state to manifest file; coordinator loads on resume

5.5 HUMAN REVIEW AND CONFIDENCE
- 97% overall accuracy can hide 40% error rate on a specific document type
- Validate accuracy by document type AND field segment before automating
- Stratified random sampling of high-confidence extractions detects novel error patterns

5.6 INFORMATION PROVENANCE
- Each finding: claim + source URL + doc name + excerpt + publication date
- Two conflicting credible sources: annotate BOTH — do not arbitrarily select one
- Temporal awareness: different publication dates explain different numbers

---

## Hard Question Bank

---

### Q1 — Case facts block vs progressive summarisation (5.1)

**Scenario: Customer Support Resolution Agent.** An agent handles a refund dispute for order #9923: $312.47 charged on April 7th, customer claims they returned the item on April 9th. After 6 exchanges, the conversation history is 12,000 tokens. The agent's context management applies progressive summarisation to older messages. In the 8th exchange, the agent asks the customer to confirm the order number "so I can look up your return." The customer responds: "I've told you this four times — it's order #9923."

**What is the root cause and the correct fix?**

A. The conversation has grown too long for the model to maintain full coherence; the fix is to use `/compact` to compress the conversation history and free context space before continuing.

B. Progressive summarisation compressed the specific transactional facts (order #9923, $312.47, April 7th, April 9th return) into a vague summary ("customer has a refund dispute"); the fix is to extract these facts into a persistent case facts block that is prepended verbatim to every prompt, never summarised.

C. The agent should use `--resume` with a named session to preserve the full conversation history without compression, ensuring the original message containing the order number is always available.

D. The agent's context window is approaching capacity; the fix is to trim verbose tool results from the history (e.g., the full order record with 40 fields) to relevant fields only, freeing space for the conversation context.

**Correct: B.** Trap: `lost-in-middle` and progressive summarisation. Trimming tool results (D) and `/compact` (A) address context size but do not protect specific transactional values from summarisation. `--resume` (C) preserves history but does not prevent summarisation from occurring within that session. The case facts block is the specific fix for this specific problem.

---

### Q2 — Escalation: explicit request vs frustration (5.2)

**Scenario: Customer Support Resolution Agent.** A customer contacts support with a billing error: they were charged $89.99 for a subscription renewal they cancelled three weeks ago. Company policy covers this scenario — cancelled subscriptions renewed in error are fully refundable. The customer's tone across the first four exchanges is frustrated. In the fifth message, the customer writes: "This is ridiculous. I want someone who can actually fix this." The agent escalates to a human.

**Was the escalation correct?**

A. The escalation was correct; the phrase "I want someone who can actually fix this" is an explicit request for a human agent and must be honoured immediately without further investigation.

B. The escalation was incorrect; frustration alone is not a valid escalation trigger. The agent should have acknowledged the frustration and processed the $89.99 refund under the cancellation policy, which directly covers this scenario.

C. The escalation was correct; the customer's inability to get resolution across five exchanges indicates the agent cannot make meaningful progress, which is a valid escalation trigger.

D. The escalation was premature; the agent should first ask a clarifying question — "Would you like me to process the refund now, or would you prefer to speak with a human?" — before escalating, giving the customer the choice.

**Correct: A.** The phrase "I want someone who can actually fix this" is an explicit request for a human — this must be honoured immediately. Option B would be correct if the customer had only expressed frustration without requesting a human (frustration ≠ escalation trigger); but once the customer explicitly requests a human, immediate escalation is required. Option C misclassifies the situation as "inability to make progress" — the agent CAN make progress, but the customer has explicitly invoked their right to a human. Option D adding a clarifying question delays an explicit human request, which is incorrect.

---

### Q3 — Conflicting sources and provenance (5.6)

**Scenario: Multi-Agent Research System.** A research system gathers information about a company's carbon emissions reduction targets. The web search subagent retrieves two credible sources: a press release from March 2024 stating "40% reduction by 2030" and an annual sustainability report from November 2024 stating "35% reduction by 2030." Both sources are from the company's official communications channels. The synthesis agent must include the emissions target in the final report.

**What is the correct approach?**

A. Use the November 2024 figure (35%) as it is more recent and supersedes the March 2024 press release; annotate the report with a note that the earlier figure has been updated.

B. Flag the conflict, include both figures with their respective source documents, publication dates, and exact excerpts, and note that the different values likely reflect a target revision between March and November 2024.

C. Exclude the emissions target from the report entirely and note that conflicting sources prevent a reliable figure from being reported.

D. Average the two figures to produce a 37.5% estimate and annotate it as a "reconciled estimate" based on the two official sources.

**Correct: B.** Trap: `attribution-fix` — the correct approach when two credible sources conflict is to annotate BOTH with full provenance and let the consumer decide, rather than arbitrarily selecting one or fabricating a reconciled value. Option A selects one source arbitrarily (recency heuristic). Option C discards valid information. Option D fabricates a new value not present in either source.

---

### Q4 — Context degradation in codebase exploration (5.4)

**Scenario: Developer Productivity with Claude.** A developer uses Claude Code to investigate a performance regression in a large codebase. After 2 hours, the session has accumulated extensive discovery output from `Read`, `Grep`, and `Glob` tool calls. The developer asks Claude Code to compare the `CacheManager` class to the `SessionStore` class for consistency. Claude Code responds: "Both classes follow typical patterns for in-memory storage with TTL-based expiry." The `CacheManager` class discovered earlier in the session used a Redis-backed implementation with no TTL, not in-memory storage.

**What is the most accurate diagnosis of this behaviour?**

A. The model is hallucinating; it has no record of the `CacheManager` class in its context because the tool results from the earlier `Read` call were purged during automatic context compression.

B. The context has filled with verbose discovery output from multiple tool calls, causing the model to lose its grip on specific findings from earlier in the session and fall back to generic descriptions instead of recalling specific class details.

C. The `/compact` command was not run between the discovery phase and the analysis phase; running `/compact` now will restore the model's access to the specific details it found about `CacheManager`.

D. The `CacheManager` analysis was performed in a separate subagent and the results were not persisted to the main session; the fix is to re-run the `CacheManager` analysis in the main session.

**Correct: B.** Context degradation is the specific symptom: the model references "typical patterns" instead of specific class details found earlier — the classic sign that verbose tool output has pushed earlier findings toward the middle of the context where they are less reliably processed. Option A (purged context) is plausible but imprecise — the tool results may still be in context but be "lost in the middle." Option C (`/compact` now) compresses history but does not restore lost detail — it would make things more consistent but can't recover specifics already diluted. Option D introduces a subagent that wasn't mentioned.

---

### Q5 — Aggregate accuracy hiding segment errors (5.5 × 4.6)

**Scenario: Structured Data Extraction.** A team automates extraction of financial data from three document types: balance sheets (3,200 per month), income statements (1,800 per month), and cash flow statements (600 per month). After running a validation set of 500 documents, the overall extraction accuracy is 96.2%. The team decides to automate fully with no human review. Two weeks later, auditors flag 23 errors in cash flow statement extractions, including 4 where operating cash flow figures were swapped with investing cash flow figures.

**What was the flaw in the validation methodology?**

A. The 500-document validation set was too small; a sample of at least 2,000 documents would have produced a reliable accuracy estimate for all three document types.

B. The team validated overall accuracy across all document types without segmenting by document type; with only ~53 cash flow statements in a 500-document sample, the 96.2% overall accuracy masked errors concentrated in cash flow statements.

C. The team should have used stratified random sampling only on high-confidence extractions, not the full validation set; errors on low-confidence documents distorted the accuracy estimate.

D. The validation set was drawn from historical data; the audited errors occurred in newly formatted cash flow statements not represented in the training distribution, making the validation set unrepresentative.

**Correct: B.** The classic aggregate-accuracy-hiding-segment-errors trap. With 600 cash flow statements per month representing ~10.7% of volume, a 500-document sample yields ~53 cash flow statements — too few to detect a concentrated error rate. The fix is always to validate accuracy by document type AND field segment before automating. Option A (larger sample) helps but misses the segmentation requirement — even a 2,000-document sample would produce a misleading overall accuracy if not segmented. Option C misapplies stratified sampling. Option D introduces a distribution shift explanation not supported by the scenario.
