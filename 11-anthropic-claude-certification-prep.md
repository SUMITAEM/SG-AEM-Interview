# Anthropic Claude Certification Prep — Complete Study Notes

> **Source:** All content sourced from official Anthropic documentation (platform.claude.com/docs) and Anthropic research publications. Last updated September 2026.

> **Note:** As of September 2026, Anthropic offers free courses through **Claude Academy** (academy.claude.com) including "AI Fluency: Frameworks and Foundations." This guide covers all foundational knowledge areas for any Claude-focused assessment.

---

## Part 1: Anthropic & Claude Overview

### What is Anthropic?

Anthropic is an AI safety company founded in 2021. Their mission is to build AI systems that are safe, beneficial, and understandable. They develop the Claude family of large language models (LLMs).

**Core belief:** AI's impact could rival the industrial revolution, potentially arriving within a decade, but "we aren't confident it will go well."

### What is Claude?

Claude is a family of state-of-the-art large language models developed by Anthropic. Claude excels at tasks involving:
- Language understanding and generation
- Reasoning and analysis
- Coding and software development
- Multilingual tasks
- Vision (image understanding)
- Tool use (function calling)

### Key Products & Platforms

| Product | Description |
|---------|-------------|
| **claude.ai** | Consumer chat interface for interacting with Claude |
| **Claude API** | Developer API for building applications with Claude |
| **Claude Console** | Developer playground for testing prompts and API calls |
| **Claude Code** | CLI tool for coding assistance in the terminal |
| **Claude Academy** | Free educational courses at academy.claude.com |
| **Claude Managed Agents** | Pre-built agent harness running on Anthropic infrastructure |

---

## Part 2: Claude Model Family — Deep Dive

### Current Model Lineup (September 2026)

| Model | API ID | Context Window | Max Output | Input $/MTok | Output $/MTok | Speed | Default Effort |
|-------|--------|---------------|------------|-------------|--------------|-------|---------------|
| **Claude Fable 5.1** | `claude-fable-5-1` | 1M tokens | 128K tokens | $10 | $50 | Slower | high |
| **Claude Opus 5.5** | `claude-opus-5-5` | 1M tokens | 128K tokens | $4 | $20 | Moderate | medium |
| **Claude Sonnet 5** | `claude-sonnet-5` | 1M tokens | 128K tokens | $2 | $10 | Fast | high |
| **Claude Haiku 4.5** | `claude-haiku-4-5-20251001` | 200K tokens | 64K tokens | $1 | $5 | Fastest | Not supported |

### Model Selection Guide

```
Need most capable reasoning / multi-day agentic tasks?
  → Claude Fable 5.1

Complex coding & long-running agent sessions?
  → Claude Opus 5.5

Everyday tasks with best speed-to-intelligence ratio?
  → Claude Sonnet 5

High-volume, lowest cost, fastest responses?
  → Claude Haiku 4.5

Unsure where to start?
  → Claude Opus 5.5 (recommended default)
```

### Key Model Facts to Remember

- **All current models** support: text + image input, text output, multilingual capabilities, vision, and tool use
- **Context window:** 1M tokens ≈ 555,000 words or 2.5M Unicode characters (on the new tokenizer from Claude 4.7+)
- **Haiku 4.5's 200K tokens** ≈ 150,000 words
- **Tokenizer change:** Claude 4.7+ uses a newer tokenizer producing ~30% more tokens for the same text
- **Knowledge cutoffs:** Fable 5.1 and Opus 5.5 have reliable knowledge through June 2026; Sonnet 5 through January 2026; Haiku 4.5 through February 2025
- **Adaptive thinking** is the only thinking mode on Claude 4.7+ models
- **Effort parameter** is not supported on Haiku 4.5

### Thinking Modes

| Mode | Description | Models |
|------|------------|--------|
| **Adaptive Thinking** | Claude dynamically decides when and how much to think. Steered via the `effort` parameter. | Claude 4.7+ (always on for Opus 5.5, Sonnet 5, Fable 5.1) |
| **Extended Thinking** | Manual mode: set `thinking.type = "enabled"` + `budget_tokens`. Deprecated on Opus 4.6 and Sonnet 4.6. | Haiku 4.5 and older models |

**Effort Levels:**
- `low` — Minimal thinking, fastest responses
- `medium` — Balanced (default for Opus 5.5)
- `high` — Deep reasoning (default for Fable 5.1 and Sonnet 5)

**Key:** Thinking tokens are billed as **output tokens** and count toward `max_tokens`.

### Legacy / Retired Models

- Claude Fable 5, Claude Mythos 5, Claude Opus 5, Claude Opus 4.8, Claude Opus 4.7, Claude Opus 4.6, Claude Opus 4.5, Claude Sonnet 4.6, Claude Sonnet 4.5 — still available
- Claude Opus 4.1, Claude Opus 4, Claude Sonnet 4, Claude Haiku 3.5 — retired (may remain on Bedrock/Google Cloud)

---

## Part 3: Platform Availability & Deployment

### Five Platforms for Claude

| Platform | Operator | Description |
|----------|----------|-------------|
| **Claude API** | Anthropic (first-party) | Direct API access, full feature set |
| **Amazon Bedrock** | AWS-operated | Claude through AWS, AWS billing |
| **Claude Platform on AWS** | Anthropic-operated on AWS | Anthropic-run but billed via AWS Marketplace |
| **Google Cloud (Vertex AI)** | Google-operated | Claude through GCP |
| **Microsoft Foundry** | Anthropic-operated on Azure | Anthropic-run, billed via Azure Marketplace |

### Feature Availability Classifications

| Classification | Meaning |
|---------------|---------|
| **Stable** | Fully supported, production-ready, standard API versioning |
| **Beta** | Preview for gathering feedback, may change significantly or be discontinued |
| **Deprecated** | Still functional but no longer recommended, migration path provided |
| **Retired** | No longer available |

### Claude Consumption Units (CCUs)

Used on Claude Platform on AWS and Microsoft Foundry:
- **1 CCU = $0.01 USD**
- Token usage rated at standard per-model rates, then converted to CCUs
- Billed hourly via marketplace, monthly invoices
- Postpaid (arrears) only, no prepaid credits

---

## Part 4: Pricing — Complete Reference

### Base Token Pricing

| Model | Input | Output | Batch Input (50% off) | Batch Output (50% off) |
|-------|-------|--------|----------------------|----------------------|
| Fable 5.1 | $10/MTok | $50/MTok | $5/MTok | $25/MTok |
| Opus 5.5 | $4/MTok | $20/MTok | $2/MTok | $10/MTok |
| Sonnet 5 | $2/MTok | $10/MTok | $1/MTok | $5/MTok |
| Haiku 4.5 | $1/MTok | $5/MTok | $0.50/MTok | $2.50/MTok |

> **MTok** = Million tokens. "$5 / MTok" means $5 for every 1,000,000 tokens.

### Prompt Caching Pricing

| Operation | Multiplier | Duration |
|-----------|-----------|----------|
| **5-minute cache write** | 1.25x base input | 5 minutes |
| **1-hour cache write** | 2x base input | 1 hour |
| **Cache read (hit) — standard** | 0.1x (10%) base input | Same as preceding write |
| **Cache read — Fable 5.1** | 0.025x (2.5%) base input | Same as preceding write |
| **Cache read — Opus 5.5** | 0.05x (5%) base input | Same as preceding write |

**Break-even:** 5-minute cache pays off after just **1 cache read**. 1-hour cache pays off after **2 cache reads**.

Two ways to enable:
1. **Automatic caching** — Add one `cache_control` field at top level (recommended)
2. **Explicit cache breakpoints** — Place `cache_control` on individual content blocks

### Data Residency Pricing

- `inference_geo: "us"` → **1.1x multiplier** on ALL token categories
- `inference_geo: "global"` (default) → standard pricing
- Available on Claude 4.6+ models only

### Fast Mode Pricing (Research Preview)

| Model | Input | Output |
|-------|-------|--------|
| Opus 5.5 | $8/MTok | $40/MTok |
| Opus 5 / Opus 4.8 | $10/MTok | $50/MTok |

Fast mode stacks with caching and data residency multipliers.

### Tool-Specific Pricing

| Tool | Additional Cost |
|------|----------------|
| **Web Search** | $10 per 1,000 searches + standard tokens |
| **Web Fetch** | No additional cost (tokens only) |
| **Code Execution** | Free with web search/fetch; otherwise $0.05/hr per container after 1,550 free hours/month |
| **Computer Use** | Standard tool use tokens + ~4,500 input tokens overhead per request |
| **Browser Use** | Standard tool use tokens + ~6,600 input tokens overhead per request |

### Managed Agents Pricing

- **Tokens:** Same rates as Messages API
- **Session runtime:** $0.08 per session-hour (metered to the millisecond, only while `running`)
- **No** Batch API discount (sessions are interactive)

### Cost Optimization Strategies

1. **Choose the right model** — Haiku for simple tasks, Sonnet for most production, Opus for complex reasoning
2. **Implement prompt caching** — Reduces costs for repeated context
3. **Use Batch API** — 50% off for non-time-sensitive workloads
4. **Monitor usage** — Track token consumption patterns
5. **Combine discounts** — Batch + prompt caching stack together

---

## Part 5: API Fundamentals — Messages API

### Request Structure

```json
{
  "model": "claude-opus-5-5",        // REQUIRED: model ID
  "max_tokens": 1024,                // REQUIRED: max output tokens
  "messages": [                       // REQUIRED: conversation messages
    {
      "role": "user",
      "content": "Hello, Claude!"
    }
  ],
  "system": "You are a helpful assistant.",  // Optional: system prompt
  "tools": [...],                     // Optional: tool definitions
  "tool_choice": {"type": "auto"},    // Optional: tool use control
  "temperature": 0.7,                 // Optional: randomness (0-1)
  "stop_sequences": ["END"],          // Optional: custom stop strings
  "thinking": {"type": "adaptive"},   // Optional: thinking config
  "stream": true                      // Optional: enable streaming
}
```

### Message Roles

| Role | Purpose |
|------|---------|
| `user` | Messages from the user |
| `assistant` | Claude's responses (used in multi-turn for conversation history) |

**System prompt** is set via the `system` parameter (NOT as a message role).

### Content Block Types

| Type | Direction | Description |
|------|-----------|-------------|
| `text` | Request & Response | Plain text content |
| `image` | Request | Image content (base64, URL, or file_id) |
| `tool_use` | Response | Claude wants to call a tool |
| `tool_result` | Request | Results of a tool execution |
| `thinking` | Response | Claude's reasoning (with adaptive/extended thinking) |
| `document` | Request | PDF or other document content |

### Stop Reasons

| Stop Reason | Meaning |
|------------|---------|
| `end_turn` | Claude finished naturally |
| `max_tokens` | Hit the max_tokens limit |
| `stop_sequence` | Encountered a custom stop sequence |
| `tool_use` | Claude wants to call one or more tools |

### Authentication

- **API Key:** Set via `x-api-key` header or `ANTHROPIC_API_KEY` environment variable
- **API Version:** Set via `anthropic-version` header (e.g., `2023-06-01`)

**Best Practices:**
- Set expiration when creating API keys
- Keep keys out of source control
- Never put keys in client-side code or prompts
- Consider Workload Identity Federation as alternative to static keys

### Rate Limits & Usage Tiers

| Tier | Description |
|------|-------------|
| **Start** | Entry-level limits for getting started |
| **Build** | Increased limits for growing applications |
| **Scale** | Highest standard limits for production |

### SDK Support

Official SDKs available in:
- Python
- TypeScript / JavaScript
- C#
- Go
- Java
- PHP
- Ruby

All SDKs support streaming, tool use, and thinking features.

---

## Part 6: Tool Use (Function Calling) — Deep Dive

### How Tool Use Works

1. You define tools with names, descriptions, and input schemas
2. Claude determines when to call a tool based on the user's request
3. Claude returns a `tool_use` content block with the tool name and arguments
4. Your code executes the tool and returns a `tool_result` content block
5. Claude uses the result to formulate its response

### Tool Categories

#### Client-Side Tools (YOU execute)

**User-Defined Tools:**
- You write the JSON schema (name, description, input_schema)
- You implement the execution logic
- Full flexibility — any function you can write

**Anthropic-Schema Client Tools:**

| Tool | Purpose |
|------|---------|
| **Bash** | Execute shell commands in a persistent session |
| **Text Editor** | View and modify text files |
| **Computer Use** | Control desktop (screenshots, mouse, keyboard) |
| **Browser Use** | Navigate, read, interact with webpages |
| **Memory** | Store and retrieve information across conversations |

#### Server-Side Tools (ANTHROPIC executes)

| Tool | Purpose | Cost |
|------|---------|------|
| **Web Search** | Search the web with cited sources | $10/1,000 searches |
| **Web Fetch** | Retrieve full content from web pages/PDFs | No extra cost |
| **Code Execution** | Run Python/bash in sandboxed container | Free with web search/fetch |
| **Advisor** | Consult higher-intelligence model mid-generation | Token costs |
| **Tool Search** | Dynamic tool discovery at scale (BM25 + regex) | Token costs |

### Tool Definition Schema

```json
{
  "name": "get_weather",
  "description": "Get the current weather for a given location.",
  "input_schema": {
    "type": "object",
    "properties": {
      "location": {
        "type": "string",
        "description": "City and state, e.g. San Francisco, CA"
      }
    },
    "required": ["location"]
  }
}
```

### Tool Choice Modes

| Mode | Behavior |
|------|----------|
| `auto` | Claude decides whether to call a tool (default) |
| `any` | Claude MUST call some tool |
| `tool` | Claude MUST call a specific named tool |
| `none` | No tool calls allowed |

**Additional option:** `disable_parallel_tool_use: true` — limits to one tool call per turn.

### Strict Tool Use

Add `strict: true` to tool definitions to guarantee Claude's tool call arguments always conform to your JSON schema exactly. Useful for production pipelines requiring validated inputs.

### Tool Use Loop (Agentic Pattern)

```
User Request → Claude Thinks → tool_use block(s) → Your Code Executes
→ tool_result block(s) → Claude Thinks Again → Final Response (or more tool calls)
```

### MCP Connector

**Model Context Protocol (MCP)** — Connect to remote MCP servers directly from the Messages API without a separate client. Enables standardized tool discovery and orchestration at scale.

---

## Part 7: Key Features & Capabilities

### Vision (Image Understanding)

**Supported formats:** JPEG, PNG, GIF, WebP
**Image sources:** Base64, URL, Files API (`file_id`)

| Limit | Value |
|-------|-------|
| Max per message (claude.ai) | 20 images |
| Max per API request (200K context) | 100 images |
| Max per API request (other models) | 600 images |
| Max dimensions | 8000 x 8000 px |
| Max file size (API) | 10 MB |
| Max file size (Bedrock/Google Cloud) | 5 MB |

**Token cost:** Each 28x28 pixel patch = 1 visual token. Cost = ⌈width/28⌉ × ⌈height/28⌉ visual tokens.

**Resolution tiers:**
- **High-resolution** (Claude 4.7+): Max 2576px long edge, up to 4784 visual tokens
- **Standard** (older models): Max 1568px long edge, up to 1568 visual tokens

**Best practice:** Place images BEFORE text in prompts for best results.

**Limitations:**
- Cannot identify/name people in images
- Cannot generate, edit, or create images
- May hallucinate on low-quality, rotated, or very small images (<200px)
- Approximate spatial reasoning and counting
- Cannot determine if an image is AI-generated

### PDF Support

Process and analyze both text and visual content from PDF documents. ZDR-eligible.

### Structured Outputs

Two approaches to guarantee schema conformance:
1. **JSON outputs** — For structured data responses
2. **Strict tool use** (`strict: true`) — For validated tool inputs

### Citations

Ground responses in source documents with exact references to sentences and passages. Enables verifiable, trustworthy outputs. ZDR-eligible.

### Search Results

Enable natural citations for RAG applications. Provide search results with proper source attribution for web search-quality citations from custom knowledge bases.

### Prompt Caching

Reuse previously processed prompt prefixes across API calls:
- **Automatic caching:** Single `cache_control` field, system manages breakpoints
- **Explicit breakpoints:** Fine-grained control over what gets cached
- Two durations: 5-minute and 1-hour

### Batch Processing

Process large request volumes asynchronously:
- **50% discount** on input and output tokens
- Results delivered asynchronously
- Not available with Fast Mode or Managed Agents

### Context Windows

Up to **1M tokens** for processing large documents, codebases, and conversations:
- 1M tokens ≈ 555,000 words (new tokenizer)
- Haiku 4.5 limited to 200K tokens

### Context Management Features

| Feature | Description |
|---------|-------------|
| **Compaction at token threshold** | Auto-summarizes earlier conversation when tokens reach trigger |
| **Context editing** | Clear tool results when approaching limits, manage thinking blocks |
| **Automatic prompt caching** | Simplified caching with single API parameter |
| **Token counting** | Determine token count before sending to Claude |

### Files API

Upload and manage files (PDFs, images, text files) to use with Claude without re-uploading with each request. Not ZDR-eligible.

### Effort Parameter

Controls how many tokens Claude uses when responding:
- Trades off between thoroughness and token efficiency
- Default varies by model (Fable 5.1: high, Opus 5.5: medium, Sonnet 5: high)
- Not supported on Haiku 4.5

### Data Residency

Control where model inference runs:
- `"global"` — Default, standard pricing
- `"us"` — US-only inference, 1.1x pricing multiplier
- Available on Claude 4.6+ models

---

## Part 8: Prompt Engineering — Comprehensive Guide

### General Principles

#### 1. Be Clear and Direct

Claude responds best to explicit instructions. Think of Claude as a brilliant but new employee who lacks context on your norms.

**Golden Rule:** Show your prompt to a colleague with minimal context. If they'd be confused, Claude will be too.

```
Less effective: "Create an analytics dashboard"

More effective: "Create an analytics dashboard. Include as many relevant
features and interactions as possible. Go beyond the basics to create a
fully-featured implementation."
```

#### 2. Add Context and Motivation

Explaining WHY improves results. Claude is smart enough to generalize from explanations.

```
Less effective: "NEVER use ellipses"

More effective: "Your response will be read aloud by a text-to-speech
engine, so never use ellipses since the engine won't know how to
pronounce them."
```

#### 3. Use Examples (Few-Shot Prompting)

Examples are one of the most reliable ways to steer output. Include **3-5 examples** for best results.

Make examples:
- **Relevant** — Mirror your actual use case
- **Diverse** — Cover edge cases, vary enough to avoid unintended patterns
- **Structured** — Wrap in `<example>` tags (multiple in `<examples>`)

#### 4. Structure Prompts with XML Tags

XML tags help Claude parse complex prompts unambiguously:
- `<instructions>` — What to do
- `<context>` — Background information
- `<input>` — Variable data
- `<example>` — Examples
- `<output>` — Expected format

Best practices:
- Use consistent, descriptive tag names
- Nest tags when content has natural hierarchy
- Combine tags within `<document index="n">` for multiple documents

#### 5. Give Claude a Role

Set a role in the **system prompt** to focus behavior and tone:

```json
{
  "system": "You are a helpful coding assistant specializing in Python.",
  "messages": [...]
}
```

Even a single sentence makes a difference in response quality.

### Advanced Techniques

#### Chain of Thought / Thinking

Configure Claude to reason step-by-step before answering:

```json
{
  "thinking": {
    "type": "adaptive",
    "display": "summarized"
  }
}
```

**Key points:**
- Thinking tokens are billed as output tokens
- Thinking blocks contain a `signature` field for multi-turn conversations
- `display: "summarized"` returns visible reasoning; `"omitted"` hides it
- On Opus 5.5, Sonnet 5, Fable 5.1: thinking is always on (no config needed)
- On Opus 4.8, 4.7, 4.6, Sonnet 4.6: must set `thinking: {type: "adaptive"}`

#### Prompt Chaining

Break complex tasks into sequential sub-tasks:
1. Each prompt focuses on one aspect
2. Output of step N becomes context for step N+1
3. Reduces complexity per step, improves accuracy

**Use when:** Task has multiple distinct phases (e.g., research → analyze → summarize).

#### Structured Outputs

Force Claude's response to match a specific schema:
1. **JSON mode:** Request JSON output format
2. **Strict tool use:** Add `strict: true` to tool definitions

#### Long-Context Prompting

Best practices for large documents:
- Place documents **before** your query
- Use XML tags to delineate document boundaries
- Give each document an index: `<document index="1">`

### Prompt Engineering Checklist

- [ ] Is the instruction clear and specific?
- [ ] Did I provide context/motivation for constraints?
- [ ] Are there 3-5 relevant, diverse examples?
- [ ] Are complex prompts structured with XML tags?
- [ ] Is there a role in the system prompt?
- [ ] Is the desired output format specified?
- [ ] Have I tested with edge cases?

---

## Part 9: Building with Claude — Three Paths

### Path 1: Messages API (Most Control)

- You write the agent loop
- You run your own tools and infrastructure
- Full control over every aspect
- Best for: Custom implementations, complex architectures

### Path 2: Claude Agent SDK (Middle Ground)

- SDK provides the agent loop and tool execution
- You operate the process
- Less boilerplate than raw API
- Best for: Standard agentic workflows, rapid prototyping

### Path 3: Claude Managed Agents (Most Offloaded)

- Anthropic hosts the agent loop, tool execution, and runtime
- Pre-built, configurable agent harness
- Session-based pricing (tokens + $0.08/session-hour)
- Best for: Long-running tasks, asynchronous work

### Use Case Guides (Production Patterns)

| Use Case | Description |
|----------|-------------|
| **Ticket Routing** | Classify and route support tickets at scale |
| **Customer Support Agent** | Context-aware chatbots for customer interactions |
| **Content Moderation** | Content filtering and moderation |
| **Legal Summarization** | Extract key information from legal documents |
| **Commerce Agent** | Shopping and merchant agents |

---

## Part 10: Safety, Ethics & Data Policies

### Anthropic's Core Views on AI Safety

**Three Key Positions:**

1. **Rapid progress is likely** — Scaling laws show predictable capability improvement. Training costs remain far below major scientific projects, with room for growth.

2. **Alignment is unsolved** — "No one knows how to train very powerful AI systems to be robustly helpful, honest, and harmless." Competitive pressures could accelerate deployment before safety is achieved.

3. **Empirical research is essential** — Safety work must engage with frontier models. Large models behave qualitatively differently from smaller ones — problems may only emerge at scale.

### Safety Research Portfolio

| Area | Description |
|------|-------------|
| **Mechanistic Interpretability** | Reverse-engineering neural networks to audit for unsafe behaviors |
| **Scalable Oversight** | Using AI to help supervise AI training (Constitutional AI) |
| **Process-Oriented Learning** | Rewarding safe reasoning steps, not just outcomes |
| **Understanding Generalization** | Tracing how training data shapes emergent behaviors |

### Three Scenarios Anthropic Prepares For

| Scenario | Description | Strategy |
|----------|-------------|----------|
| **Optimistic** | Current safety techniques are largely sufficient | Continue current approaches |
| **Intermediate** | Significant effort can prevent catastrophe | Invest heavily in safety research |
| **Pessimistic** | Safety may be unsolvable | Evidence-gathering becomes critical |

Anthropic's strategy **hedges across all three** rather than betting on one.

### Data Retention Policies

**Key principles:**
- Retained data is **never used for model training** without express permission
- Anthropic designs for the **smallest possible retention footprint**
- Different APIs and features have different storage needs

### Zero Data Retention (ZDR)

Where a feature doesn't require storage of customer prompts or responses, it may be ZDR-eligible.

**ZDR-Eligible Features:**
- Context windows, thinking, vision, PDF support
- Citations, search results, structured outputs
- Tool use (client-side), batch tool definitions
- Prompt caching, token counting
- Web search and web fetch (except with dynamic filtering)
- Computer use, browser use, bash, text editor, memory tools

**NOT ZDR-Eligible:**
- Batch processing
- Code execution
- Files API
- Agent Skills
- MCP connector

### HIPAA Readiness

Anthropic offers HIPAA-ready API access with the same feature eligibility structure as ZDR.

### Data Processor Roles

| Platform | Data Processor |
|----------|---------------|
| Claude API, Claude Platform on AWS, Microsoft Foundry | **Anthropic** |
| Amazon Bedrock | **AWS** |
| Google Cloud / Vertex AI | **Google** |

### Responsible Use

- Claude cannot be used to name people in images
- Claude's outputs should not substitute for professional medical diagnosis
- Always verify Claude's interpretations for high-stakes use cases
- Claude cannot determine if an image is AI-generated

---

## Part 11: Practice Test — 50 MCQs with Answers

### Section A: Models & Pricing (Questions 1-10)

**Q1.** Which Claude model is recommended as the default starting point for most workloads?

A) Claude Fable 5.1
B) Claude Opus 5.5
C) Claude Sonnet 5
D) Claude Haiku 4.5

**Answer: B**
The docs state: "If you're unsure which model to use, start with Claude Opus 5.5 for most workloads."

---

**Q2.** What is the context window size for Claude Haiku 4.5?

A) 1M tokens
B) 500K tokens
C) 200K tokens
D) 100K tokens

**Answer: C**
Haiku 4.5 has a 200K token context window. All other current models (Fable, Opus, Sonnet) have 1M token windows.

---

**Q3.** What discount does the Batch API provide on standard token costs?

A) 25%
B) 50%
C) 75%
D) 30%

**Answer: B**
The Batch API allows asynchronous processing with a 50% discount on both input and output tokens.

---

**Q4.** What is the base output token price for Claude Sonnet 5?

A) $5 / MTok
B) $10 / MTok
C) $15 / MTok
D) $20 / MTok

**Answer: B**
Sonnet 5 is priced at $2/MTok input and $10/MTok output.

---

**Q5.** What multiplier does the 5-minute prompt cache write incur over base input price?

A) 1.1x
B) 1.25x
C) 1.5x
D) 2x

**Answer: B**
5-minute cache write = 1.25x base input price. 1-hour cache write = 2x base input price.

---

**Q6.** What is the maximum output token limit for Claude Fable 5.1?

A) 64K tokens
B) 128K tokens
C) 256K tokens
D) 1M tokens

**Answer: B**
Fable 5.1, Opus 5.5, and Sonnet 5 all support up to 128K output tokens. Haiku 4.5 supports 64K.

---

**Q7.** What is the cache hit cost for Claude Fable 5.1 relative to base input price?

A) 10% (0.1x)
B) 5% (0.05x)
C) 2.5% (0.025x)
D) 1% (0.01x)

**Answer: C**
Fable 5.1 cache hits cost 2.5% of base input ($0.25/MTok). Standard models use 10%. Opus 5.5 uses 5%.

---

**Q8.** Which pricing multiplier applies when using US-only data residency?

A) 1.0x (no change)
B) 1.05x
C) 1.1x
D) 1.25x

**Answer: C**
Specifying `inference_geo: "us"` incurs a 1.1x multiplier on all token pricing categories.

---

**Q9.** How many platforms currently offer Claude models?

A) Three
B) Four
C) Five
D) Six

**Answer: C**
Five: Claude API, Amazon Bedrock, Claude Platform on AWS, Google Cloud/Vertex AI, Microsoft Foundry.

---

**Q10.** What is the default effort level for Claude Opus 5.5?

A) low
B) medium
C) high
D) Not supported

**Answer: B**
Opus 5.5 defaults to "medium." Fable 5.1 and Sonnet 5 default to "high." Haiku 4.5 doesn't support effort.

---

### Section B: API & Tools (Questions 11-20)

**Q11.** What `stop_reason` indicates that Claude wants to call a tool?

A) end_turn
B) tool_use
C) stop_sequence
D) max_tokens

**Answer: B**
When Claude wants to call tools, the response has `stop_reason: "tool_use"` and contains `tool_use` content blocks.

---

**Q12.** Which `tool_choice` type forces Claude to call a specific named tool?

A) auto
B) any
C) tool
D) required

**Answer: C**
`tool` forces a specific tool. `any` forces some tool. `auto` lets Claude decide. `none` prevents calls.

---

**Q13.** Server-side tools run on which infrastructure?

A) The user's application server
B) Anthropic's infrastructure
C) AWS Lambda
D) The client browser

**Answer: B**
Server tools (web search, web fetch, code execution) run on Anthropic's infrastructure with no handler code needed.

---

**Q14.** What is the per-search cost for the Web Search tool?

A) $1 per 1,000 searches
B) $5 per 1,000 searches
C) $10 per 1,000 searches
D) Free

**Answer: C**
Web search costs $10 per 1,000 searches, plus standard token costs for search-generated content.

---

**Q15.** What additional cost does the Web Fetch tool incur beyond tokens?

A) $5 per 1,000 fetches
B) $0.01 per fetch
C) No additional cost
D) $10 per 1,000 fetches

**Answer: C**
Web fetch has no additional charges beyond standard token costs for the fetched content.

---

**Q16.** In the Messages API, which parameter sets the system prompt?

A) context
B) instructions
C) system
D) prompt

**Answer: C**
The `system` parameter sets the system prompt for context and behavior instructions.

---

**Q17.** What is the session runtime charge for Claude Managed Agents?

A) $0.04 per session-hour
B) $0.08 per session-hour
C) $0.15 per session-hour
D) Free

**Answer: B**
Session runtime is $0.08/session-hour, metered to the millisecond only while status is `running`.

---

**Q18.** Which content block type must you send back after executing a tool?

A) tool_output
B) tool_response
C) tool_result
D) function_result

**Answer: C**
After executing a tool, send a `tool_result` content block with the `tool_use_id` and the result content.

---

**Q19.** What is the MCP connector used for?

A) Managing Claude's memory
B) Connecting to remote MCP servers from the Messages API
C) Monitoring Claude's performance
D) Compressing prompts

**Answer: B**
The MCP connector connects to remote Model Context Protocol servers directly from the Messages API.

---

**Q20.** Which of the following is NOT an Anthropic-schema client tool?

A) Bash
B) Text Editor
C) Web Search
D) Computer Use

**Answer: C**
Web Search is a server-side tool. Bash, Text Editor, Computer Use, Browser Use, and Memory are client tools.

---

### Section C: Prompt Engineering (Questions 21-30)

**Q21.** According to Anthropic's "golden rule" for prompt clarity, what should you do?

A) Use the longest prompt possible
B) Show the prompt to a colleague with minimal context
C) Always use XML tags
D) Include at least 10 examples

**Answer: B**
Golden rule: Show your prompt to a colleague with minimal context. If they'd be confused, Claude will be too.

---

**Q22.** How many examples does Anthropic recommend for few-shot prompting?

A) 1-2
B) 3-5
C) 7-10
D) 15+

**Answer: B**
Include 3-5 examples for best results, making them relevant, diverse, and wrapped in `<example>` tags.

---

**Q23.** What markup format does Anthropic recommend for structuring complex prompts?

A) JSON
B) YAML
C) XML tags
D) Markdown headers

**Answer: C**
XML tags help Claude parse complex prompts unambiguously. Use `<instructions>`, `<context>`, `<input>`.

---

**Q24.** What is "adaptive thinking" in Claude models after 4.7?

A) Claude follows a fixed thinking template
B) Claude dynamically decides when and how much to think
C) User manually sets thinking budget tokens
D) Thinking is always disabled

**Answer: B**
Adaptive thinking lets Claude dynamically decide how much to think. It's the only mode on Claude 4.7+.

---

**Q25.** Where should role information be placed in a Claude API request?

A) In the first user message
B) In the system prompt
C) As a tool definition
D) In the stop_sequences

**Answer: B**
Set a role in the system prompt to focus Claude's behavior and tone.

---

**Q26.** What is prompt chaining?

A) Sending multiple API calls simultaneously
B) Breaking complex tasks into sequential sub-tasks
C) Linking multiple models together
D) Caching prompts across sessions

**Answer: B**
Prompt chaining breaks complex tasks into sequential sub-tasks, each focusing on one aspect.

---

**Q27.** Why does adding context/motivation improve prompts?

A) It makes prompts longer
B) Claude can generalize from the explanation
C) It forces XML tag usage
D) It reduces token count

**Answer: B**
Explaining WHY helps Claude better understand goals. Claude generalizes from the explanation.

---

**Q28.** What does the `effort` parameter control?

A) Output format
B) How many tokens Claude uses when responding
C) The model's temperature
D) API rate limits

**Answer: B**
Effort controls how many tokens Claude uses, trading off thoroughness and token efficiency.

---

**Q29.** When using examples in prompts, what tags should wrap them?

A) `<sample>` tags
B) `<demo>` tags
C) `<example>` tags
D) `<test>` tags

**Answer: C**
Wrap examples in `<example>` tags (multiple in `<examples>`) so Claude distinguishes them from instructions.

---

**Q30.** What does `strict: true` in a tool definition guarantee?

A) The tool always executes
B) Claude's tool calls match your schema exactly
C) The tool runs server-side
D) Error messages are suppressed

**Answer: B**
`strict: true` ensures Claude's tool call arguments always conform to your JSON schema exactly.

---

### Section D: Features & Capabilities (Questions 31-40)

**Q31.** Which feature grounds responses in source documents with exact references?

A) Structured outputs
B) Citations
C) Search results
D) Prompt caching

**Answer: B**
Citations provide detailed references to exact sentences and passages for verifiable, trustworthy outputs.

---

**Q32.** What image formats does Claude's vision support?

A) JPEG and PNG only
B) JPEG, PNG, GIF, and WebP
C) All image formats
D) JPEG, PNG, and SVG

**Answer: B**
Claude supports JPEG, PNG, GIF, and WebP. Animations are unsupported (only first frame used).

---

**Q33.** What is the maximum image dimension Claude accepts?

A) 4096 x 4096 px
B) 8000 x 8000 px
C) 2576 x 2576 px
D) 1920 x 1080 px

**Answer: B**
The maximum dimensions per image are 8000 x 8000 px.

---

**Q34.** How are visual tokens calculated for images?

A) Total pixels / 1000
B) ⌈width/28⌉ × ⌈height/28⌉
C) Width × Height
D) File size in KB

**Answer: B**
Each 28×28 pixel patch = 1 visual token. Cost = ⌈width/28⌉ × ⌈height/28⌉.

---

**Q35.** What does the "compaction at a token threshold" feature do?

A) Compresses images
B) Summarizes earlier conversation parts when tokens reach a trigger
C) Reduces prompt cache size
D) Removes stop sequences

**Answer: B**
Server-side context summarization that automatically summarizes earlier conversation parts.

---

**Q36.** Which feature enables processing large request volumes asynchronously?

A) Streaming
B) Prompt caching
C) Batch processing
D) Fast mode

**Answer: C**
Batch processing allows asynchronous processing at 50% token discount.

---

**Q37.** How many approaches does structured outputs offer for schema conformance?

A) One
B) Two
C) Three
D) Four

**Answer: B**
Two: JSON outputs for structured data, and strict tool use for validated tool inputs.

---

**Q38.** Which source types can be used to provide images in the API?

A) Base64 only
B) URL only
C) Base64, URL, and file_id
D) Base64 and URL only

**Answer: C**
Three source types: base64-encoded, URL reference, and file_id from the Files API.

---

**Q39.** What does the Token Counting API help you do?

A) Count words in responses
B) Determine token count before sending to Claude
C) Track billing automatically
D) Measure response latency

**Answer: B**
Token counting lets you determine token count before sending, helping with prompt optimization.

---

**Q40.** Can Claude generate, edit, or create images?

A) Yes, all models can generate images
B) Only Fable 5.1 can generate images
C) No, Claude is image understanding only
D) Yes, with the Vision tool

**Answer: C**
Claude is an image understanding model only. It can interpret and analyze images but cannot generate, produce, edit, or create them.

---

### Section E: Safety & Data Policies (Questions 41-50)

**Q41.** What is Anthropic's position on using customer data for model training?

A) All data is used for training
B) Data is used unless opted out
C) Data is never used without express permission
D) Only anonymized data is used

**Answer: C**
Retained data is never used for model training without express permission.

---

**Q42.** What does ZDR stand for?

A) Zero Downtime Recovery
B) Zero Data Retention
C) Zero Delay Response
D) Zone Data Replication

**Answer: B**
Zero Data Retention — Anthropic does not store customer prompts or responses for eligible features.

---

**Q43.** Which of these features is NOT ZDR-eligible?

A) Context windows
B) Vision
C) Batch processing
D) PDF support

**Answer: C**
Batch processing is not ZDR-eligible. Context windows, vision, PDF support, and thinking are ZDR-eligible.

---

**Q44.** What are Anthropic's three preparedness scenarios?

A) Small, medium, large
B) Optimistic, intermediate, pessimistic
C) Internal, external, hybrid
D) Current, near-term, long-term

**Answer: B**
Optimistic (current techniques suffice), intermediate (effort prevents catastrophe), pessimistic (safety may be unsolvable).

---

**Q45.** Which research area involves reverse-engineering neural networks?

A) Scalable oversight
B) Process-oriented learning
C) Mechanistic interpretability
D) Understanding generalization

**Answer: C**
Mechanistic interpretability: reverse-engineering neural networks to audit for unsafe behaviors.

---

**Q46.** On Amazon Bedrock, who is the data processor?

A) Anthropic
B) The cloud provider (AWS)
C) The customer
D) A third-party auditor

**Answer: B**
On Bedrock and Google Cloud, the cloud provider is the data processor. On Claude API, Anthropic is.

---

**Q47.** What is Constitutional AI?

A) A legal compliance framework
B) A technique using AI to help supervise AI training
C) A government regulation
D) A model architecture

**Answer: B**
Constitutional AI is part of scalable oversight, using AI principles to help supervise AI training.

---

**Q48.** What does Anthropic say about the current state of AI alignment?

A) It's fully solved
B) It's mostly solved with minor gaps
C) No one knows how to achieve robust alignment
D) It's impossible to solve

**Answer: C**
"No one knows how to train very powerful AI systems to be robustly helpful, honest, and harmless."

---

**Q49.** Does Anthropic offer HIPAA-ready API access?

A) No
B) Yes, with the same feature eligibility as ZDR
C) Yes, for all features
D) Only on Bedrock

**Answer: B**
Yes, with the same feature eligibility structure as ZDR.

---

**Q50.** Why does Anthropic believe empirical safety research requires frontier models?

A) Smaller models are too expensive
B) Problems may only emerge at scale
C) Legal requirements mandate it
D) Smaller models don't support safety features

**Answer: B**
Large models behave qualitatively differently from smaller ones. Safety problems may only emerge at scale.

---

## Quick Reference Card

### Token Approximations
- 1 token ≈ 4 characters ≈ 0.75 words (English)
- 1M tokens ≈ 555K words (new tokenizer, Claude 4.7+)
- 200K tokens ≈ 150K words

### Essential API Headers
```
x-api-key: $ANTHROPIC_API_KEY
anthropic-version: 2023-06-01
content-type: application/json
```

### Model ID Quick Reference
```
claude-fable-5-1       → Most capable reasoning
claude-opus-5-5        → Agentic coding (recommended default)
claude-sonnet-5        → Speed + intelligence balance
claude-haiku-4-5-20251001  → Fastest, lowest cost
```

### Prompt Engineering Mnemononic: CERRS
- **C**lear and direct instructions
- **E**xamples (3-5, diverse, in XML tags)
- **R**ole in system prompt
- **R**easoning context (explain WHY)
- **S**tructure with XML tags

### Study Resources
1. **Official Docs:** platform.claude.com/docs
2. **Claude Academy:** academy.claude.com
3. **Prompt Tutorial:** github.com/anthropics/prompt-eng-interactive-tutorial
4. **Cookbook:** platform.claude.com/cookbook
5. **Safety Research:** anthropic.com/research
6. **Transparency Hub:** anthropic.com/transparency
