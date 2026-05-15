# AI for Regulatory Compliance: Three Failures and a New Architecture

> Regulatory Gap Analysis — P&P vs EBA Guidelines

> **Status**: 🔧 Architecture Rebuild | **Platform**: Dify + Claude Sonnet | **Previous Approach**: RAG (abandoned) | **Current Approach**: Two-LLM Pipeline + Iterator

-----

## Overview

This project aims to automate the comparison of internal Policies & Procedures (P&P) against EBA regulatory guidelines — identifying gaps clause by clause, with traceable citations and structured output suitable for audit review.

The first architectural approach (RAG with two knowledge bases) was abandoned after hitting three fundamental problems. This README documents what went wrong, the new architecture, and current status.

-----

## What Went Wrong: Three Problems with the RAG Approach

### Problem 1: Chunking Granularity

Dify’s document chunking splits by token count. EBA guidelines have a natural structure where each paragraph corresponds to one discrete regulatory requirement. Token-based splitting breaks this structure — requirements get cut mid-sentence, or two separate requirements get merged into one chunk.

Fixing this properly requires custom Python-based chunking, which is currently a skill gap being addressed.

### Problem 2: RAG Is Not Designed for Cross-Document Comparison

The original design used two knowledge bases — one for EBA guidelines, one for P&P. In practice, the LLM retrieves fragments from both sides and attempts to hold them simultaneously while making a judgment. Results were inconsistent.

**Root cause**: RAG is optimised for retrieval-and-answer tasks. Structured comparison between two document corpora is a different task pattern that RAG handles poorly.

### Problem 3: No Applicability Context

EBA guidelines cover a wide scope of institution types and business activities. Our bank does not operate in retail banking or securities underwriting — a significant portion of EBA requirements are simply not applicable. Without business scope context, the system compared everything, generating substantial noise in the output.

-----

## New Architecture: Two-LLM Pipeline Replicating the Audit Workflow

The new approach was developed through a brainstorming session with Claude. The key insight that reframed the problem:

**This isn’t a RAG problem — it’s about replicating the audit workflow with AI.**

Audit does compliance gap analysis in four steps:

1. Understand what the regulatory requirement is actually saying — translate regulatory language into “this requires us to do X”
1. Judge whether it applies to this institution
1. Find the corresponding P&P clause
1. Make a coverage determination

The original RAG approach tried to do all four steps in one LLM call. Regulatory language is dense — a single article often packs multiple obligations into compound sentences. One LLM doing all four steps produces unstable output.

The new architecture splits this into two LLMs in sequence, each with a single job:

```
[External Preparation]
EBA Guideline PDF
→ Python chunking script (in progress — currently a skill gap)
→ Structured list: {id, chapter, article, text}

[Dify Workflow]
Input: EBA chunk list + P&P document
│
└── Iterator Node (loop over EBA chunks)
    │
    ├── LLM 1: Understanding Layer
    │   Input:  One EBA clause (raw regulatory text)
    │   Output: Structured interpretation
    │           {who, what, frequency, conditions,
    │            applicable: yes/no, reason}
    │
    └── LLM 2: Comparison Layer
        Input:  LLM 1 output (structured brief)
                + Full P&P document (as context)
        Output: Structured judgment
                {status, pp_reference, reasoning, confidence}

[Aggregation]
All outputs → structured table → compliance reviewer handoff
```

### LLM 1 — Understanding Layer

Reads each EBA clause and outputs a structured interpretation. Turns dense regulatory language into a clear brief: who must do what, by when, under what conditions, and whether it applies to our bank (using a business scope description in the system prompt).

### LLM 2 — Comparison Layer

Takes the structured brief from LLM 1 plus the full P&P as context, and outputs a structured judgment. Each output includes coverage status, P&P citation, reasoning, and confidence level.

Each LLM has one job. Output is more stable, and when something goes wrong, it’s clear which layer needs fixing.

-----

## Output Schema

```json
{
  "eba_id": "Art15-Para2",
  "eba_text": "original clause text",
  "interpretation": {
    "who": "credit institutions",
    "what": "conduct annual stress testing",
    "frequency": "at least annually",
    "applicable": true
  },
  "status": "Covered | Partially Covered | Not Covered | Not Applicable",
  "pp_reference": "Section 4.2, Page 23",
  "reasoning": "P&P Section 4.2 explicitly requires...",
  "confidence": "High | Medium | Low"
}
```

-----

## Why This Is More Suitable Than a Direct LLM Comparison

A simpler approach — loading both documents into an LLM and asking “is there a gap?” — may produce usable results, but has a structural problem for regulated environments:

*“The AI said it’s aligned”* is not an auditable conclusion.

Audit requires: which specific regulatory requirement, which P&P section, what is the reasoning, who reviewed it. The two-LLM pipeline produces this output by design — every row in the output table has a source reference and a reason. Low-confidence items are flagged for human review.

-----

## Current Status

|Component                     |Status       |Notes                                                                          |
|------------------------------|-------------|-------------------------------------------------------------------------------|
|Problem diagnosis             |✅ Complete   |Three root causes identified                                                   |
|New architecture design       |✅ Complete   |Developed via AI brainstorm                                                    |
|Python chunking script        |📋 Not started|Requires learning Python — in progress                                         |
|Dify Iterator + two LLM nodes |📋 Not started|Key open question: can two LLM nodes run sequentially inside one Iterator loop?|
|System prompt (business scope)|📋 Not started|Requires input from compliance team                                            |
|End-to-end test               |📋 Not started|—                                                                              |

-----

## Open Questions

- **Iterator + two LLMs**: Can Dify’s Iterator node support two sequential LLM nodes inside a single loop? Not yet validated.
- **Applicability accuracy**: Business scope filtering via system prompt will have edge cases; confidence scoring routes these to human review.
- **P&P context length**: Full P&P as context per LLM 2 call is feasible within Claude Sonnet’s context window, but needs validation with actual documents.

-----

## Lessons from the RAG Approach

1. RAG is a retrieval pattern, not a comparison pattern — the distinction matters
1. One LLM doing too many things at once produces unstable output — decompose tasks
1. Matching AI architecture to the human workflow it replaces (audit process) gives clearer design direction
1. When stuck on an AI architecture problem, using AI to brainstorm the solution is a legitimate and useful approach

-----

## Series Context

|Project                |Status      |
|-----------------------|------------|
|AI News Alarm          |✅ Deployed  |
|Regulatory Gap Analysis|🔧 Rebuilding|
|Sector Research Agent  |✅ Deployed  |
|

-----

*Author: Deployment Lab | AI-augmented credit risk analysis*
