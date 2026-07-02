# UC2 Regulatory Gap Analysis — Evidence-Grounded Policy Design Review Workflow

> **Status**: 🔄 V10 component-level workflow in calibration | **Platform**: Microsoft Copilot + Python | **Scope**: Policy design assessment only | **Control Point**: Human reviewer owns final judgement

-----

## Overview

This use case has evolved from a simple RAG experiment into an **evidence-grounded regulatory reasoning workflow** for policy-design gap assessment.

The original assumption was that the problem was mainly about retrieval: load EBA Guidelines on loan origination and monitoring (EBA LOM) and internal Policies & Procedures (P&P) into a knowledge base, retrieve relevant passages, and ask an LLM whether the internal policy covers the regulatory requirement.

That turned out to be the wrong task framing. Retrieval can find potentially relevant evidence, but the real problem is deciding whether the evidence is sufficient to cover a regulatory obligation in a way that is traceable, reviewable, and defensible.

The V10 architecture therefore separates responsibilities:

```text
Regulatory text
→ decomposition
→ evidence retrieval
→ component-level LLM judgement
→ deterministic gate engine
→ reviewer validation
→ final workbook / audit pack
```

Copilot is no longer positioned as the final decision-maker. It acts as a controlled component-level judgement assistant. Python handles deterministic gates, roll-up rules, audit trail generation, workbook output, and version comparison. Human reviewers confirm final status, materiality, overrides, and remediation actions.

A more accurate description of this use case is:

> **A policy-design gap assessment assistant that supports evidence-grounded review, deterministic roll-up, reviewer validation, and audit trail generation.**

It is not a compliance decision system and does not assess execution effectiveness.

-----

## Core Lesson: Retrieval Is Not the Compliance Conclusion

RAG is useful for evidence discovery, but regulatory gap analysis is not a standard retrieval-and-answer task.

A regulatory requirement can contain:

- Subject and responsibility
- Required action
- Scope and applicability conditions
- Minimum standards or thresholds
- Governance and approval expectations
- Procedure requirements
- Documentation requirements
- Controls, exceptions, and review triggers
- Specific substantive objects that must be considered

A policy passage can be semantically similar to a regulatory paragraph while still failing to cover a required legal component. For example, internal policy may mention general borrower creditworthiness, but not the specific cash-flow, governance, documentation, or minimum-standard element required by EBA.

The workflow therefore treats retrieval scores as support for evidence discovery only. They do not directly determine whether a requirement is covered, partially covered, or a gap.

-----

## Scope Boundary: Policy Design, Not Execution Effectiveness

The workflow assesses **internal policy design only**.

It asks whether the internal documents contain the necessary policy principles, procedures, governance routes, responsibilities, documentation requirements, controls, or minimum standards.

It does not ask whether those policies were actually performed in live cases, whether audit samples prove execution, or whether operational teams followed the procedure in practice.

This boundary matters because earlier versions could produce statements such as:

> "The policy covers this requirement, but audit samples are needed to confirm implementation."

That mixes two different questions. Missing audit samples are not a policy-design gap. They are an execution verification issue. Execution evidence may be recorded as a follow-up item, but it should not be used to create or remove a policy-design gap.

-----

## Target Architecture

```text
Python:
PDF extraction
chunking
evidence units
hybrid retrieval
component bundle construction
deterministic gate engine
audit trail
workbook generation
version comparison

Copilot:
component-level legal sufficiency judgement
reasoning draft
gap explanation draft

Human reviewer:
final judgement
override
materiality validation
remediation decision
```

The operating principle is simple:

> Copilot performs controlled component judgement, Python performs reproducible decision logic, and compliance / credit experts perform final review.

This architecture is intended for an internal banking environment: it does not rely on external APIs for the core workflow, does not transfer final compliance judgement to an LLM, and preserves reviewer override plus a full audit trail.

-----

## Phase 1: Regulatory Decomposition

**Input**: EBA LOM P001-P277

**Output**:

```text
EBA paragraph
→ regulatory atom
→ material legal element
→ typed legal component tests
```

The project has moved beyond matching entire regulatory paragraphs against internal policy text. Each regulatory paragraph is decomposed into smaller units that can be judged separately.

Current decomposition direction:

```text
EBA paragraph
→ regulatory atom
→ legal element
→ typed legal component
→ evidence bundle
→ component judgement
→ element roll-up
→ requirement-level decision
```

Each component becomes a question that can be adjudicated against evidence:

```text
component_id
component_type
component_question
must_show
acceptable_evidence
not_sufficient_if
materiality
```

Example:

```text
component_type: procedure_test
component_question: Do the bank's internal P&P define a procedure to perform regular credit reviews for medium-sized or large enterprises?
must_show: explicit review procedure, review frequency, or creditworthiness reassessment step
acceptable_evidence: policy clause, procedure step, credit assessment template, approval checklist
not_sufficient_if: only generic borrower monitoring is mentioned
materiality: material
```

Indicative component taxonomy:

| Component type | Question it tests |
|---|---|
| action_test | Does policy require the action demanded by the regulation? |
| responsibility_test | Is ownership or accountable party defined? |
| scope_test | Does the policy scope match the regulatory scope? |
| condition_trigger_test | Are triggering conditions captured? |
| threshold_test | Are minimum standards or thresholds included? |
| governance_test | Are approval, escalation, or committee routes clear? |
| control_test | Are controls or monitoring mechanisms designed? |
| procedure_test | Are procedure steps defined? |
| documentation_test | Are records, evidence, or documentation requirements stated? |
| exception_test | Are exceptions or overrides governed? |
| substantive_object_test | Are specific required factors or objects explicitly covered? |

-----

## Phase 2: Component-Level Evidence Retrieval

**Input**: typed component tests + internal P&P evidence units

The mature part of the workflow is the evidence discovery layer. Current structured assets include:

- 277 EBA paragraphs / requirements
- 536 legal elements
- 865 evidence units extracted from 12 internal P&P documents
- Hybrid retrieval with stored BM25, LSA, character TF-IDF, keyword overlap, metadata, and graph score components

Earlier versions focused mainly on paragraph-level retrieval. V10 adds component-level retrieval because different components require different evidence.

For example:

```text
action_test query
scope_test query
procedure_test query
documentation_test query
governance_test query
```

The evidence bundle combines multiple retrieval routes:

```text
paragraph-level top evidence
+ component-specific evidence
+ cross-reference evidence
+ same-section neighbouring evidence
```

Evidence bundle output:

```text
evidence_bundle_id
evidence_unit_ids
retrieval_scores
retrieval_reason
source_document
section/page reference
```

Retrieval scores remain visible and auditable, but they are not coverage decisions.

-----

## Phase 3: Copilot / LLM Component Judge

Copilot answers one controlled question at a time:

> Given this component test and this evidence bundle, is this component covered by the bank's internal policy design?

The LLM is constrained to the supplied evidence bundle and must output a fixed schema:

```json
{
  "component_id": "...",
  "coverage_label": "Explicitly covered | Implicitly covered | Weak / partial | Not evidenced | Contradicted / below minimum | Not applicable",
  "cited_evidence_ids": ["..."],
  "cited_snippets": ["..."],
  "reasoning": "...",
  "gap_type": "policy wording gap | scope gap | procedure gap | governance gap | documentation gap | control gap | minimum standard gap | none",
  "materiality": "material | non-material",
  "confidence": "high | medium | low",
  "review_required": true
}
```

Key judgement rules:

- The LLM cannot cite content outside the evidence bundle.
- Without an evidence ID, it cannot label a component as covered.
- Implicit coverage does not automatically equal design covered.
- Context can support applicability, but cannot erase an identified gap.
- Execution evidence is excluded from policy-design gap conclusions.

This turns Copilot into a bounded component judge, not an open-ended compliance opinion generator.

-----

## Phase 4: Deterministic Gate Engine

Final status is not determined by Copilot and not by averaging similarity scores.

Python applies deterministic gates and roll-up rules. Same input plus same rules should produce the same output.

Four decisive gates:

```text
Gate 1: Applicability
Gate 2: Evidence source adequacy
Gate 3: Typed component coverage
Gate 4: Conflict / below-minimum internal standard
```

Illustrative roll-up logic:

```text
if Gate 1 = Not applicable:
    final_status = Not applicable

elif Gate 1 = Conditionally applicable:
    final_status = Conditionally applicable

elif Gate 2 = Insufficient evidence:
    final_status = Insufficient evidence

elif any component = Contradicted / below minimum:
    final_status = Potential policy conflict

elif any mandatory material component = Not evidenced:
    final_status = Not design covered

elif any material component = Weak / partial:
    final_status = Partially covered

elif any material component = Implicitly covered:
    final_status = Partially covered unless reviewer override

else:
    final_status = Design covered
```

This is a major change from the previous threshold approach, where a high checklist score could become "Covered". In V10, required components and hard gates matter more than averages.

-----

## Phase 5: Audit Trail and Evidence Pack

Every EBA paragraph must retain a complete audit trail:

```text
eba_paragraph_id
eba_text_hash
legal_element_ids
component_ids
evidence_bundle_id
evidence_unit_ids
retrieval_score_components
llm_prompt_version
llm_output_version
component_labels
rollup_rule_fired
final_status_before_override
reviewer_override_status
reviewer_name
review_date
override_reason
```

Primary outputs:

```text
1. assessment_workbook.xlsx
2. audit_pack.json / csv
```

The workbook is designed for reviewer workflow. The audit pack is designed for model governance, internal audit, second-line challenge, and future version comparison.

-----

## Phase 6: Reviewer Workflow

Reviewers should not be asked to read 277 full records in flat order. The workflow prioritises cases by risk and review need.

Priority levels:

| Priority | Records |
|---|---|
| Priority 1 | Potential policy conflict; Not design covered; Insufficient evidence on material requirement |
| Priority 2 | Partially covered; Implicitly covered material components; Low-confidence LLM judgement |
| Priority 3 | Design covered with high confidence; Not applicable |

Reviewer fields:

```text
agree_with_ai
override_status
override_reason
required_policy_action
owner
target_date
management_comment
```

The reviewer is not expected to rerun the AI. The reviewer validates:

- Whether the cited evidence is correct
- Whether the gap is real
- Whether materiality is reasonable
- Whether remediation ownership is clear

-----

## Phase 7: Gold Set Calibration

Before running all 277 EBA paragraphs, the next step is a 30-item calibration set.

The gold set should cover:

```text
clearly covered
clearly not covered
partial wording gap
scope gap
procedure gap
documentation gap
governance gap
minimum standard gap
conditionally applicable
insufficient evidence
potential conflict
```

Calibration workflow:

```text
AI output
→ deterministic gate result
→ human reviewer label
→ error log
→ prompt / rule adjustment
```

Key metrics:

```text
status agreement rate
material gap recall
false covered rate
evidence citation accuracy
reviewer override rate
```

The most important control metric is **false covered rate**. In a banking control environment, classifying a real gap as covered is more dangerous than classifying a covered item as partial.

-----

## Phase 8: Prompt and Rule Impact Analysis

Every change to prompts, component taxonomy, or gate rules should be tested against the same gold set.

Comparison examples:

```text
v9.1 vs v10.0
prompt_a vs prompt_b
gate_rule_v1 vs gate_rule_v2
```

Impact report output:

```text
status changed paragraphs
component labels changed
new gaps created
old gaps removed
reason for change
```

Without an impact report, the production version should not be replaced.

-----

## Evolution from Earlier Versions

### Initial RAG Approach

The first design treated the task as a retrieval problem. EBA requirements and internal P&P documents were loaded into knowledge bases and compared directly. This was abandoned because RAG is not designed for clause-by-clause, auditable regulatory gap assessment.

### Pipeline Decomposition

The next design split applicability judgement from coverage judgement. This was an important correction: one module determines whether the requirement applies; another assesses whether internal policy design covers it.

### Quantitative Retrieval and Checklist Scoring

The following version introduced hybrid retrieval scores and a 10-item policy design checklist. This improved traceability, but the coverage decision still relied too heavily on scoring thresholds.

### V10 Component-Level Workflow

V10 keeps the evidence discovery improvements, but moves the core judgement to typed component tests, controlled LLM component judgement, deterministic gates, and human reviewer validation.

-----

## Current Status

| Component | Status | Notes |
|---|---|---|
| RAG failure diagnosis | ✅ Complete | Retrieval-only framing abandoned |
| Policy-design scope boundary | ✅ Complete | Execution effectiveness excluded from gap conclusion |
| Regulatory decomposition | 🔄 In progress | EBA paragraphs → atoms → legal elements → component tests |
| Evidence unit extraction | ✅ Initial version complete | 865 evidence units from 12 internal P&P documents |
| Hybrid retrieval | ✅ Initial version complete | BM25, LSA, char TF-IDF, keyword overlap, metadata, graph scores retained |
| Component-level retrieval | 🔄 In design | Retrieval by action, scope, procedure, documentation, governance, and other component types |
| Copilot component judge | 🔄 In design | Fixed-schema judgement constrained to evidence bundle |
| Deterministic gate engine | 🔄 In design | Python roll-up and hard gates replace threshold-only decisioning |
| Workbook / audit pack | 📋 Next | Reviewer workbook plus JSON / CSV audit pack |
| Gold set calibration | 📋 Next | 30-item reviewer-labelled calibration set before full 277-paragraph run |
| Prompt / rule impact analysis | 📋 Next | Version comparison required before production replacement |

-----

## Lessons Learnt

1. The task is not "make RAG better"; it is evidence-grounded regulatory reasoning.
1. Retrieval finds candidate evidence, but does not decide compliance sufficiency.
1. Policy design and execution effectiveness must be kept separate.
1. Component-level judgement is more reviewable than paragraph-level similarity.
1. Final status should be produced by deterministic gates, not by LLM confidence or average scores.
1. Gold set calibration is necessary before scaling to all 277 requirements.
1. False covered rate is the most important error to control in a banking environment.

-----

## Series Context

| Project | Status |
|---|---|
| AI News Alarm | ✅ Deployed |
| Regulatory Gap Analysis | 🔄 V10 component-level workflow in calibration |
| Sector Research Agent | ✅ Deployed |
| Junior Credit Analyst | 🧪 Workflow testing |

-----

*Deployment Lab | AI-augmented credit risk analysis*
