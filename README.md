# AI-Assisted Procurement Review

> **Core outcome: a traffic-light review report (🟢 no discrepancy flagged / 🔴 review required) is designed to help senior approvers make informed decisions with less administrative cross-checking—while final judgment remains human.**
>
> An anonymized case study of a user-triggered Python workflow that uses an LLM to compare PO records with supporting documents and presents 🟢 / 🔴 findings with cited evidence. **Senior approvers** are leaders with approval authority.

## Contents

- **Business case — why it matters**
    - [Executive Summary](#executive-summary)
    - [Business Process: As-Is to To-Be](#business-process-as-is-to-to-be)
    - [Business Value and Evidence](#business-value-and-evidence)
        - [Measures for a future improvement phase](#measures-for-a-future-improvement-phase)
- **Solution design — how it works**
    - [Exception Feedback: Governed Prompt Updates](#exception-feedback-governed-prompt-updates)
        - [Synthetic example of the rule register](#synthetic-example-of-the-rule-register)
    - [LLM Harness and Review Output](#llm-harness-and-review-output)
        - [Synthetic example of the generated report](#synthetic-example-of-the-generated-report)
        - [Summary view](#summary-view)
        - [Detailed comparison view](#detailed-comparison-view)
- **Delivery and trust — who did what, and what to know**
    - [My Role and Technology](#my-role-and-technology)
    - [Scope, Limitations, and Privacy](#scope-limitations-and-privacy)

## Executive Summary

### Business context and challenge

- **Approval context:** Enterprise POs typically pass through internal checkpoints before purchase release. Senior decision-makers need confidence that key commercial details are supported by the PO record and its evidence.
- **Review challenge:** Much of the review involves administrative, field-by-field reconciliation—checking quantities, prices, totals, and terms across the PO and its supporting documents. The challenge is that this transactional checking draws attention away from the approver's higher-level judgment: assessing exceptions and making the business approval decision.

### Solution and intended value

- **Solution:** A user-triggered Python workflow automates evidence collection and first-pass matching through three category-specific LLM steps. It consolidates 🟢 / 🔴 statuses, exceptions, and source references in a review report.
- **Intended value:** The report shifts the approver's starting point from raw information to an organized assessment, aiming to reduce administrative burden and focus attention on exceptions.

### Adoption and my contribution

- **Observed adoption:** The workflow ran **290 times in its first six months**, typically covering **1–4 POs per run**. The user described the report as helpful.
- **My contribution:** I implemented and delivered the initial manager-defined concept. After launch, I independently identified, designed, and implemented a governed feedback-to-prompt mechanism for recurring exceptions, with project-PM review.

## Business Process: As-Is to To-Be

### As-Is — Manual comparison

```mermaid
flowchart LR
    S((Start)) --> A[Pending PO]
    A --> B[Open PO record]
    B --> C[Find and open supporting documents]
    C --> D[Manually compare fields and terms]
    D --> E[Approver interprets findings]
    E --> F[Make approval decision]
    F --> Z((End))
```

### To-Be — Automated preparation, human decision

```mermaid
%%{init: {"flowchart": {"nodeSpacing": 18, "rankSpacing": 22, "diagramPadding": 4}, "themeVariables": {"fontSize": "12px"}}}%%
flowchart TD
    START([Start]) --> BEGIN[User clicks Run]
    BEGIN --> COLLECT[Collect pending POs and documents]

    subgraph EXTRACTION["1. Document extraction"]
        direction TB
        EXT_RULES[Select active extraction rules] --> EXT_PROMPT[Add rules to extraction prompt]
        EXT_PROMPT --> EXT_LLM[LLM extracts document data]
        EXT_LLM --> EXT_VALIDATE[Validate extraction]
    end
    COLLECT --> EXT_RULES

    subgraph DOC_AUDIT["2. PO-level review"]
        direction TB
        DOC_RULES[Select active PO-level rules] --> DOC_PROMPT[Add rules to PO-level prompt]
        DOC_PROMPT --> DOC_LLM[LLM reviews PO terms]
        DOC_LLM --> DOC_VALIDATE[Validate results and select evidence]
    end
    EXT_VALIDATE --> DOC_RULES

    subgraph ITEM_AUDIT["3. Line-item review"]
        direction TB
        ITEM_RULES[Select active line-item rules] --> ITEM_PROMPT[Add rules to line-item prompt]
        ITEM_PROMPT --> ITEM_LLM[LLM compares PO line items]
        ITEM_LLM --> ITEM_VALIDATE[Validate results]
    end
    DOC_VALIDATE --> ITEM_RULES

    ITEM_VALIDATE --> REPORT[Build summary and detailed report]
    REPORT --> REVIEW[Approver reviews findings and evidence]
    REVIEW --> DECISION[Human makes final decision]
    DECISION --> FINISH([End])

    FEEDBACK[Human-maintained feedback rule register] -.-> EXT_RULES
    FEEDBACK -.-> DOC_RULES
    FEEDBACK -.-> ITEM_RULES

    classDef human fill:#FFF7E8,stroke:#D97706,stroke-width:2.5px,color:#222222
    classDef bot fill:#EFF6FF,stroke:#2563EB,stroke-width:2.5px,color:#222222
    classDef boundary fill:#F3F4F6,stroke:#6B7280,stroke-width:2px,color:#222222
    class BEGIN,REVIEW,DECISION,FEEDBACK human
    class COLLECT,EXT_RULES,EXT_PROMPT,EXT_LLM,EXT_VALIDATE,DOC_RULES,DOC_PROMPT,DOC_LLM,DOC_VALIDATE,ITEM_RULES,ITEM_PROMPT,ITEM_LLM,ITEM_VALIDATE,REPORT bot
    class START,FINISH boundary
```

**Legend:** Orange = human action/input; blue = automated step; gray = start/end.

| Stage | What happens | Result passed forward |
|---|---|---|
| Document extraction | Apply active extraction rules; extract information from supported PDF/image attachments. | Structured document summary and line items. |
| PO-level review | Apply matching active rules; review overall terms such as payment terms and warranty, and identify relevant evidence. | PO-level checks and selected evidence context. |
| Line-item review | Apply matching active rules; compare PO line items with the selected evidence. | Item-level checks and comparison details. |

## Business Value and Evidence

The value proposition is **decision-maker leverage**: evidence gathering and first-pass comparison are prepared before the report reaches the approver, who can start with 🟢 / 🔴 statuses, potential exceptions, and cited evidence rather than assembling comparisons from raw records.

**Evidence and limits:** Usage and positive user feedback indicate adoption and perceived usefulness—not time saved, improved accuracy, fewer control incidents, or financial return. No before/after measurement was conducted.

### Measures for a future improvement phase

To quantify value in a future phase, first agree KPI definitions and data ownership with the process owner, establish a baseline, and then track:

| Candidate measure | Example definition |
|---|---|
| Review cycle time | Time from run start to report availability; separately, report availability to human decision when timestamps exist. |
| Adoption and throughput | Run frequency and number of POs reviewed per run. |
| Finding quality | Share of AI-flagged checks confirmed, corrected, or dismissed by the human reviewer. |
| Evidence coverage | Share of reviewed POs with required checks and traceable source evidence. |

These are proposed measures, not current metrics or achieved results.

## Exception Feedback: Governed Prompt Updates

After launch, recurring exceptions made repeated prompt edits cumbersome. I designed a separate rule register: at workflow setup, the program loads **Active** rules and routes each by category to document extraction, PO-level review, or line-item review; **Draft** rules are excluded. The mechanism is human-curated, not autonomous learning.

### Synthetic example of the rule register

The examples below are fictional and illustrate the management design only; they are not copied from real feedback or business rules.

| Category | User feedback *(synthetic)* | Curated prompt rule *(synthetic)* | Status |
|---|---|---|---|
| Document extraction | “The scan is faint. Please do not guess unreadable values.” | Mark values that cannot be read confidently as unavailable and surface them for human review. | 🟢 Active |
| PO-level review | “A quote may list an optional service separately from the goods.” | Do not include a clearly optional fee in a required-total comparison unless the PO includes it. | 🟢 Active |
| Line-item review | “The short product name differs, but the model identifier matches.” | Use an exact model identifier as matching evidence when available; flag conflicting identifiers. | Draft |

I considered vector retrieval, but the observed rule volume was small enough to include relevant active guidance directly in the prompts. This workflow does not use embeddings or vector search.

## LLM Harness and Review Output

The harness constrains response shape and normalizes fields for downstream reporting. It cannot establish factual correctness, so source evidence and human review remain necessary.

- **Pydantic** models define required fields, nested audit checkpoints, and permitted status values.
- **Instructor** requests responses using those models.
- Validators normalize selected values, while application code assembles report rows from named checkpoints.

Illustrative example only; this is not production code:

```python
from typing import Literal

from pydantic import BaseModel, Field


class Checkpoint(BaseModel):
    name: str
    system_value: str
    evidence_value: str
    status: Literal["GREEN", "RED"]
    explanation: str


class AuditReport(BaseModel):
    overall_status: Literal["GREEN", "RED"]
    checkpoints: list[Checkpoint] = Field(min_length=1)


report = instructor_client.chat.completions.create(
    model="configured-model",
    response_model=AuditReport,
    messages=[{"role": "user", "content": "Review the supplied synthetic evidence."}],
)
```

### Synthetic example of the generated report

All identifiers, values, comments, and filenames below are invented for illustration. The review result is binary: 🟢 no discrepancy flagged or 🔴 review finding. 🟢 summary cells are intentionally blank; 🔴 summaries show the compared values directly.

#### Summary view

| PO record *(synthetic)* | PO-level status | PO-level summary | Line-item status | Line-item summary | Source evidence *(synthetic)* |
|---|---|---|---|---|---|
| PO-DEMO-001 | 🟢 |  | 🔴 | Item 2 tax rate: PO 8% vs. attachment 10%. | demo-quote-revB.pdf |
| PO-DEMO-002 | 🔴 | Payment terms: PO Net 30 vs. attachment Net 45. | 🟢 |  | demo-quote-v2.pdf |

#### Detailed comparison view

| PO *(synthetic)* | Review level | Checkpoint | PO value | Document value | Result | Reviewer note | Source evidence *(synthetic)* |
|---|---|---|---|---|---|---|---|
| PO-DEMO-001 | Line item | Tax rate — Item 2 | 8% | 10% | 🔴 Difference | Confirm the applicable treatment against the source document. | demo-quote-revB.pdf |
| PO-DEMO-001 | Line item | Item description — Item 1 | Standard adapter kit | Adapter kit, standard | 🟢 Match | Description is consistent; wording order differs. | demo-quote-revB.pdf |
| PO-DEMO-002 | PO-level | Payment terms | Net 30 | Net 45 | 🔴 Difference | Review the terms and any later supporting evidence. | demo-quote-v2.pdf |
| PO-DEMO-002 | PO-level | Warranty period | 12 months | 12 months | 🟢 Match | Values agree in the cited document. | demo-quote-v2.pdf |

Reviewers scan the summary, inspect 🔴 findings and cited sources, then make the decision. 🟢 is not approval, and 🔴 findings may need context because the model can be overly strict.

## My Role and Technology

### My role

- **Initial delivery:** The business requirements and high-level concept had already been established by my manager. I translated that direction into a working implementation and supported testing, acceptance, and deployment.
- **Post-launch improvement:** I identified the recurring exception-management problem, proposed the categorized and status-controlled prompt mechanism to the project PM, and independently completed its design, implementation, and deployment.

### Technology used

Python · Playwright · Google Gemini (`google-genai`) · Instructor · Pydantic · Google Sheets

## Scope, Limitations, and Privacy

- This is AI-assisted PO review, not autonomous approval or end-to-end process mining.
- No cycle-time savings, accuracy improvement, or ROI has been measured.
- This portfolio case is anonymized. Company names, internal system identifiers, credentials, real prompts, transaction data, and proprietary rules are omitted. All examples are synthetic, and the code snippet is newly written for illustration.
