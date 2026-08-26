# Phase 2 output — GDPR Compliance section, drafted for transfer

Destination: **Section 2 of the Compliance Package Report**. This is finished text, not notes; Phase 5 assembles rather than re-authors. Every figure and identifier is traceable to the three completed tools.

---

## 2. GDPR Compliance

### 2.1 Processing scope and legal basis pairing

The Article 30 register for this deployment records **ten processing activities**. The supplied draft recorded four. Six were added during this review: supporting medical documentation ingest, RAG knowledge base retrieval, the high-risk human review queue, inference telemetry logging, the training corpus export to NovaTriage Analytics Ltd., and the separation of live triage processing from training reuse, which the draft had collapsed into a single record.

**Every one of the ten activities involves Article 9 special category data.** There is no low-sensitivity remainder of this deployment to which lighter controls could reasonably be applied, and the security posture in Sections 3 and 4 should be read against that fact.

| Record | Activity | Art 6 basis | Art 9 flag | Art 9 condition |
|---|---|---|---|---|
| P-01 | Live claim narrative processing (operational triage) | 6(1)(b) contract; 6(1)(f) for third parties named in narratives | **Yes** | 9(2)(g) substantial public interest — insurance |
| P-02 | Supporting medical documentation ingest | 6(1)(b) contract | **Yes** | 9(2)(g); genetic data where lab results include genetic testing |
| P-03 | Narratives and documentation retained as training corpus | 6(1)(f) legitimate interests | **Yes** | 9(2)(g) |
| P-04 | Self-reported demographic monitoring field | 6(1)(a) explicit consent | **Yes** | 9(2)(a) explicit consent |
| P-05 | Automated triage output and clinical-gravity scores | 6(1)(b) for the decision; 6(1)(f) for evaluation retention | **Yes — derived** | 9(2)(g); Art 22(4) additionally engaged |
| P-06 | Historic adjuster notes as training corpus | 6(1)(f) legitimate interests | **Yes** | 9(2)(g) |
| P-07 | RAG knowledge base retrieval at inference | 6(1)(b) own-record grounding; 6(1)(f) corpus maintenance | **Yes** | 9(2)(g) |
| P-08 | High-risk human review queue | 6(1)(b) contract; Art 22(3) safeguard | **Yes** | 9(2)(g) |
| P-09 | Inference and prompt telemetry logging | 6(1)(f) — security, Recital 49 | **Yes — derived** | 9(2)(g) |
| P-10 | Training corpus export to NovaTriage Analytics Ltd. | 6(1)(f) legitimate interests | **Yes** | 9(2)(g) |

#### Three classifications corrected against the supplied draft

**Claim narratives (P-01).** The supplied workbook contradicted itself: Tab 1 recorded Article 9(2)(g), Tab 2 and scenario §2.2 recorded "No — N/A." A system that ingests clinical narratives and medical documentation and determines clinical gravity is processing data concerning health. Resolved to **Yes**. Left unresolved, the contradiction alone would have failed the audit before its substance was reached.

**Derived triage scores (P-05).** The draft recorded "No — derived internal operational sorting index." A classification expressing clinical gravity, computed from health narratives and attached to an identified policyholder, reveals health status; inferred special-category data is special-category data. Reclassified to **Yes**. This is the more contestable of the corrections and is presented as a reasoned position for exactly that reason — an auditor will raise it either way, and a documented conclusion survives the question where silence does not.

**Historic adjuster notes (P-06).** The draft recorded "Conditional — post-redaction review required." Article 9 admits no conditional state: it applies to a category of data or it does not. Redaction is a **minimisation control**, not a basis for de-flagging, and the de-identification evaluation at 2.3 records NER false negatives as a live residual risk, so completeness cannot be assumed. Reclassified to **Yes**, with redaction recorded as the control.

The Article 6 basis for claim narratives was also moved from 6(1)(f) to **6(1)(b)**. Adjudicating a submitted claim is contractual performance; reaching for legitimate interests where contract necessity squarely applies needlessly imports an Article 21 objection right into a process the controller cannot decline to perform.

#### Two structural completeness findings

**Article 30(1) content.** The supplied template omitted three mandatory elements: controller and DPO contact details under 30(1)(a), transfer safeguards documentation under 30(1)(e), and a general description of technical and organisational security measures under 30(1)(g). All three are added. The security-measures omission was the most consequential — nothing in the register connected the defence-in-depth posture assessed in Sections 3 and 4 to the processing it protects. That column now records applied controls **and** known deficiencies at this date, because a register that lists only what works, when six controls are non-compliant or partial, misstates the position it exists to record.

**Article 9 field inventory.** Naming "health data" as a category is not an Article 30 record. The register is supported by an inventory of twenty-five named data elements — presenting condition, symptom onset and severity, injury mechanism and body site, ICD-coded and free-text diagnosis, procedures, medication names and dosages, admission and treatment dates, practitioner and facility identifiers, prognosis, functional limitation, and the derived clinical-gravity outputs.

That inventory surfaced a category of exposure the draft did not record. Claimants write freely, and narratives foreseeably contain Article 9 categories the deployment never solicits: **mental health, pregnancy and maternity, substance use, sexual orientation arising in the course of describing a related condition, religious belief where a treatment was declined on that basis, and trade union membership in occupational injury claims.** No control currently targets these — the NER pipeline is tuned for direct identifiers, not for suppressing unsolicited special-category disclosure.

The inventory also identified a **category of data subject absent from the draft entirely**: dependants, beneficiaries, treating clinicians, employers and witnesses named within narratives. Their health data is processed with no consent, no notice and no realistic Article 14 route. Article 14(5)(b) disproportionate-effort relief must be documented rather than assumed — addressed at DPIA GAP-07.

---

### 2.2 DPIA gap analysis and remediation plan

The five-step gap-detection methodology was applied: enumerate Section 3 risks, map Section 4 mitigations to them, flag risks with no mitigation as True Gaps, test existing mitigations for a named control, named owner, effectiveness criterion and review cadence, and complete an entry for each.

**A sixth step was added, and it produced three of the seven gaps.** Steps 1 to 5 compare two lists and can only find gaps *between* them. A risk the DPIA author never identified in Section 3 has no Section 4 entry either, and the methodology as written is structurally blind to it. Section 3 was therefore re-tested for completeness against the deployment as described in the scenario package.

**Seven gaps across six mitigation categories: five True Gaps, two Present-but-Weak Mitigations.**

| Ref | Gap | Type | Category | Owner |
|---|---|---|---|---|
| GAP-01 | Intermediate transcript cache retention | Present-but-Weak | Data Minimisation | Data Engineering Lead |
| GAP-02 | Article 9 training corpus crossing the trust boundary | True Gap | Technical Safeguards / Purpose Limitation | Principal AI Architect |
| GAP-03 | Access control over high-risk flagged claim narratives | True Gap | Technical Safeguards | Chief Information Security Officer |
| GAP-04 | Purpose limitation between operational triage and retraining | Present-but-Weak | Purpose Limitation Enforcement | Lead AI Infrastructure Engineer |
| GAP-05 | Meaningful human review asserted but unevidenced | **True Gap** | Rights and Freedoms Safeguards | AI Governance Lead |
| GAP-06 | No Chapter V determination for cross-border transfers | True Gap | Transfer Governance | Lead Data Privacy Officer |
| GAP-07 | No objection route or third-party notice for secondary use | True Gap | Transparency and Data Subject Rights | Lead Data Privacy Officer |

#### GAP-05 is the most consequential entry in the plan

Every other gap concerns the security or governance of data within a lawful processing operation. GAP-05 concerns whether the operation is lawful at all.

The deployment's Article 22 position rests on the proposition that model outputs *influence but do not solely determine* claim outcomes, because a human reviews high-risk cases. Nothing in the package evidences that proposition. There is no measured review rate, no override rate, and no confirmation that reviewers hold the authority and competence to depart from the model. Set against that:

- the system is the **sole** pre-authorization screening layer, with no parallel fallback;
- it **dispatches automated transactional instructions**, not recommendations awaiting human execution;
- red-team testing bypassed routing to human review in **14.2%** of adversarial variations.

If human involvement is not meaningful, Article 22(1) applies to processing of special category data. Article 22(4) then restricts that processing to 9(2)(a) or 9(2)(g) grounds with suitable safeguards, and Article 22(3) requires a demonstrable route for the data subject to obtain human intervention, express their point of view and contest the decision. None of this is currently documented.

The control introduced instruments the review queue for three measures — review rate, override rate, median time-on-record — publishes a reviewer authority matrix confirming that adjudicators may overturn any model output without escalation and are not penalised for doing so, and establishes an **override floor** below which the review step is presumed nominal and the Article 22 position is re-opened rather than assumed to hold.

**Residual risk:** until the first measurement is taken, the deployment operates with an unverified Article 22 position, and the measurement may show that the position does not hold. Instrumentation alone cannot close this; it can only make the answer visible. Automation bias is separately untouched — a reviewer who defers to the model because it is the model produces a healthy departure rate only where the model errs in ways a reviewer would notice, and concordance is not evidence of independent judgement.

#### A technical correction to an inherited control

The scenario package's mitigation for cross-boundary export specified *differential privacy transformations applied to the historical claim corpora prior to export*. That control cannot work as described. DP-SGD perturbs gradients during backpropagation — it is a property of a training run, not a transformation applicable to a dataset and then shipped. A corpus described as "processed with DP-SGD" before export is an unmodified corpus, every narrative in it as readable on the vendor's infrastructure as on ours.

Under the original wording, the largest privacy exposure in the deployment would have been recorded as mitigated while remaining wholly unprotected — a worse position than an open gap, because an open gap is visible. The controls are re-pointed at GAP-02: **masking protects the export, DP-SGD protects the model.**

#### Two residual risks that no listed control closes

Carried into the Risk Disposition rather than presented as resolved:

- the exported corpus **remains personal data** in the vendor's hands whatever masking is applied (GAP-02);
- the **Article 22 position remains unverified** until the first review-rate measurement is taken (GAP-05).

---

### 2.3 De-identification recommendation and residual risk memorandum

**Recommendation: hybrid — clinical NLP-NER token masking applied to the corpus before export, combined with Differential Privacy (DP-SGD) applied to the model training run.**

#### Why two controls

The deployment presents two distinct exposures and no single technique addresses both. **Corpus confidentiality**: a dataset of Article 9 clinical narratives leaves Meridian's trust boundary and sits on a third party's infrastructure — what protects it must operate on the exported artefact. **Model memorisation**: an LLM trained on clinical narratives can reproduce training text and is vulnerable to membership inference, which for a health corpus discloses the condition by implication — what protects against that must operate on the training process. Pairing the two is not redundancy; they defend different objects.

#### Why this fits this deployment specifically

- **The corpus is unstructured clinical prose, not tabular records.** This removes K-anonymity from consideration entirely — generalisation and suppression are defined over structured quasi-identifier columns and there is nothing here to generalise. It also bounds what masking achieves, since identifying information is distributed through the prose rather than isolated in fields.
- **The model must retain clinical semantics to function.** ClaimsTriage classifies clinical gravity. Every technique that flattens clinical detail degrades the thing the system exists to do, and the accuracy floor is contractual: 91.0% TPR against a current 94.2%. There is **3.2 points of headroom**, and de-identification spends some of it. That is the real constraint on how aggressive masking can be.
- **The corpus crosses a trust boundary.** This is what makes an export-surviving control mandatory. Were training performed in-house, DP-SGD with strong access control would be defensible. It is not defensible once the data sits where Meridian cannot observe it.

Token-based pseudonymisation was rejected as a primary control: the mapping key exists and narrative context remains structurally intact, so it fails the EDPB linkability test on its own terms. K-anonymity was rejected as designed for tabular microdata, and because its outlier suppression would systematically remove the rare presentations that matter most clinically.

#### What this recommendation does not claim

**The output is not anonymous data.** NER-masked narratives remain personal data under Article 4(5) and Recital 26: means reasonably likely to be used for re-identification still exist, because the quasi-identifiers are in the prose and cannot be removed without destroying utility. The corpus is pseudonymised to a high standard, and the full weight of the GDPR continues to apply to it.

This matters practically. Because the corpus is not anonymised, the export remains a transfer of personal data, and the Article 28 processing agreement and the Chapter V determination at GAP-06 carry the protection that de-identification cannot. **Treating the masked corpus as anonymous would remove exactly those safeguards at the moment they are most needed.**

Nor does DP-SGD render the model anonymous. It bounds membership inference advantage by a stated epsilon; it does not reduce it to zero, and a weak epsilon bounds very little. The guarantee is only as strong as the budget, which is why the budget must be documented and accounted cumulatively rather than described as "differential privacy applied."

> This paragraph replaces the assertion in the supplied report template that differential privacy "satisfies the EDPB anonymisation tests for singling out, linkability and inference, providing a legally defensible stance for exporting data." That claim is not supportable and would not survive challenge.

#### Residual risks stated for acceptance

| Ref | Residual risk | Rating | Accepting owner | Review |
|---|---|---|---|---|
| R-DEID-01 | Semantic re-identification via narrative quasi-identifiers | Medium | Lead Data Privacy Officer | Each corpus freeze |
| R-DEID-02 | Utility and fairness degradation on long-tail presentations | Medium | Principal AI Architect | Each model release |
| R-DEID-03 | NER false negatives leaving identifiers unmasked | Medium | Principal AI Architect | Each corpus freeze |
| R-DEID-04 | Corpus remains personal data in the vendor's hands | Medium | Lead Data Privacy Officer | Semi-annually |
| R-DEID-05 | Epsilon budget exhaustion across successive retrains | Low–Medium | Principal AI Architect | Each training run |

Two deserve emphasis in the risk disposition. **R-DEID-02** is a fairness exposure, not only a utility one: DP noise disproportionately degrades accuracy on classes with few training examples, which in a clinical corpus means rare conditions and atypical claim profiles — so the claimants with the least common presentations receive the least reliable triage, in a system that governs access to settlement. **R-DEID-05** fails silently: the CVE-2025-XXXX remediation requires an unscheduled full retrain that consumes privacy budget outside the planned schedule, and without cumulative accounting successive retrains erode the guarantee until DP-SGD is present in the configuration but meaningless in effect.

All five are carried as discrete tracked line items in Section 5. A residual risk stated in the de-identification evaluation and absent from the risk disposition is an orphan finding — the specific cross-referencing failure the evaluation's own audit note warns causes audit failure.

---

### 2.4 Open determinations affecting the GDPR position

Five determinations bear on this section and are recorded as unresolved rather than assumed. An assumed answer that proves wrong is worse than a declared gap.

| Ref | Question | What turns on it |
|---|---|---|
| **OD-01** | Main establishment and lead supervisory authority | Article 9(2)(g) is not self-executing. It requires a named Member State legal provision and an appropriate policy document — the condition relied on for **eight of ten records**. Also determines Article 56 routing and which authority would receive an Article 36 consultation. |
| **OD-02** | Country of establishment of NovaTriage Analytics Ltd. | Decides whether Chapter V applies at all. "Ltd." is used in both the UK and Ireland and no country is recorded. Ireland: intra-EU, Article 28 agreement sufficient. UK: third country, adequacy or Article 46(2)(c) SCCs plus a transfer impact assessment. |
| **OD-03** | Measured review rate, override rate, reviewer authority | **The determination that can change the audit verdict.** See GAP-05. |
| **OD-04** | Azure region and Microsoft sub-processor chain | Telemetry derived from Article 9 narratives, including embedding vectors, flows through a chain never enumerated. A transfer route easy to overlook, because monitoring is not usually thought of as a transfer. |
| **OD-07** | Is self-reported ethnicity a model input feature or a monitoring dimension? | The draft records both purposes. If a training feature in a system that prioritises claims, the deployment carries direct discrimination risk under Article 5(1)(a) and engages Article 22(4) on a special-category input. If used only to test outputs for disparate impact, it is a fairness safeguard and a point in the deployment's favour. The two cannot both stand. |

**The RoPA transfers column has been changed from "none identified at this stage" to UNDETERMINED across nine records, with the tenth (P-10) recorded as a known transfer with an outstanding mechanism.** The original entry records that no assessment was performed. It is not a finding, and presenting it to an auditor as one is the kind of unforced error that converts a remediation order into a pause directive.

---

## Traceability

| Report element | Source tool | Artifact |
|---|---|---|
| 2.1 legal basis matrix, Art 9 corrections, Art 30 completeness | RoPA | `RoPA_Completed_MHP.xlsx` — sheets 1, 2, 3, 6 |
| 2.2 gap analysis and remediation | DPIA Mitigation Plan | `DPIA_Mitigation_Plan_Completed_MHP.docx` — Parts A, B |
| 2.3 recommendation and residual risks | De-Identification Matrix | `DeIdentification_Comparison_Matrix_Completed_MHP.docx` — Sections 1, 2, 3 |
| 2.4 open determinations | All three | RoPA sheet 4; DPIA Part D; Phase 1 frame §8 |

## Feeds forward

- **R-DEID-01 to R-DEID-05** → Section 5 risk disposition, five discrete line items
- **GAP-01 to GAP-07** → Section 5, with GAP-05 as the highest-consequence entry
- **C3.3 (DP-SGD absent)** → Section 3 model security, as the technical precondition of the 2.3 recommendation
- **GAP-02 export control** → Section 4 operational security, vendor boundary
- **OD-03** → Section 5 risk disposition and the Section 1 verdict
