# UC1 AI News Alarm — Automated Sector Intelligence Briefing

> **Status**: ✅ Deployed & Active | **Daily Time Saved**: ~0.5 hrs/person | **Coverage**: EU Region, Multiple Sectors

## Overview

AI News Alarm is an automated intelligence briefing workflow that delivers daily structured sector news digests via email. It uses a **dual-LLM agentic architecture**: one model acts as a research agent (tool-calling), the other as a formatter (structured output). The system is deployed.

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
         │   Claude Sonnet 4.6   │  ← Tool-calling + Web search (Tavily, raw text off)
         │   max_tokens: 5,000   │
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
| Max Tokens | 5,000 (optimised) | 2,048 (default) / 5,000 (tested) |
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
// ✅ Claude Sonnet 4.6 — max_tokens: 5,000 + Tavily raw text off
"text": "\nHere is your comprehensive EU region sector news briefing:\n\n
---\n\n## 🌱 1. [SECTOR NAME]\n\nTitle: ...\nDate: ...\nSource: ..."
```

**Root cause**: FM capacity — both in context window size and the model's ability to separate reasoning from output — directly determines the quality of agentic workflows.

---

## Post-Deployment Optimisations

Following extended testing with the internal AI team, two configuration changes improved output quality:

| Parameter | Before | After | Effect |
|---|---|---|---|
| Claude max_tokens | 128,000 | 5,000 | Tighter output scope, reduced verbosity |
| Tavily raw text | On | Off | Cleaner search results passed to research agent |

Both changes were identified through empirical testing rather than design assumption — a reminder that optimal LLM configuration often requires real-world validation.

---



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

---

## Future Roadmap

The information-gathering layer of this workflow could feed into a future Early Warning Indicator (EWI) engine:

- **EWI Engine**: Extend sector monitoring to detect market anomalies, regulatory shifts, and supply chain signals
- **Multi-region support**: Expand geographic coverage beyond current scope
- **Formatter stability**: Replace Qwen with a more constrained structured-output approach (JSON schema enforcement)
- **Personalization**: Role-based briefing profiles (sector-specific digests per team)

---

## Lessons Learnt

1. **FM model capacity matters end-to-end** — context window size and reasoning quality both affect final output structure
2. **Dual-LLM separation of concerns works** — research agent + formatter is a sound architectural pattern
3. **Prompt engineering for formatters is critical** — the last-mile LLM determines perceived output quality
4. **Agentic workflows need determinism guardrails** — non-deterministic outputs require schema enforcement at the formatter layer

---

*Part of the UC (Use Case) AI deployment series.*
