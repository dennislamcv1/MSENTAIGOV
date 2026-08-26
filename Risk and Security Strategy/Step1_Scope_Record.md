# Step 1 — Scenario Frame and Scope Record

**Initiative:** Contoso Financial Service GenAI Knowledge Assistant (Azure OpenAI)
**Prepared by:** AI Governance Lead
**Purpose:** Frozen reference for the AI Risk & Security Strategy Report. Every downstream section, artifact and control statement must conform to the boundaries below. Referencing an out-of-scope component in a later section is a consistency failure.

---

## 1. Report Audience and Decision Being Supported

**Primary audience:** Contoso Financial Service Executive Risk Committee.

| Member | Role in this decision |
|---|---|
| Chief Risk Officer (CRO) | Chair; owns enterprise risk appetite and the residual-risk acceptance decision |
| Chief Information Security Officer (CISO) | Owns technical control adequacy and inference-endpoint protection |
| Chief Compliance Officer (CCO) | Owns regulatory alignment and data protection exposure |

**Decision requested:** A go / no-go determination on production deployment, scheduled for next month.

**What the committee has asked to see:** the top risks, the organisation's quantified financial exposure, how the inference endpoint is protected, and whether the initiative aligns with the corporate security strategy pillars — in one actionable document.

---

## 2. Initiative Scope

### 2.1 What the system does

An internal, Azure OpenAI–based retrieval-augmented generation (RAG) knowledge assistant. Staff ask natural-language questions; the system retrieves from an indexed corpus of approved internal knowledge and synthesises a grounded, citation-backed answer in real time.

The assistant is **advisory and read-only**. It answers questions. It does not execute transactions, initiate workflows, or make decisions about customers.

### 2.2 Who uses it

Approximately 2,000 internal staff across three functions (see assumption A-09):

| User group | Typical use |
|---|---|
| Customer service staff | Product terms, fee schedules, servicing procedures, complaint-handling steps |
| Operations staff | Back-office processing procedures, exception handling, control steps |
| Compliance staff | Policy lookup, regulatory procedure reference, control-testing guidance |

No customer, contractor, or third-party access.

### 2.3 How it connects to existing infrastructure

| Layer | Integration |
|---|---|
| Access | Internal staff web portal and Microsoft Teams channel |
| Identity | Microsoft Entra ID SSO; token-based authentication, no static keys; group-based entitlements per function |
| Orchestration | API gateway enforcing rate limits and maximum token length per user |
| Guardrails (inbound) | Input validation and prompt-injection filtering before retrieval |
| Retrieval | Permission-trimmed vector index built from approved document repositories |
| Inference | Azure OpenAI Service in Contoso's private Azure tenant, reached over a private endpoint; no Contoso data used for model training |
| Guardrails (outbound) | Content filter with PII and confidentiality scrubbing; citation and groundedness enforcement |
| Audit | Microsoft Purview immutable telemetry — user ID, input prompt, model output hash |

### 2.4 Architecture reference (carried from the scenario brief)

```
[ Staff Prompt ] --> [ Guardrail Layer ] --> [ RAG Vector Index ]
                                                    |
[ Verified Answer ] <-- [ Content Filter ] <-- [ Azure OpenAI Endpoint ]
```

### 2.5 Corporate security strategy pillars

Every control and architectural decision in this report maps to one or more of:

- **Pillar 1 — Confidentiality & Perimeter Security:** preserving data boundary rules and blocking cross-tenant or horizontal leakage paths.
- **Pillar 2 — Data Integrity & Provenance:** validating queries, protecting prompt templates, blocking adversarial manipulation.
- **Pillar 3 — Auditability & Logging Non-Repudiation:** guaranteeing immutable, user-attributed forensic trails.

---

## 3. In-Scope Elements

### 3.1 Components assessed

| # | Component |
|---|---|
| C-1 | Staff-facing chat interface (portal and Teams) |
| C-2 | Entra ID authentication and entitlement layer |
| C-3 | API gateway and orchestration layer |
| C-4 | Inbound guardrail / input validation layer |
| C-5 | Document ingestion and indexing pipeline (classification, hashing, versioning) |
| C-6 | Vector index and document metadata store |
| C-7 | Azure OpenAI inference endpoint (model deployment, system prompt, generation parameters) |
| C-8 | Outbound content filter and citation enforcement |
| C-9 | Purview immutable telemetry and audit log store |
| C-10 | Answer-quality feedback and human review loop |

### 3.2 Data flows assessed

| # | Flow |
|---|---|
| F-1 | Staff prompt → gateway → inbound guardrail → retrieval |
| F-2 | Approved source documents → ingestion pipeline → vector index |
| F-3 | Retrieved context + prompt → Azure OpenAI inference endpoint → completion |
| F-4 | Completion → outbound content filter → staff interface with citations |
| F-5 | All interactions → immutable telemetry store |
| F-6 | Staff feedback on answers → quality review queue → corpus correction |

### 3.3 Data in scope for ingestion

Approved-source allow-list, mirroring the brief's ingestion safe zone:

- Internal policy documents and policy libraries
- Product guides, terms, and fee schedules
- Standard operating procedures, process manuals, and procedural wikis

All ingested content is classified, hashed, and version-tagged before indexing.

### 3.4 Stakeholders

This list is fixed. Risk owners named in the risk register are drawn only from it.

| Stakeholder | Interest / accountability |
|---|---|
| Executive Risk Committee (CRO, CISO, CCO) | Deployment decision; residual risk acceptance |
| AI Governance Lead | Report author; risk register owner; governance framework |
| Head of Information Security | Endpoint controls, threat mitigations, ADR ownership |
| Data Protection Officer | Personal data processing, RoPA, retention |
| Chief Compliance Officer's delegate — Compliance Operations Manager | Regulatory alignment; accuracy of compliance answers |
| Head of Platform Engineering | Azure tenant, deployment, availability |
| Product Owner, Knowledge Assistant | Corpus curation, user experience, feedback loop |
| Customer Service Operations Director | Business use, front-line answer quality |
| Operations Director | Business use, procedural accuracy |
| Internal Audit | Independent assurance over the audit trail |
| End users (CS, operations, compliance staff) | Day-to-day use; misuse and over-reliance exposure |

---

## 4. Out-of-Scope Elements

Explicitly excluded from this assessment. Nothing in Sections 1–7 of the report may rely on, or claim coverage of, any item below.

| # | Exclusion | Basis |
|---|---|---|
| X-1 | Customer-facing or any external deployment | Internal staff only |
| X-2 | Live PII of customers, employees, or candidates in the indexed corpus | Brief, data boundaries |
| X-3 | Corporate financial disclosures, market strategies, unreleased product roadmaps | Brief, data boundaries |
| X-4 | Individual compensation, bonuses, performance reviews, employment contracts | Brief, data boundaries |
| X-5 | Automated workflow execution or transaction initiation by the assistant | Assumption A-04 |
| X-6 | Autonomous decisioning about customers (credit, AML, eligibility, pricing) | Advisory-only design |
| X-7 | Model fine-tuning or training on Contoso data | Tenant configuration |
| X-8 | Security certification of the underlying Azure platform | Inherited from cloud provider assurance |
| X-9 | Unsanctioned public GenAI tool use by staff (shadow AI) | Separate governance workstream |
| X-10 | The HR onboarding assistant use case | Separate initiative (see A-02) |
| X-11 | Third-party risk assessment of Microsoft as a supplier | Existing TPRM programme |

---

## 5. Documented Assumptions

Each assumption below is **not** explicitly stated in the scenario brief. Reviewers should evaluate them directly.

| ID | Assumption | Why it is needed | Confidence |
|---|---|---|---|
| A-01 | The platform is Azure OpenAI Service inside Contoso's private Azure tenant | The brief says only "a foundational LLM hosted within a private corporate cloud tenant." Azure is inferred from Entra ID and Microsoft Purview in the controls inventory, and from the committee's request | High |
| A-02 | The subject is the internal knowledge assistant for CS, operations and compliance — not the HR onboarding assistant the brief is titled for | The brief and the committee's request describe different systems. The brief's architecture, guardrail pattern and data-boundary discipline are carried forward; its HR user population and IT-ticket automation are not | High — confirm with committee |
| A-03 | No RoPA excerpt was supplied. Processing is assumed as: purpose — internal staff knowledge retrieval; personal data — staff identity attributes and prompt-log content only; lawful basis — legitimate interest; retention — per corporate log retention schedule | The report cannot assert data protection compliance without a processing record | **Low — must be confirmed with the DPO before go-live** |
| A-04 | Version 1 is advisory and read-only; no downstream workflow triggering | The committee's request describes retrieval and synthesis only. The brief's IT-ticket automation belongs to the HR use case | High |
| A-05 | The corpus is limited to internal policy documents, product guides and procedures, on an approved-source allow-list | Mirrors the brief's ingestion safe zone, generalised to the operational corpus | High |
| A-06 | The three corporate security strategy pillars are those named in the ADR and Alignment Matrix templates | The brief does not contain a pillars section | High |
| A-07 | The existing AI controls inventory comprises Entra ID token authentication, input validation pipelines, Purview immutable logging, and outbound PII content filters | The brief does not contain a controls inventory; these are the four controls carried in the Alignment Matrix | Medium |
| A-08 | Contoso operates across both US and EU jurisdictions, bringing prudential model-risk expectations and EU data protection law into the regulatory perimeter | Needed to justify jurisdictional-conflict risk in the register | Medium |
| A-09 | The user population is approximately 2,000 staff across the three functions | Needed to size exposure and rate-limiting thresholds | Low — indicative only |
| A-10 | Go-live is targeted within 30 days of committee approval, with a 90-day post-deployment assurance window | Aligns to the committee's stated decision timetable | Medium |

---

## 6. Conflicts Identified in the Source Pack

Recorded so the committee can see they were resolved deliberately rather than overlooked.

| Conflict | Resolution |
|---|---|
| The scenario brief describes an HR onboarding assistant; the committee's request describes a CS/operations/compliance knowledge assistant | Committee request is canonical (A-02). Brief supplies architecture and data-boundary discipline only |
| The pre-filled report template and the risk register template assign the same risk IDs to different risks (prompt injection is R-02 in one, R-03 in the other) | A single authoritative register is built from this scope record. All artifacts reference those IDs. The template's conflicting table is discarded |
| The FAIR calculator quantifies "R-03 — data leak via prompt injection," which does not match the register's R-03 | The FAIR scenario is repointed to the prompt-injection risk's ID in the rebuilt register |
| The STRIDE worksheet is framed around new-hire session tokens and IT provisioning gates | Recast onto the Azure OpenAI inference endpoint and permission-trimmed retrieval, per the committee's request |
| The report template's appendix certifies that no generative AI was used to produce the report | Replaced with an accurate AI-use disclosure recording the tool used, what it drafted, and what was independently reviewed and corrected |

---

## 7. Verified Figures Carried Forward

The FAIR calculator's supplied bounds were recomputed and are arithmetically sound:

| Bound | Loss Event Frequency | Loss Magnitude per event | Annualised Loss Expectancy |
|---|---|---|---|
| Minimum | 0.10 / yr | $500,000 | **$50,000** |
| Most likely | 0.25 / yr | $2,500,000 | **$625,000** |
| Maximum | 0.50 / yr | $12,000,000 | **$6,000,000** |

The pre-filled report quotes only the $50,000–$6,000,000 envelope. The $625,000 most-likely figure is the anchor a risk committee will use and will be stated explicitly.

---

**Status:** Awaiting confirmation. On approval this record is frozen and Step 2 drafting begins against it.
