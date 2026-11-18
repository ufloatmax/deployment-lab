# UC2 Regulatory Gap Analysis — Evidence-Grounded Policy Design Review Workflow

> **Status**: ⏸ Paused after end-to-end runs on four policy areas, one area at a time | **Platform**: Microsoft Copilot | **Scope**: Policy design assessment only | **Control Point**: Human reviewer owns final judgement

-----

## Overview

This workflow reviews internal credit policies against EBA guidelines and related regulatory sources, one requirement at a time, and identifies where policy design needs to change.

Within the capability of the models currently available internally, each run is limited to one policy area. A seven-stage, gate-controlled policy review workflow has been run end to end on **four policy areas**.

Development is now paused. With current model capabilities, further prompt optimisation offers diminishing returns. The next project is an Early Warning Engine.

This page documents the method only. Findings relate to internal policies and are not published.

-----

## Background: From RAG to an Audit-Style Workflow

The first version loaded EBA guidelines and internal policies into knowledge bases and asked an LLM whether the policy covered each requirement. That treated the task as retrieval, which was the wrong framing.

A regulatory requirement can combine a responsible party, a required action, applicability conditions, minimum standards, governance and approval expectations, documentation, and review triggers. A policy passage can be semantically similar to a regulatory paragraph and still miss a required element. Retrieval can find candidate evidence; it cannot decide whether that evidence is sufficient.

The workflow was therefore rebuilt to follow the way an auditor works: understand the requirement, decide whether it applies, find where policy addresses it, then judge coverage, with every step traceable to its source. Running this across all requirements at once proved beyond current internal model capability, so scope was narrowed to one policy area per run.

-----

## Scope Boundary: Policy Design, Not Execution Effectiveness

The workflow assesses **internal policy design only**.

It asks whether the internal documents contain the necessary policy principles, procedures, governance routes, responsibilities, documentation requirements, controls, or minimum standards.

It does not ask whether those policies were actually performed in live cases, whether audit samples prove execution, or whether operational teams followed the procedure in practice. Missing operating evidence is an execution verification issue, not a policy-design gap. It may be recorded as a follow-up item, but it does not create or remove a gap.

-----

## Workflow

### Design goal

Every final finding must be traceable from a regulatory source, through obligation extraction, applicability, policy location, clause mapping and cross-policy validation, to one focused and remediable policy requirement.

The method is built to avoid two common failure modes:

- **Regulation stacking**: collecting many sources and keyword-matching them against policy, which mixes legal requirements, supervisory expectations, best practice and control preferences.
- **Count-driven findings**: splitting one issue into several, or restating related issues, so the gap list grows long but cannot be turned into an executable policy change.

### Seven stages, each closed by a gate

| Stage | Purpose | Key control |
|---|---|---|
| 1. Document intake and baseline | Fix the document inventory: version, effective date, owner, controlled status, completeness | No substantive analysis until upload is explicitly complete and the inventory is frozen |
| 2. Regulatory as-of point | Record each source's version, amendment status, application date, exact provision and verified official link | Sources aligned to the policy's target implementation date and classified by authority type |
| 3. Institutional applicability profile | Legal entity, business model, exposures, accounting framework, reporting scope, governance | Profile must support an applicable / not applicable / partially applicable / to be confirmed call for every obligation |
| 4. Atomic obligation extraction | Decompose source text bottom-up: source → proposition → atomic obligation → applicability | Support-strength test on every candidate; the obligation count emerges, it is not set in advance |
| 5. Policy location test | Decide where each obligation should live before judging coverage | Every obligation assigned to one of four locations |
| 6. Primary mapping and cross-policy validation | Map to exact page, section and current text; then test related policies | Definitions, triggers, ownership and approval authority checked for consistency; hand-offs confirmed as received |
| 7. Focused gaps | Atomic, evidence-linked gaps with a clear boundary | Gap list limited to changes to the target policy or hand-offs it must establish; other issues go to a separate related-policy log |

Gates keep scope from drifting, stop conclusions from being reasoned backwards from a preferred answer, and keep rework local: a wrong source version sends the review back to one stage, not to the start.

### Support-strength test

| Category | Treatment |
|---|---|
| Direct requirement | Obligation register |
| Necessary implication | Obligation register |
| Supervisory interpretation | Obligation register |
| Reasonable control design | Separate control-design recommendations log |
| No support | Removed |

This prevents a reviewer's preferred process design from being presented as a regulatory obligation.

### Policy location test

"The regulation requires it and the target policy does not say it, so it is a gap" is not always true. Each obligation is placed in one of four locations:

1. Core rule that must be written into the target policy (scope, definitions, judgement principles, responsibilities, approvals).
2. Detail that may sit in another document, provided the target policy establishes a controlled reference.
3. Fully carried by another controlled document, provided hierarchy, ownership and interfaces are clear.
4. Implementation or operating evidence. Its absence from the policy text is not a policy gap.

A controlled reference names the referenced document, the delegated matter, the trigger, the responsible function, the independent review or approval function, the record produced, where it is stored, and the escalation route. A "related documents" list on the cover page is not enough.

### Human validation, drafting and implementation

- **Independent validation gate.** After gap assessment, a reviewer with regulatory, risk, accounting or business expertise confirms the regulatory basis, applicability, whether other controlled documents were missed, whether the gap requires a change to the target policy, and whether findings should be merged. Only validated gaps proceed to drafting.
- **Formal policy language.** Drafts state who acts, on what trigger, what assessment is required, who independently reviews or approves, by when, what record is produced, where it is kept, and how exceptions escalate. Advisory wording such as "consider strengthening" is not accepted.
- **No automatic cross-framework conclusions.** Where prudential classification, accounting classification and regulatory reporting interact, each framework's determination is made and recorded separately.
- **Tracked changes.** Approved wording is inserted into the current effective version as tracked changes, each mapped to its gap ID, rather than rewriting an "ideal" policy.

### End-to-end trace

```text
Regulatory source
→ exact provision
→ source proposition
→ atomic obligation
→ applicability conclusion
→ policy location
→ primary policy mapping
→ cross-policy validation
→ focused gap
→ human validation
→ proposed policy wording
→ tracked change
→ approval and implementation evidence
```

The trace runs in both directions: from any tracked change, a reviewer can reach the gap, the obligation and the regulatory source.

### Deliverables

- **Management review report**: what needs to change, why, and who owns it.
- **Reproducible workpaper**: document inventory, regulatory sources, institutional profile, source propositions, atomic obligations, policy locations, primary mapping, cross-policy tests, focused gaps, related-policy issues, evidence limitations, stage gates, open items, change log, end-to-end trace and data-quality checks. Stable unique IDs allow the analysis to be reproduced without chat history or personal notes, and data-quality checks flag broken trace links, obligations without sources, unverified links or page references, and blanks confused with "not applicable" or "not available".

-----

## Results

### What these runs show, and what they do not

- **Shows**: the full chain runs end to end, one policy area at a time, across four policy areas.
- **Scope and stability**: with the model available to the bank today, the narrower the policy area, the more consistent the output from run to run. The broadest area, with the most interfaces to other policies, was the least consistent. This is a limit of model capability, not of the workflow design.
- **Does not yet show**: accuracy. Output that is consistent across runs is not necessarily correct; whether a finding is right remains a reviewer's call.

-----

## Current Status

| Component | Status | Notes |
|---|---|---|
| Seven-stage gated workflow | ✅ Run end to end | Four policy areas, one area per run |
| Run-to-run consistency | ⚠️ Depends on scope | Narrower areas more consistent; the broadest area least consistent |
| Accuracy | 📋 Not yet formally measured | Consistent output is not necessarily correct; reviewer judgement required |
| Findings | 🔒 Internal | Only the method is published |
| Development | ⏸ Paused | Further prompt optimisation offers diminishing returns with current models |

-----

## Lessons Learnt

1. The task is not "make RAG better"; it is evidence-grounded regulatory reasoning. Retrieval finds candidate evidence, but does not decide whether it is sufficient.
1. Policy design and execution effectiveness must be kept separate.
1. The model sets the scope. With current internal models, narrowing to one policy area at a time made end-to-end runs possible, and the narrower the area, the more consistent the output.
1. A requirement missing from the target policy is not automatically a gap: test where it should live first.
1. Lock the document set and the regulatory as-of date before forming any view; early conclusions cause the most rework.
1. Consistent is not the same as correct. Agreement across runs shows the prompt constrains the model; it does not show the findings are right.
1. False covered rate is the most important error to control in a banking environment.

-----

## Series Context

| Project | Status |
|---|---|
| AI News Alarm | ✅ Deployed |
| Regulatory Gap Analysis | ⏸ Paused after end-to-end runs on four policy areas |
| Sector Research Agent | ✅ Deployed |
| New Financing Due Diligence Workflow | ✅ Shared |

-----

*Deployment Lab | AI-augmented credit risk analysis*
