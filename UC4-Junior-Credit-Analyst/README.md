# UC4 Using AI as a Junior Credit Analyst

> **Status**: 🧪 Workflow Testing | **Platform**: Microsoft Copilot | **Primary Pattern**: Guided document review + source-cited drafting | **Control Point**: Credit professional owns the risk judgement

-----

## Overview

This use case documents a practical workflow for using Copilot in due diligence and portfolio monitoring. The central design principle is to treat AI as a **junior credit analyst who is new to the job and needs guidance**, not as an expert system that can produce final credit conclusions directly.

The workflow was developed through live testing rather than a pre-designed methodology. Its value comes from using Copilot to read, organise, cite, and draft, while keeping human judgement and review at the centre of the process.

The examples described here are general project finance and post-close credit management scenarios. They illustrate the workflow pattern only and do not correspond to any specific client, loan, transaction, or actual request.

-----

## Why the Junior Analyst Analogy Works

A junior analyst's first draft is not expected to be perfect. The useful part is that they can read through a stack of documents, extract relevant information, organise it into a usable format, and show exactly where each finding came from. The senior reviewer then questions, challenges, corrects, and forms the judgement.

Copilot is most useful when held to the same standard:

- It must organise information, not replace credit judgement.
- It must cite the source for every finding.
- Its conclusions must be questioned through follow-up review.
- The final risk judgement remains with the credit professional.

This framing keeps the reviewer in a healthy state of scepticism. The key question becomes: "Where does that come from?" rather than "Does this sound convincing?"

-----

## Workflow

```
Document pack
        |
        v
Read one document at a time
        |
        v
Source-cited summary
        |
        v
Reviewer Q&A and challenge
        |
        v
Human-owned risk judgement
        |
        v
Copilot drafts credit opinion
        |
        v
Credit professional edits final version
```

### Step 1: Read and Summarise with Source References

The prompt explicitly requires Copilot to provide a source for every finding: document name, section or clause, and page number where available.

Documents are fed one by one rather than as a single bundle. Typical inputs include:

- Main project or commercial contract
- Credit or financing agreement
- Technical adviser report
- Legal opinion
- Insurance documents
- Financial due diligence report

This looks like a formatting requirement, but it is really a control mechanism. Once every finding has a source reference, the reviewer can verify the original text directly, spot fabricated or misattributed content more easily, and reuse the citation later in the credit opinion.

For compliance-heavy work, this turns AI output from "looks comprehensive" into something closer to reviewable work product.

### Step 2: Challenge the Judgement Through Q&A

The first summary is reviewed line by line, similar to reviewing a junior analyst's draft:

- Why was this item flagged as a risk?
- Do two documents say different things about this mechanism?
- Was the main agreement cross-checked against the financing documents?
- Is this risk confirmed, mitigated, or still an open point?

Most useful risk issues surface during this challenge process, not in the initial summary.

### Step 3: Form the Overall Risk Judgement

After the Q&A cycle, the reviewer has built up their own view of the weak points, mitigants, open confirmations, and overall credit relevance.

The most important principle is:

> AI helps organise information to the point where a judgement can be made, but the risk judgement itself remains mine to own.

This mirrors normal delegation. However good the junior analyst's work is, decision responsibility does not transfer to them.

### Step 4: Draft the Credit Opinion, Then Review

Once the risk view is formed, Copilot is asked to draft a first-version credit opinion based on the risk points identified by the reviewer. The draft should include:

- Each confirmed risk point in detail
- Corresponding mitigants
- Outstanding items
- Cross-references to the source citations

Copilot can produce a structurally useful first draft, but it remains a draft. The credit professional edits the language, adjusts emphasis, adds judgement, and finalises the version that can move forward.

-----

## Portfolio Monitoring Extension

The same pattern also applies to portfolio monitoring.

For existing loans, Copilot is first given the original credit approval documents and the latest monitoring report to build a background brief. This is equivalent to giving a new colleague the approval history and asking them to understand the file before analysing a new request.

Once the background brief is in place, Copilot can support analysis of scenarios such as:

- Consent requests
- Waiver requests
- Covenant-related questions
- Potential conflicts with prior arrangements
- Relevant precedents from earlier monitoring periods

The same control pattern applies: Copilot reads first, the reviewer questions and challenges, and the final judgement remains human-owned.

-----

## Context Window Bottleneck

The main limitation is context window capacity. As document volume grows, a single conversation can no longer reliably hold all relevant information. The model may lose or confuse earlier content, and key extracts need to be pasted again.

This issue appears in both due diligence and monitoring, but monitoring can hit the ceiling faster because the background record accumulates over multiple periods and requests.

The current mitigation under testing is a **project document overview**: a structured summary of key facts, source references, open issues, and prior conclusions that can be carried into a new conversation when the current one approaches its limit.

Open questions remain:

- How detailed does the overview need to be to preserve useful context?
- Which facts need full source references versus brief reminders?
- How should the overview be maintained across repeated monitoring cycles?

-----

## Control Principles

| Principle | Practical Rule |
|---|---|
| Source traceability | Every finding needs document, section or clause, and page reference where available |
| One document at a time | Read and summarise key documents individually before combining the view |
| Challenge before judgement | Use Q&A to test assumptions, conflicts, and missing cross-checks |
| Human ownership | AI organises information; the credit professional owns the risk conclusion |
| Draft, not final | Copilot can prepare a credit opinion draft, but final language and judgement are reviewed and edited |

-----

## Preliminary Conclusions

1. Treating AI as a guided junior analyst is more realistic than treating it as an expert system.
1. Source references are the single most practical control: low cost, high value for traceability and error detection.
1. The multi-round Q&A step cannot be skipped; this is where many real risk issues surface.
1. Context window limits are the main workflow bottleneck for large due diligence packs and accumulated monitoring records.
1. A portable project document overview is the current direction for context carry-over, but the right level of detail still needs testing.

-----

## Roadmap

- [ ] Test project document overview format across longer due diligence reviews
- [ ] Define a minimum source citation standard for Copilot outputs
- [ ] Create reusable prompt templates for due diligence summary, Q&A challenge, and credit opinion drafting
- [ ] Test the same workflow across repeated portfolio monitoring periods

-----

*Deployment Lab | AI-augmented credit risk analysis*
