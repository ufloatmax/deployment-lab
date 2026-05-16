# UC3 Sector Research Agent

> **Status**: ✅ Deployed & Active | **Platform**: Microsoft Copilot (M365 Enterprise) | **Architecture**: Copilot Agent + Prompt Template | **Review Loop**: Credit Expert in-the-loop

-----

## Overview

Sector Research Agent is a Copilot Agent deployed on Microsoft Copilot (M365 Enterprise), designed to automate the research and synthesis phase of credit sector analysis.

**Implementation note**: This is a Copilot-native Agent — Copilot’s internal workflow (web search, synthesis, generation) runs as-is. What is customised is the **prompt template** that governs the Agent’s behaviour: what to research, how to filter, which credit framework to apply, what output format to produce, and critically — what to do when information is not found (state it directly; do not infer or fabricate). The Agent’s internal execution is Copilot’s own; the analytical discipline is encoded in the prompt.

-----

## Architecture

```
User Input (Sector Name)
        │
        ▼
┌─────────────────────────────────────┐
│   Copilot Agent (M365)              │
│                                     │
│   Defined by Prompt Template:       │
│   ├── Research scope & sources      │
│   ├── Filtering criteria            │
│   ├── Credit analysis framework     │
│   ├── Output schema                 │
│   └── Anti-hallucination rules      │
│       (no info → say so, not guess) │
│                                     │
│   Executed by Copilot natively:     │
│   ├── Web search & retrieval        │
│   ├── Information synthesis         │
│   └── Report generation             │
└────────────────┬────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────┐
│   Credit Expert Review Loop         │
│   - Human validation (2nd defence)  │
│   - Correction & annotation         │
│   - Approval / re-run               │
└─────────────────────────────────────┘
```

-----

## Design Philosophy

The core constraint of Copilot Agent deployment is that the internal workflow is fixed — custom nodes, conditional branches, and step-level logic are not available. What you control is the prompt.

This shifts the design challenge from *workflow engineering* to *prompt engineering*: encoding the full analytical sequence into a template that reliably produces consistent, credit-relevant output across different sectors and time periods.

The prompt template was co-developed iteratively with Copilot itself, stress-testing instructions and identifying gaps until output quality stabilised.

### Anti-Hallucination: Two-Layer Defence

Hallucination is a specific risk in credit contexts — a plausible but incorrect data point can affect the entire analysis.

**Layer 1 — Prompt-level prevention**: The template explicitly instructs the Agent: if information is not found, state that directly. No inference, no gap-filling, no fabrication.

**Layer 2 — Expert review loop**: Prompt instructions reduce hallucination tendency but cannot eliminate it entirely. Every output is reviewed by a credit expert before use.

### Why Copilot Agent (not Dify, not custom Python)?

Selected for this phase due to:

- **Zero infrastructure overhead**: Agent creation within existing M365 enterprise licence, no IT procurement
- **Fast iteration**: Prompt changes are instant — no deployments, no pipeline restarts

**Key limitations**:

- No custom workflow definition — complex logic (conditional branches, multi-step triggering) is not achievable
- **Black-box process**: The report generation is opaque — there is no visibility into each step’s input and output. This is a meaningful audit and compliance gap. Dify’s workflow, by contrast, exposes input/output at every node, making the process traceable and audit-ready.

> **Planned Phase 2**: Dify implementation, pending internal plugin access approval. Dify enables visual workflow editing with node-level customisation — UC3 would upgrade from prompt-driven to workflow-driven, with full step-by-step traceability.

-----

## Two Usage Patterns

In practice, the Agent has settled into two distinct use cases depending on familiarity with the sector:

**Pattern 1 — Known sectors (existing portfolio)**
Analysts review the generated report directly. It serves as a structured information digest to confirm “has anything material changed?” Research that previously took 2–3 days is compressed to half a day.

**Pattern 2 — New sectors (no prior coverage)**
The report is not read directly. Instead, it is used as a **high-quality source list** — analysts read the cited primary sources themselves, build their own understanding, and write their own report independently.

The rationale: for unfamiliar markets, analysts need to form their own judgment framework without AI-mediated conclusions. The Agent’s value here is in identifying which sources are worth reading — the most time-consuming step when entering a new sector. Research that previously took 1–2 weeks is compressed to 3–4 days.

-----

## Credit Expert Review Loop

The review loop is a core design feature, not optional:

1. **Accuracy gate**: Even with anti-hallucination prompts, LLMs can produce plausible but incorrect sector data. Expert review is the final check before outputs inform credit decisions.
1. **Regulatory requirement**: EU AI Act and EBA guidelines require human oversight for AI outputs that inform material credit decisions. Human-in-the-Loop is a compliance requirement.
1. **Prompt improvement signal**: Expert corrections accumulate as feedback for iterative prompt refinement.

-----

## Quantitative Impact

|Scenario                      |Before   |After                 |Change           |
|------------------------------|---------|----------------------|-----------------|
|Known sector — periodic update|2–3 days |Half day (review only)|~80% reduction   |
|New sector — initial research |1–2 weeks|3–4 days              |~50–70% reduction|


> Figures based on internal usage and analyst estimates.

-----

## Known Limitations

|Issue                                                  |Severity|Status                    |
|-------------------------------------------------------|--------|--------------------------|
|Black-box process — no step-level audit trail          |High    |Dify migration planned    |
|No custom workflow (conditional logic, API integration)|Medium  |Dify migration planned    |
|Hallucination risk (mitigated, not eliminated)         |Medium  |Two-layer defence in place|
|No persistent memory across sessions                   |Low     |By design                 |

-----

## Roadmap

- [ ] Dify implementation once plugin access approved (adds workflow visibility and audit trail)
- [ ] Structured time tracking to formalise quantitative impact
- [ ] Expand sector coverage

-----

*Deployment Lab | AI-augmented credit risk analysis*
