# PLAN: Claude Sonnet 4.6 Workshop Notebooks — Build Guide

> **Purpose**: This document is a complete, self-contained plan for building a set of Jupyter notebooks that showcase Claude Sonnet 4.6 capabilities on Microsoft Foundry. Hand this to a coding agent or developer and they can reproduce the full set from scratch.

---

## Overview

Create **5 Python notebooks** under `sandbox/` that teach Claude Sonnet 4.6 capabilities through Microsoft Foundry. The audience is **AI-familiar but new to Foundry and Claude models**.

### File Structure

```
sandbox/
├── requirements.txt                # Shared dependencies
├── 00-foundry-e2e.ipynb            # End-to-end Foundry workflow
├── 01-multi-turn-chat.ipynb        # Multi-turn conversations
├── 02-reasoning.ipynb              # Extended thinking & reasoning
├── 03-multimodal.ipynb             # Vision & multi-modal input
└── 04-code-generation.ipynb        # Code generation & analysis
```

### Dependencies (`requirements.txt`)

```
agent-framework
agent-framework-foundry
python-dotenv
jupyter
ipykernel
Pillow
requests
```

---

## Design Principles

1. **Use `agent-framework` + `agent-framework-foundry`** — same libraries as the workshop. NOT the raw Anthropic SDK.
2. **Alternating markdown/code cells** — every code cell MUST be preceded by a markdown cell explaining what comes next.
3. **Title cell describes a use case** — each notebook opens with a real enterprise scenario, then walks through how the model enables the solution.
4. **All code cells use top-level `await`** — ipykernel supports this natively. Do NOT use `asyncio.run()` or `async def main()`.
5. **Self-contained** — each notebook loads credentials from `../.env` and can run independently.
6. **Analogies and metaphors throughout** — make complex concepts accessible to newcomers.
7. **Enterprise use cases** — realistic scenarios with clear business value.
8. **End with exercises** — each notebook has a "Try It Yourself" section.

---

## Environment Setup

The repo root `.env` file contains:

```env
FOUNDRY_ENDPOINT="https://<resource>.services.ai.azure.com/anthropic/"
FOUNDRY_API_KEY="<your-api-key>"
FOUNDRY_MODEL_DEPLOYMENT="claude-sonnet-4-6"
```

Every notebook starts with this common setup pattern:

```python
import os
from dotenv import load_dotenv
from agent_framework import Agent
from agent_framework.foundry import AnthropicFoundryClient

load_dotenv(dotenv_path="../.env")

chat_client = AnthropicFoundryClient(
    model=os.environ["FOUNDRY_MODEL_DEPLOYMENT"],
    api_key=os.environ["FOUNDRY_API_KEY"],
    base_url=os.environ["FOUNDRY_ENDPOINT"],
)
```

---

## API Reference (agent-framework)

### Core

```python
# Chat client — connection to the model
chat_client = AnthropicFoundryClient(model=, api_key=, base_url=)

# Agent — wraps client with persona, memory, tools
agent = Agent(client=chat_client, name=, instructions=, tools=, default_options=)

# Sessions — multi-turn conversation memory
session = agent.create_session()
response = await agent.run("message", session=session)
print(response.text)
```

### Messages & Content (for multi-modal)

```python
from agent_framework import Message, Content

# Text
Content.from_text("What do you see?")

# Image from bytes
Content.from_data(data=image_bytes, media_type="image/png")

# Image from URL
Content.from_uri("https://example.com/image.jpg", media_type="image/jpeg")

# Compose a multi-content message
message = Message(role="user", contents=[image_content, text_content])
response = await agent.run(message, session=session)
```

### Options (temperature, extended thinking)

```python
from agent_framework.anthropic import AnthropicChatOptions

# Temperature control
options = AnthropicChatOptions(temperature=0.0, max_tokens=2048)

# Extended thinking (NOTE: temperature MUST NOT be set when thinking is enabled)
options = AnthropicChatOptions(
    thinking={"type": "enabled", "budget_tokens": 5000},
    max_tokens=16000,  # must be > budget_tokens
)

agent = Agent(client=chat_client, name="x", default_options=options)
```

### Tracing / Observability

```python
from agent_framework.observability import configure_otel_providers

# Console tracing — prints trace spans to stdout (great for notebooks)
configure_otel_providers(enable_console_exporters=True, enable_sensitive_data=True)

# After enabling, all agent.run() calls automatically emit OpenTelemetry traces
```

### Evaluations

```python
from agent_framework import evaluate_agent, evaluator, keyword_check, LocalEvaluator, CheckResult

# Built-in keyword check
kw = keyword_check("policy", "remote work")

# Custom evaluator
@evaluator(name="quality_check")
def check_quality(query: str, response: str) -> bool:
    return len(response) > 50

@evaluator(name="tone_check")
def check_tone(response: str) -> CheckResult:
    informal = ["hey", "gonna", "wanna"]
    found = [w for w in informal if w.lower() in response.lower()]
    return CheckResult(passed=len(found) == 0, score=1.0 - len(found) * 0.2,
                       reason=f"Informal: {found}" if found else "OK")

# Combine and run
local_eval = LocalEvaluator(kw, check_quality, check_tone)
results = await evaluate_agent(agent=agent, queries=["..."], evaluators=local_eval)

# Results
for r in results:
    print(f"{r.provider}: {r.passed}/{r.total}")
    for item in r.items:
        for score in item.scores:
            print(f"  {score.name}: {'PASS' if score.passed else 'FAIL'} ({score.score})")
```

### Foundry Cloud Evaluators (requires OpenAI deployment)

```python
from agent_framework.foundry import FoundryEvals

# Built-in evaluator constants:
# Quality: COHERENCE, FLUENCY, RELEVANCE, GROUNDEDNESS, SIMILARITY
# Task: TASK_ADHERENCE, RESPONSE_COMPLETENESS, INTENT_RESOLUTION
# Safety: HATE_UNFAIRNESS, SELF_HARM, SEXUAL, VIOLENCE
# Tool: TOOL_CALL_ACCURACY, TOOL_SELECTION, TOOL_INPUT_ACCURACY
```

---

## Notebook 0: `00-foundry-e2e.ipynb`

### 🚀 End-to-End AI Agent Development with Microsoft Foundry

**Use Case**: NovaCorp, a B2B SaaS company, needs an internal AI policy assistant for HR policies, benefits, and compliance questions. This notebook walks through the ENTIRE Foundry lifecycle.

**Goal**: Show learners the complete pipeline — model selection → agent creation → tracing → evaluation → red teaming → deployment. Each section explains WHY that step matters, not just HOW.

| # | Type | Content |
|---|------|---------|
| 1 | MD | **Title & Use Case**. NovaCorp Policy Assistant scenario. Pipeline overview diagram. Analogy: building a car — design, test each component, crash-test, then ship. Learning objectives. |
| 2 | MD | **🏗️ Step 1: Model Selection — Choosing the Right Brain**. Analogy: hiring — match skill level to the job. Explain model catalog: Claude Sonnet 4.6 ("senior engineer"), Haiku ("efficient assistant"), GPT-4o (alternative). Concepts: model vs deployment, endpoint, API key. |
| 3 | Code | Setup: load .env, create client, create NovaCorp Policy Assistant agent with rich system prompt (identity, tone, knowledge scope, uncertainty handling). Test with a simple query. |
| 4 | MD | **🤖 Step 2: Agent Creation — Building Your AI Employee**. Analogy: model is the brain, Agent is the whole person — brain + job description + desk + filing cabinet. Three pillars: client, instructions, session. |
| 5 | Code | Multi-turn policy conversation: "What is the remote work policy?" → "How many PTO days?" → "Can I roll over unused PTO?" Show session maintaining context. |
| 6 | MD | **🔍 Step 3: Tracing — X-Ray Vision for Your Agent**. Analogy: restaurant kitchen camera — see what happened when a customer complains. Explain OpenTelemetry, key metrics (latency, tokens, errors). |
| 7 | Code | Enable console tracing with `configure_otel_providers`. Run a query. Traces print to stdout. |
| 8 | MD | **📊 Reading Trace Output**. Explain span structure: agent run span, chat client span, token counts. In production → Azure Monitor. |
| 9 | MD | **✅ Step 4: Evaluation — Grading Your Agent**. Analogy: pilot flight simulator — test every scenario before passengers board. Two types: Local evaluators (unit tests, fast/free/deterministic) vs Foundry evaluators (AI judge code reviews). List all built-in Foundry evaluator categories. |
| 10 | Code | Create custom evaluators: keyword check for policy terms, response quality check, professional tone check, policy format check. Run `evaluate_agent()` with 3-4 queries. Display formatted results. |
| 11 | MD | **☁️ Foundry Evaluators — AI-Powered Quality Checks**. Explain FoundryEvals (uses judge model). Show code pattern in markdown (not executable — requires GPT-4o deployment). |
| 12 | MD | **🛡️ Step 5: Red Teaming — Stress-Testing Safety**. Analogy: banks hire people to try to break in. Categories: prompt injection, jailbreaking, information extraction, role confusion. Explain Azure AI Content Safety. |
| 13 | Code | Create adversarial test cases (3-4 attack types). Build `safety_check` evaluator. Run `evaluate_agent()` with adversarial queries. Display pass/fail results. |
| 14 | MD | **Interpreting Red Team Results**. What pass/fail means, how to strengthen system prompt, Azure AI Content Safety as production layer. |
| 15 | MD | **🚢 Step 6: Deployment — Going to Production**. Analogy: car is tested, time to hit the road. Options: Container Apps, App Service, Functions, Agent-as-API. Show FastAPI pattern in markdown code block. Production considerations: rate limiting, auth, monitoring, cost. |
| 16 | Code | Simulated deployment test: function that acts as API endpoint — takes message, runs through agent, returns structured response with metadata (text, latency, tokens). Run test queries, print table. |
| 17 | MD | **🗺️ The Complete Foundry Journey**. ASCII pipeline diagram. Key takeaways (Foundry is complete platform, every step has tooling, safety is baked in). Point to notebooks 01-04 for deep dives. |

---

## Notebook 1: `01-multi-turn-chat.ipynb`

### 🗣️ Multi-Turn Chat with Claude Sonnet 4.6

**Use Case**: TechNova Inc., a tech company, needs an AI IT help desk agent ("Nova") that can diagnose employee laptop/VPN/email issues through multi-turn conversations.

| # | Type | Content |
|---|------|---------|
| 1 | MD | **Title & Use Case**. TechNova scenario. Learning objectives: connect to Foundry, system prompts, sessions, context comparison, tone/temperature. |
| 2 | MD | **🔧 Setting Up Your Connection to Microsoft Foundry**. Analogy: Foundry is a restaurant kitchen — models are chefs, deployments are stations, API key is your reservation. Explain 3 env vars. |
| 3 | Code | Setup: imports, load_dotenv(`../.env`), create client, create basic agent, test with "Hello, are you there?" |
| 4 | MD | **📋 System Prompts — The Employee Handbook**. Analogy: employee handbook for a new hire — who they are, how to talk, what they can/can't do, when to escalate. |
| 5 | Code | Create Agent with IT help desk system prompt (name "Nova", professional/friendly tone, step-by-step troubleshooting, escalation rules). Send: "My laptop won't connect to the office Wi-Fi." |
| 6 | MD | **🔄 Multi-Turn Conversations — The Notebook That Follows You**. Analogy: doctor's medical record — without it, every visit starts from scratch. Explain `create_session()`. |
| 7 | Code | Create session. 3-turn conversation: (1) "VPN keeps disconnecting every 10 minutes" → (2) "Windows 11, GlobalProtect, started after last week's update" → (3) "Yes, tried restarting. Error says 'Gateway timeout'." Print each response. |
| 8 | MD | **🚫 What Happens Without Context?** Analogy: calling help desk and getting a different agent each time. |
| 9 | Code | Same Turn 2 and Turn 3 messages WITHOUT session. Show generic/confused responses. |
| 10 | MD | **🎭 Tone Control — Same Agent, Different Personality**. Analogy: casual Slack message vs formal email — same person, different voice. |
| 11 | Code | Two agents: formal enterprise vs casual/friendly system prompts. Same IT issue to both. Print comparison. |
| 12 | MD | **🌡️ Temperature — The Creativity Dial**. Analogy: "by the book" (0.0) vs "creative jazz" (1.0). Low for troubleshooting, higher for brainstorming. |
| 13 | Code | Same prompt at temperature=0.0 vs 1.0, run each twice. Show consistency vs variance. Use `AnthropicChatOptions`. |
| 14 | MD | **🎯 Try It Yourself** + Key Takeaways. Exercises: modify system prompt, extend to 5+ turns, try different temperatures. |

---

## Notebook 2: `02-reasoning.ipynb`

### 🧠 Extended Thinking & Reasoning with Claude Sonnet 4.6

**Use Case**: Meridian Capital, a mid-size investment firm. The CFO doesn't just want answers — she wants to see the reasoning behind complex investment decisions.

**CRITICAL**: When `thinking` is enabled, `temperature` MUST NOT be set. `max_tokens` must be greater than `budget_tokens`. Minimum budget is 1,024 tokens.

| # | Type | Content |
|---|------|---------|
| 1 | MD | **Title & Use Case**. Meridian Capital scenario — CFO needs to see reasoning. Learning objectives: understand extended thinking, enable it, analyze traces, tune budget, know when to use it. |
| 2 | Code | Setup: imports, load_dotenv, create client. Create basic agent (no thinking yet). Quick test. |
| 3 | MD | **💭 Standard Responses — The Black Box Problem**. Analogy: consultant's final report — clean but you can't see how they got there. For million-dollar decisions, not good enough. |
| 4 | Code | Ask WITHOUT extended thinking: "Should Meridian invest $5M in a Series B SaaS startup with $2M ARR, 40% YoY growth, but -$1.5M EBITDA?" |
| 5 | MD | **🔓 Extended Thinking — Showing the Work**. Analogy: consultant thinking out loud across the table — raw reasoning, trade-offs, doubts. Explain the `thinking` option. |
| 6 | Code | Same question WITH thinking enabled (`budget_tokens=5000`). Show thinking trace AND final answer separately. Iterate over `response.items` to find thinking vs text content blocks. |
| 7 | MD | **🔍 Analyzing the Thinking Trace**. What to look for: structure, factors, assumptions, uncertainty. "The thinking trace is your audit trail — in regulated industries, showing WHY is compliance." |
| 8 | Code | Helper function to extract and display thinking vs answer in formatted way. Re-run analysis. |
| 9 | MD | **⏱️ Budget Tokens — Allocating Thinking Time**. Analogy: consultant time allocation — 5 min quick take vs full afternoon deep dive. Min 1,024 tokens. |
| 10 | Code | Compare same complex question with small budget (1024) vs large (10000). Show thinking traces side by side — highlight depth difference. |
| 11 | MD | **📊 Complex Scenario — Full Risk Assessment**. Present multi-factor M&A scenario: revenue data, market position, regulatory risks, team, integration costs. |
| 12 | Code | Run complex acquisition analysis with generous thinking budget. Full thinking process + recommendation. |
| 13 | MD | **⚖️ When to Use Extended Thinking**. Decision guide: ✅ complex multi-step, high-stakes, need audit trail, ambiguous. ❌ simple lookups, basic text gen, cost-sensitive, latency-critical. Note: no temperature with thinking. |
| 14 | MD | **🎯 Try It Yourself** + Key Takeaways. Exercises: legal contract analysis, architecture review, vary budget tokens. |

---

## Notebook 3: `03-multimodal.ipynb`

### 👁️ Multi-Modal Vision with Claude Sonnet 4.6

**Use Case**: PrecisionTech, an electronic components manufacturer, needs AI-powered visual inspection for quality control + automated invoice/document processing.

**Images**: Do NOT store images in repo. Either:
- Programmatically generate images using Pillow (mock products, charts, invoices)
- Use public URLs (Wikipedia Commons, etc.)

Use `from IPython.display import display, Image as IPImage` to show images inline.

| # | Type | Content |
|---|------|---------|
| 1 | MD | **Title & Use Case**. PrecisionTech scenario — QA inspection + invoice processing. Learning objectives: send images, Content.from_data/from_uri, structured extraction, multi-image, prompt engineering. |
| 2 | Code | Setup: imports (os, dotenv, agent_framework, PIL, base64, io, requests, IPython.display), load_dotenv, create client and vision agent. |
| 3 | MD | **📸 How Claude "Sees" Images**. Analogy: describing a painting to a friend on the phone — Claude does the reverse, receives pixels and builds understanding. Can also read text, understand charts, spot defects. Two methods: `Content.from_data()` (raw bytes) and `Content.from_uri()` (URLs). |
| 4 | Code | Generate test image with Pillow (colorful geometric pattern with text). Send via `Content.from_data()`. Display image inline + print Claude's description. |
| 5 | MD | **🏭 Quality Inspection — Spotting Defects**. On the production line, human inspectors get tired. Claude doesn't. Explain inspection prompt crafting. |
| 6 | Code | Generate two Pillow images: "good" product (clean green rect with "PASS") and "defective" (red rect with scratch/mark). Send defective one with structured inspection prompt. Print findings. |
| 7 | MD | **🌐 Images from URLs**. Explain `Content.from_uri()` for web-hosted images. |
| 8 | Code | Use public URL image (Wikipedia Commons chart/diagram). Send via `Content.from_uri()`. Print analysis. |
| 9 | MD | **📄 Document Intelligence — Extracting Structured Data**. Analogy: giving Claude reading glasses and a spreadsheet template. Hundreds of invoices → JSON. |
| 10 | Code | Generate mock invoice image with Pillow (company name, date, line items, totals). Prompt: extract as JSON with vendor, date, items, subtotal, tax, grand_total. Display image + print extraction. |
| 11 | MD | **🔀 Multi-Image Comparison**. Multiple Content items in a single Message. |
| 12 | Code | Two variant product images (different colors, one with flaw). Send both in one message asking Claude to compare and identify differences. Show both inline. |
| 13 | MD | **🎯 Prompt Engineering for Vision — Getting Better Results**. Vague vs specific prompts. "What's in this?" vs "Identify all text, list anomalies, rate quality 1-10." |
| 14 | Code | Same image, two prompts (vague vs specific). Compare response quality. |
| 15 | MD | **🎯 Try It Yourself** + Key Takeaways. Exercises: real photo from phone, screenshot of spreadsheet, two product photos. |

---

## Notebook 4: `04-code-generation.ipynb`

### 💻 Code Generation & Analysis with Claude Sonnet 4.6

**Use Case**: LegacyBridge Financial, a financial services company modernizing their 20-year-old Java/COBOL trading platform to Python.

| # | Type | Content |
|---|------|---------|
| 1 | MD | **Title & Use Case**. LegacyBridge scenario — CTO greenlit modernization. Learning objectives: generate code, review/debug, translate languages, generate tests, iterative refinement, IaC. |
| 2 | Code | Setup: imports, load_dotenv, create client. Agent with code expert system prompt (Python/Java/COBOL/Terraform/Bicep expertise, docstrings, type hints, error handling, best practices). `temperature=0.0`. |
| 3 | MD | **🏗️ Code Generation — From Specification to Implementation**. Analogy: senior developer pair programming — needs context about codebase, constraints, goals. Specific specs → better code. |
| 4 | Code | Specification: `calculate_trade_settlement` function — trade dict input, T+1 stocks / T+2 bonds, commission fee, type hints, docstring, edge cases. Print generated code. |
| 5 | MD | **🔍 Code Review — Finding Bugs Before They Find You**. Analogy: tireless code reviewer who's read every security advisory. |
| 6 | Code | Give Claude a deliberately buggy `process_payment` function. Bugs: no error handling for unknown currency, discount on wrong amount, missing import, unclosed file handle, path injection risk, no input validation. Print Claude's review. |
| 7 | MD | **🔄 Language Translation — Modernizing Legacy Code**. Analogy: Rosetta Stone between programming languages. COBOL still processes 95% of ATM transactions. Preserve business logic. |
| 8 | Code | Java `TradeValidator` class → modern Python with dataclasses, type hints, Pythonic patterns. Print translation with explanations. |
| 9 | MD | **🧪 Test Generation — Building a Safety Net**. Analogy: measuring tape before cutting fabric — capture exact behavior before modernizing. |
| 10 | Code | Generate pytest tests for the Python trade validator: happy path, edge cases, boundary conditions, error cases. |
| 11 | MD | **🔁 Iterative Refinement — The Conversation Advantage**. Multi-turn meets code gen. Natural back-and-forth like real pair programming. |
| 12 | Code | Multi-turn session: (1) "Write an order book class" → (2) "Add validation and error handling" → (3) "Add bid-ask spread and market depth" → (4) "Add type hints and docstrings." Print each iteration. |
| 13 | MD | **☁️ Infrastructure as Code — DevOps Automation**. Everything as code — reproducible, version-controlled, reviewable. |
| 14 | Code | Generate Bicep/Terraform for: Container App API, PostgreSQL, Key Vault, Service Bus, managed identity, networking. Print IaC. |
| 15 | MD | **🎯 Try It Yourself** + Key Takeaways. Exercises: review a real function, translate between languages, generate tests for untested code. |

---

## Key Design Decisions (with rationale)

| Decision | Rationale |
|----------|-----------|
| **agent-framework** over raw Anthropic SDK | Consistent with workshop; teaches production framework |
| **Alternating markdown/code cells** | Every step explained before executed; good for learning |
| **Enterprise use cases** | IT help desk, financial analysis, quality inspection, code modernization — clear business value |
| **Analogies throughout** | Bridge for learners new to Foundry/Claude (restaurant kitchen, employee handbook, medical record, etc.) |
| **Self-contained notebooks** | Any notebook runs independently; no cross-dependencies |
| **Top-level await** | Clean notebook code; ipykernel handles async natively |
| **Pillow-generated images** | No stored assets; notebooks generate their own test images |
| **Local evaluators for demos** | FoundryEvals needs GPT-4o; local evaluators always work |
| **Red teaming with custom evaluators** | Demonstrates the concept without external dependencies |
| **Deployment as simulation** | Shows the pattern without requiring Azure resources beyond the model |

---

## Reproduction Steps

1. Ensure the repo has a `.env` file at the root with `FOUNDRY_ENDPOINT`, `FOUNDRY_API_KEY`, and `FOUNDRY_MODEL_DEPLOYMENT`
2. Create `sandbox/requirements.txt` with the dependencies listed above
3. Run `pip install -r sandbox/requirements.txt`
4. Create each notebook following the cell-by-cell plans above
5. Verify: each notebook should have alternating MD/code cells, use top-level `await`, load from `../.env`, and pass JSON validation
