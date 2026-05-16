# UC1 AI News Alarm — Automated Sector Intelligence Briefing

> **Status**: ✅ Deployed & Active | **Daily Time Saved**: ~0.5 hrs/person | **Coverage**: EU Region, Multiple Sectors

## Overview

AI News Alarm is an automated intelligence briefing workflow that delivers daily structured sector news digests via email. It uses a **dual-LLM agentic architecture**: one model acts as a research agent (tool-calling), the other as a formatter (structured output). The system is deployed in production.

**Target**: Multiple industry sectors within the EU region.

---

## Architecture

```
┌─────────────────────────────────────────────────┐
│              Orchestration Layer                 │
│         (Scheduling / Trigger / Config)          │
└────────────────────┬────────────────────────────┘
                     │
         ┌───────────▼───────────┐
         │   Research Agent      │
         │   Claude Sonnet 4.6   │  ← Tool-calling + Web search (Tavily)
         │   max_tokens: 128,000 │
         └───────────┬───────────┘
                     │  Raw structured news data
         ┌───────────▼───────────┐
         │   Formatter LLM       │
         │   Qwen                │  ← Email body generation
         └───────────┬───────────┘
                     │  HTML/Markdown email body
         ┌───────────▼───────────┐
         │   Email Delivery      │  → Team Inbox
         └───────────────────────┘
```

---

## Model Selection: Key Findings

A central design decision was choosing the right Foundation Model (FM) for the research agent role. During testing, two models were evaluated:

| Parameter | Claude Sonnet 4.6 | Amazon Nova Pro V1 |
|---|---|---|
| Max Tokens | 128,000 | 2,048 (default) / 5,000 (tested) |
| Reasoning Style | Internal (clean output) | Exposed `<thinking>` tags |
| Output Quality | Structured, logical | Token leakage of reasoning steps |
| Tool-Calling | Reliable | Functional but verbose |

### The Token Leakage Problem

Amazon Nova Pro V1 at default settings (2,048 tokens) exposes its chain-of-thought reasoning directly in the final output:

```json
// ❌ Nova Pro V1 — max_tokens: 2048
"text": "<thinking> To provide comprehensive and detailed summaries of the 
latest events in the specified sectors within the EU region, I will use the 
`tavily_search` tool... </thinking>\n\nHere are the top stories..."
```

Increasing to 5,000 tokens improved output quality but did not fully resolve the leakage issue. Claude Sonnet 4.6, by contrast, produces clean, structured output by keeping its reasoning internal:

```json
// ✅ Claude Sonnet 4.6 — max_tokens: 128,000
"text": "\nHere is your comprehensive EU region sector news briefing:\n\n
---\n\n## 🌱 1. [SECTOR NAME]\n\nTitle: ...\nDate: ...\nSource: ..."
```

**Root cause**: FM capacity — both in context window size and the model's ability to separate reasoning from output — directly determines the quality of agentic workflows.

---

## Known Issues & Improvement Areas

Based on 6 days of production testing:

| Issue | Frequency | Root Cause | Proposed Fix |
|---|---|---|---|
| Inconsistent email formatting | ~50% of emails | Non-deterministic LLM output (Qwen) | Stricter prompt constraints + output schema |
| Missing hyperlinks in output | Observed instance | Formatter prompt ambiguity | Explicit link inclusion instruction in prompt |
| Format variation across emails | Observed instances | Temperature / sampling variance | Lower temperature + few-shot examples in prompt |

**Key insight**: LLM output is inherently non-deterministic. For production pipelines requiring consistent formatting, the formatter prompt should include:
- Explicit output schema definition
- Negative examples (what NOT to do)
- Required fields checklist

---

## Quantified Impact

| Metric | Value |
|---|---|
| Daily time saved per user | **~0.5 hours** |
| Geographic focus | EU Region |
| Delivery frequency | Daily |
| Testing period | 6+ days (active) |

## Lessons Learnt

1. **FM model capacity matters end-to-end** — context window size and reasoning quality both affect final output structure
2. **Dual-LLM separation of concerns works** — research agent + formatter is a sound architectural pattern
3. **Prompt engineering for formatters is critical** — the last-mile LLM determines perceived output quality
4. **Agentic workflows need determinism guardrails** — non-deterministic outputs require schema enforcement at the formatter layer

---

*Part of the UC (Use Case) AI deployment series.*
