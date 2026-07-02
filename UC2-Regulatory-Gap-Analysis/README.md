# UC2 Regulatory Gap Analysis — From LLM Judgement to Quantitative Scoring

> **Status**: 🔄 v3 Quantitative Scoring Complete | **Platform**: Microsoft Copilot | **Scope**: Policy design assessment only | **Next Step**: Five-gate coverage decision model

-----

## Overview

This use case automates regulatory gap analysis between internal Policies & Procedures (P&P) and EBA Guidelines on loan origination and monitoring (EBA LOM). The workflow has evolved from a RAG-based comparison, to a decomposed LLM pipeline, and now to a quantified retrieval and scoring model.

The previous architecture upgrade split applicability judgement and coverage judgement so that each module had one job. The single-requirement end-to-end flow has now been tested. This version continues that work, with two major changes:

- The platform has moved from Dify to Microsoft Copilot.
- Python-based preprocessing and analysis code is now being developed with GPT-5.5 assistance.

The bigger change, however, is conceptual: the workflow now defines much more precisely what it is actually assessing.

-----

## The Boundary Decision: What Is Being Assessed?

The phrase "does the internal policy meet the regulatory requirement?" sounds simple, but it contains two different questions.

**1. Policy design level**

This asks whether the policy principles are written, procedures are defined, governance routes are clear, responsibilities are assigned, and required wording or components are present in the internal documentation.

**2. Execution level**

This asks whether those policy designs are actually carried out in practice, and whether audit samples or operating evidence can prove execution.

These are different questions. They require different inputs, different analysis logic, and serve different review purposes.

Earlier versions did not separate them clearly enough, so the output could contain findings like:

> "The internal policy covers this regulatory requirement, but audit samples are needed to confirm implementation."

That looks reasonable at first, but it mixes two assessment layers. Missing audit samples are not a policy design gap; they are a separate execution verification question.

This version therefore makes a clear boundary decision:

> **The workflow assesses internal policy design only. It does not assess execution effectiveness.**

Execution evidence can still be flagged as a follow-up verification item, but it should not appear in the policy design gap list.

-----

## End-to-End Architecture

The current workflow has 13 stages across four phases.

### Phase A — Structuring (S1-S5)

1. Extract 277 atomic EBA LOM regulatory requirements.
1. Run four-level applicability judgement.
1. Extract evidence units from internal policy documents.
1. Build a policy ontology.
1. Build a cross-reference graph.

### Phase B — Retrieval (S6-S7)

6. Run six-signal quantitative hybrid retrieval.
1. Build an evidence pack for each applicable regulatory requirement.

### Phase C — Scoring (S8-S11)

8. Apply a 10-item policy design checklist.
1. Classify coverage using score thresholds.
1. Run substantive element tests for requirements that contain specific mandatory elements.
1. Apply conservative override where the evidence is insufficient or contradictory.

### Phase D — Classification and Prioritisation (S12-S13)

12. Classify design-level gaps into four practical categories.
1. Assign review priority scores for human reviewer workflow.

```
EBA LOM requirements + internal policy documents
        |
        v
[A. Structuring]
Atomic requirements -> applicability -> evidence units -> ontology -> graph
        |
        v
[B. Retrieval]
Six-signal hybrid retrieval -> evidence pack
        |
        v
[C. Scoring]
10-item checklist -> threshold class -> substantive tests -> conservative override
        |
        v
[D. Reviewer Handoff]
Design gap category -> priority score -> human review
```

-----

## Why Move from LLM Judgement to Quantitative Scoring?

The previous framework relied heavily on LLM judgement to classify each requirement as Covered, Partially Covered, or Gap. The categories were useful, but the judgement process had two weaknesses.

**First, it was not transparent enough.** When the model returned "Partially Covered", it was hard to see which dimension was weak, how weak it was, and what evidence drove the conclusion.

**Second, it was not stable enough.** The same requirement could produce a different classification on rerun, without a clear way to tell whether the change came from new evidence or model randomness.

The current version introduces quantitative retrieval and checklist-based scoring. The point is not to pretend that the number is perfectly precise. The point is to make the judgement traceable, challengeable, and replaceable.

-----

## Six-Signal Hybrid Retrieval

The retrieval stage uses a weighted hybrid formula:

```text
hybrid_score = 0.30 × lexical_score          // BM25 / word-level TF-IDF
             + 0.20 × lsa_score              // truncated SVD semantic proxy
             + 0.15 × char_tfidf_score       // character n-gram similarity
             + 0.15 × keyword_overlap_score  // Jaccard keyword overlap
             + 0.15 × metadata_score         // family / lifecycle / scope match
             + 0.05 × graph_score            // cross-reference graph score
```

Each component score is stored separately, and the formula text is saved alongside the result. If a retrieved evidence pack looks wrong, the reviewer can inspect which signal pulled the score up or down.

This is a practical auditability improvement: retrieval quality can now be reviewed signal by signal, instead of being treated as a black-box LLM output.

-----

## Ten-Item Policy Design Checklist

The coverage scoring stage uses a 10-item binary checklist. Each item is scored 1 or 0, then divided by 10.

Examples of design checks include:

- Is the policy principle stated?
- Are procedure steps defined?
- Is governance or approval responsibility clear?
- Are required roles or ownership identified?
- Are escalation, exception, or monitoring mechanisms described where relevant?
- Is the evidence tied to the right lifecycle stage and policy scope?

Coverage thresholds:

| Checklist score | Default result |
|---|---|
| ≥ 0.80 | Covered |
| 0.45-0.80 | Partially covered |
| < 0.45 | Not covered |

The checklist does not replace expert review. It makes the route to the classification visible.

-----

## Substantive Element Testing: P188 Example

The checklist captures general design completeness, but some EBA requirements include specific mandatory elements. For those, a separate substantive test is needed.

For example, EBA LOM P188 concerns the main repayment source and cash flow assessment for project finance. The assessment should explicitly cover four substantive elements:

1. The main repayment source is project income.
1. Project cash flow is assessed.
1. Future income-generating capacity after completion is assessed.
1. Regulatory and legal constraints affecting profitability are explicitly considered, such as price regulation, availability-based payment arrangements, or environmental legislation.

An illustrative calculation under the current logic:

| Field | Result |
|---|---|
| policy_checklist_score | 0.70 (7/10) |
| substantive_p188_score | 0.75 (3/4 substantive elements met) |
| default_quantitative_result | Partially covered |
| conservative_override_applied | No |
| **final_design_coverage** | **Partially covered** |

The missing item is the fourth substantive element: the policy covers project repayment source, cash flow analysis, and credit committee governance, but does not explicitly list the types of regulatory or legal constraints required by EBA.

This is a **policy wording gap**. It is not a missing operating process. The remediation is different, so the gap category needs to be different.

-----

## Gap Classification

This version replaces the previous six-dimensional severity scoring and eight-category gap taxonomy with a smaller set of design-level categories.

The previous taxonomy looked comprehensive, but it was difficult for reviewers to act on. When a single requirement had six dimensions scored from 1 to 3, the reviewer still had to interpret what to do next.

The current structure is simpler:

- The checklist shows how many design components are covered.
- The threshold gives the default coverage class.
- The substantive test shows what specific required element is missing.
- The gap category points to the practical remediation route.

Current design-level gap categories are limited to four:

| Gap category | Meaning |
|---|---|
| Policy wording gap | The concept exists, but required wording or specific elements are missing |
| Procedure design gap | The policy principle exists, but procedure steps are not sufficiently defined |
| Governance design gap | Ownership, approval, escalation, or committee responsibility is unclear |
| No policy coverage | No relevant policy design evidence was found |

-----

## Conservative Override

Quantitative scores should not automatically become final conclusions. A conservative override can still change the final design coverage where the evidence is weak, conflicting, or too indirect.

Examples:

- A high checklist score is based on evidence from the wrong policy scope.
- Retrieved evidence is semantically similar but not actually responsive to the requirement.
- Substantive mandatory elements are missing even though general policy design is strong.
- Internal policy documents conflict with each other.

This keeps the scoring model useful without letting the score overrule reviewer judgement.

-----

## Current Status

| Component | Status | Notes |
|---|---|---|
| RAG failure diagnosis | ✅ Complete | Retrieval-only architecture abandoned for structured comparison |
| Applicability / coverage split | ✅ Complete | Each module has one primary judgement task |
| Single-requirement end-to-end flow | ✅ Complete | Flow validated on one requirement |
| Platform migration | ✅ Complete | Workflow moved from Dify to Microsoft Copilot |
| Python processing code | 🔄 In progress | Developed with GPT-5.5 assistance |
| Quantitative retrieval | ✅ Complete | Six-signal hybrid formula defined |
| Checklist scoring | ✅ Complete | 10-item binary policy design checklist implemented conceptually |
| Substantive element testing | ✅ Complete | Illustrated through P188 logic |
| Coverage decision model | 📋 Next | Upgrade from threshold scoring to five-gate decision model |

-----

## Next Step: Five-Gate Coverage Decision Model

The current coverage decision still has a limitation: a checklist score of 0.80 or above can become "Covered", which still treats a high score as equivalent to sufficient coverage.

The next version will replace this with five gates that must be passed sequentially:

1. Applicability judgement
1. Evidence sufficiency judgement
1. Necessary component judgement
1. Substantive element judgement
1. Conflict detection

This should make the coverage decision more robust than a threshold score alone.

-----

## Lessons Learnt

1. Define the assessment boundary before designing the model. Policy design and execution effectiveness are different questions.
1. LLM judgement is useful, but unsupported categorical output is too opaque for compliance review.
1. Quantitative scoring is valuable when it improves traceability, not because the number is inherently precise.
1. Substantive element tests are needed where a regulation requires specific named components.
1. Simpler gap categories can be more useful than a more comprehensive taxonomy if they map directly to remediation actions.

-----

## Series Context

| Project | Status |
|---|---|
| AI News Alarm | ✅ Deployed |
| Regulatory Gap Analysis | 🔄 v3 quantitative scoring complete; next step is five-gate coverage decision |
| Sector Research Agent | ✅ Deployed |
| Junior Credit Analyst | 🧪 Workflow testing |

-----

*Deployment Lab | AI-augmented credit risk analysis*
