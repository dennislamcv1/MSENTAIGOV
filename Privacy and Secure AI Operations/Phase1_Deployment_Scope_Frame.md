# Phase 1 — Deployment Scope Frame
**Insurance Claims Triage AI (v2.4) — Privacy & Secure Operations Compliance Package**

| | |
|---|---|
| Controller | Meridian Health Partners Ltd. |
| Document reference | MHP-AI-GOV-2026-PKG01-SCOPE |
| Package effective date | July 3, 2026 |
| Prepared by | AI Governance Lead |
| Classification | Internal Restricted — Confidential Regulatory Audit Material |
| Source of authority | Insurance Claims Triage AI Scenario Package (MHP-AI-GOV-2026-PKG01), read in full prior to opening any template |

---

## 0. Standing of this document

This is the **control document** for the compliance package. Every subsequent section — GDPR Compliance, Model Security, Operational Security, Risk Disposition — describes the system as defined here, using the identifiers fixed here.

The rule for the rest of the engagement: **before drafting any section, verify the framing against §7 (Integration Contract). Before submitting, verify every section against §7 again.** Scope drift is not caught by reading a section on its own; it is caught by comparing sections to a fixed reference. This is that reference.

Where the scenario package is silent, this document says so explicitly rather than filling the gap silently. Assertions are graded in §8 as **stated**, **derived**, **determined**, or **unresolved**. An auditor is entitled to know which is which, and a package that quietly presents determinations as facts fails on exactly that.

---

## 1. System identity

| Attribute | Value |
|---|---|
| System name | Insurance Claims Triage AI ("ClaimsTriage") |
| Version under assessment | **v2.4** |
| System type | LLM-based, transformer architecture, retrieval-augmented (RAG) |
| Base library at training | NeuralCore v3.3.2 |
| In production since | Approximately January 2026 (six months prior to package date) |
| Prior version | v2.3, compiled on NeuralCore v2.9.1 — retained as rollback contingency only |
| Operational criticality | Sole primary screening layer; no parallel fallback |
| Business lines served | High-volume health insurance claims |

**v2.4 is the system under assessment.** v2.3 enters the package only as a rollback target in Operational Security. No section may describe v2.3 as the operating system, and no section may attribute v2.3's performance characteristics to the deployment.

---

## 2. What the system does

End-to-end processing chain, as described in the scenario package:

1. **Intake** — claims arrive through a public-facing web claim interface. The system ingests unstructured free-text clinical narratives, claimant statements, historical medical files, and multi-format PDF narrative documents.
2. **Tokenization** — Unicode normalization, string cleaning, vocabulary lookup, integer token sequence generation.
3. **Embedding** — token-to-vector mapping in high-dimensional space, positional encoding.
4. **Attention with RAG retrieval** — context window assembles the untrusted claim text alongside authoritative retrieved material: historical clinical records, corporate insurance policy guidelines, and medical adjudication underwriting reference databases.
5. **Classification** — the system determines **clinical gravity** and assigns a priority score.
6. **Output decoding and dispatch** — generated output is serialized and dispatched as **automated transactional instructions** to downstream Claims Management Systems: the Automated Claims Adjudication Queue, the High-Risk Human Review Queue, and Medical Underwriting Systems.

Two characteristics of this chain drive most of the package and must not be softened in any section:

- **It is the sole pre-authorization screening layer.** No secondary legacy fallback runs in parallel. Every claim passes through it, and there is no independent path by which a misrouted claim is caught.
- **It dispatches automated transactional instructions**, not recommendations. Routing is executed by the system, not proposed to a human for execution.

---

## 3. Operational baselines

| Metric | Value | Bears on |
|---|---|---|
| Daily ingestion volume | ~4,200,000 claims/day | Availability risk, Art 35 "large scale", backlog modelling |
| Peak window | 11:00–15:00 local, ~700,000 claims over 4 hours (~48.6/sec) | Rate-limit control adequacy (C1.1) |
| Average claim value | $87 | Impact modelling |
| Baseline leakage/fraud rate | $0.42 per $1,000 (0.042%) | Deviation detection baseline |
| Derived daily transaction value | ~$365.4M/day | Impact modelling |
| Derived baseline leakage | ~$153,468/day | Impact modelling |
| v2.4 True Positive Rate | 94.2% | Current performance |
| Contractual SLA floor | 91.0% | Breach threshold |
| v2.3 True Positive Rate | 90.4% | Rollback residual exposure |
| Rollback trigger | Rolling 24h TPR below 89.5%, measured in a 1h window | Rollback Plan |
| Manual backlog if rolled back | +159,600 claims/day (4.2M × 3.8%) | Rollback rejection rationale |

Only these four performance figures exist in this package: **94.2 / 91.0 / 90.4 / 89.5**. The 92.5% threshold appearing in the supplied Rollback Plan and Patch-vs-Retrain matrix is discarded — it originates with a different vulnerability and would cause the rollback target to breach its own trigger on arrival.

---

## 4. Data processed

### 4.1 Registered processing activities

| Ref | Activity | Personal data | Special category |
|---|---|---|---|
| P-01 | Claim Narrative Free-Text Body | Unstructured narratives, accident summaries, personal statements | **Yes — health, Art 9(1)** |
| P-02 | Self-Reported Demographic Field | Voluntary structured demographic monitoring | **Yes — racial/ethnic origin, Art 9(1)** |
| P-03 | Automated Triage Output / Scores | AI-generated priority scores linked to policyholder records | **Yes — health (derived), Art 9(1)** — see §4.3 |
| P-04 | Historic Adjuster Notes | Free-text adjuster observations from prior evaluation cycles | **Yes — health, Art 9(1)** — see §4.3 |

### 4.2 Processing activities not currently registered

Identified during the read-through; each is live processing of personal data and each requires a RoPA entry before audit:

| Ref | Activity | Why it is processing |
|---|---|---|
| P-05 | Supporting medical documentation ingest | Stage 1 explicitly ingests multi-format PDF medical documents — Art 9 health data, registered nowhere |
| P-06 | RAG knowledge base retrieval | Historical clinical records queried at inference time against live claims |
| P-07 | High-Risk Human Review Queue | Art 9 narratives surfaced to named reviewers; the Art 22 safeguard itself |
| P-08 | Inference and prompt telemetry logging | Azure Monitor / Defender ingest of prompt lengths, embedding vectors, logit entropy |
| P-09 | Training corpus export to NovaTriage | The cross-border transfer, as a distinct activity from the training purpose |

### 4.3 Two Article 9 classifications corrected at the frame

Both were carried as "No" or "Conditional" in the source pack. Fixing them here, once, prevents four sections inheriting the error.

**P-03, derived triage scores.** A score expressing *clinical gravity*, computed from health narratives and attached to an identified policyholder, reveals health status. Data revealing a special category is special-category data regardless of whether it was collected or inferred. Classified **Article 9**. This is the more contestable of the two, and it is deliberately taken as a documented position rather than left as a bare "No" — an auditor will ask, and "we assessed it and concluded X because Y" survives that question in a way that silence does not.

**P-04, historic adjuster notes.** Carried as "Conditional — post-redaction review required." Article 9 admits no conditional state: it applies to a category of data or it does not. Redaction is a **minimisation control**, not a basis for de-flagging — and our own De-Identification Matrix records NER false negatives as a live residual risk, so the redaction cannot be assumed complete. Classified **Article 9**, with redaction recorded as the control that reduces the risk rather than the fact.

### 4.4 Article 9 conditions relied on

| Activity | Art 6 basis | Art 9 condition |
|---|---|---|
| P-01, P-03, P-04, P-05, P-06 | 6(1)(f) Legitimate Interests | **9(2)(g)** — substantial public interest (insurance purposes), requiring an identified Member State law basis and an appropriate policy document |
| P-02 | 6(1)(a) Explicit consent | **9(2)(a)** — explicit consent |

Note for Phase 2: reliance on 9(2)(g) is not self-executing. It requires a named Member State legal provision, proportionality, and safeguards — none of which the current pack identifies. That is a Phase 2 work item, not an assumption.

---

## 5. Data subjects

**In scope:**

- Policyholders submitting claims against active coverage
- Policyholders who voluntarily completed demographic monitoring
- Policyholders whose claims were processed through AI screening
- Policyholders assessed by adjusters in prior evaluation cycles
- **Third parties named incidentally within claim narratives** — dependants, beneficiaries, treating clinicians, witnesses

That last category is not in the supplied RoPA and needs to be. Free-text clinical narratives routinely name people other than the claimant, and their health data is processed with no consent, no notice, and no realistic Art 14 route. This is a genuine transparency exposure, not a technicality.

**Out of scope:** Meridian employees and adjusters in their capacity as data subjects. Adjuster notes are authored by employees but the data subjects of the *content* are policyholders. Employee monitoring is a separate processing operation and is not assessed here.

---

## 6. Jurisdiction, roles, and regulatory audience

### 6.1 Roles

| Party | Role | Basis |
|---|---|---|
| Meridian Health Partners Ltd. | **Controller** — sole, for all activities P-01 to P-09 | Determines purposes and means |
| NovaTriage Analytics Ltd. | **Processor** (Art 28) — AI model development vendor | Processes on documented instruction |
| Microsoft (Azure / Defender for Cloud) | **Processor** — ICT infrastructure and telemetry | Inferred from stated tooling |

No joint controllership is asserted. If NovaTriage determines any element of training purpose or method independently, Art 26 is engaged and this determination changes — a question the Art 28 DPA review in Phase 2 must answer rather than assume.

### 6.2 Jurisdictional posture

Meridian is a multinational health insurer operating **across the EU and additional jurisdictions**. GDPR applies in full, both by establishment (Art 3(1)) and to the monitoring and processing of EU data subjects' data wherever performed (Art 3(2)).

The scenario package names **no specific country for any party**. Four determinations are therefore open and are recorded as such rather than assumed:

| Ref | Open determination | Why it matters |
|---|---|---|
| **OD-01** | Main establishment and lead supervisory authority | Decides Art 56 one-stop-shop and who receives the Art 36 consultation if one is required |
| **OD-02** | NovaTriage Analytics Ltd. country of incorporation | "Ltd." is used in both the UK and Ireland. Ireland → intra-EU, no Chapter V. UK → third country, adequacy reliance plus monitoring. This single unknown decides whether Chapter V is engaged at all |
| **OD-04** | Azure region and Microsoft sub-processor chain for telemetry | Defender for Cloud and Azure Monitor ingest prompt and vector telemetry derived from Art 9 data |
| **OD-05** | The identity of the "additional jurisdictions" | Determines which non-EU regimes apply alongside GDPR |

Until OD-02 and OD-04 are answered, the RoPA's "no transfers identified at this stage" is **not a finding** — it is a statement that we have not looked. That distinction is the difference between a defensible register and an indefensible one.

### 6.3 GDPR obligations engaged

| Article | Engaged because | Lands in |
|---|---|---|
| 5(1)(a)–(f), 5(2) | Fairness, purpose limitation, minimisation, accuracy, storage limitation, integrity; accountability | GDPR §2 |
| 6(1)(a), 6(1)(f) | Consent and legitimate interests both relied on | GDPR §2 |
| 9(1), 9(2)(a), 9(2)(g) | Health data and racial/ethnic origin at scale | GDPR §2 |
| 12–14 | Transparency, including **Art 14** for third parties named in narratives | GDPR §2 |
| 15(1)(h) | Right to meaningful information about the logic of automated decisions | GDPR §2 |
| 17, 18, 21 | Erasure, restriction, and **objection to legitimate-interest processing** — directly relevant to training reuse | GDPR §2 |
| **22(1)–(4)** | ADM with significant effects on special-category data; **Art 22(4)** restricts this to 9(2)(a) or 9(2)(g) grounds with safeguards | GDPR §2 + Risk Disposition |
| 24, 25 | Accountability; data protection by design and default | All sections |
| 28 | NovaTriage processor obligations and instruction boundaries | GDPR §2 |
| 30(1)(a), (d), (e), (g) | Record completeness — three elements currently absent, including the **security measures description** | GDPR §2 |
| 32 | Security of processing — the defence-in-depth posture is the Art 32 evidence | Model + Operational Security |
| 33, 34 | Breach assessment for the ~1,200 adversarial queries that executed through the full stack | Operational Security |
| 35, **36** | DPIA; and **prior consultation** if high residual risk remains unmitigated | GDPR §2 + Risk Disposition |
| 44–49 | Chapter V transfer determination, pending OD-02 and OD-04 | GDPR §2 |

Article 36 deserves a flag now rather than at Phase 5: if the DPIA leaves high residual risk that we cannot mitigate, prior consultation with the supervisory authority is **mandatory**, and we are already six months into live processing with critical findings open.

### 6.4 Adjacent regimes considered and scoped out

Declared explicitly, because silence on a regime an auditor expects to see considered reads as an oversight rather than a decision.

| Regime | Position | Rationale |
|---|---|---|
| **EU AI Act** | **Considered; out of scope for this package; separate determination required (OD-06)** | Annex III point 5(c) covers *risk assessment and pricing* in life and health insurance — ClaimsTriage does neither; it routes claims post-issuance. Point 5(d) covers *emergency healthcare patient triage systems* — ClaimsTriage triages claims, not patients. Neither is squarely engaged, but "determines clinical gravity" and "triage" sit close enough to both that a **documented negative determination** is required rather than silence. Timing has also moved: Annex III high-risk obligations were deferred from 2 August 2026 to **2 December 2027** under the Digital Omnibus provisional agreement (6 May 2026, confirmed by Member States 13 May 2026), binding on Official Journal publication. Article 50 transparency obligations were **not** deferred and remain live from 2 August 2026 — relevant if claimants interact with the system directly through the web interface. |
| **DORA** | Considered; out of scope, noted | Insurance undertakings are in scope of DORA (applicable since 17 January 2025), and NovaTriage would be an ICT third-party service provider. ICT risk management and third-party register obligations may overlap materially with this package's Operational Security section, but DORA compliance is not what next month's audit assesses. |
| **NIS2** | Considered; not engaged | The health sector annex covers healthcare providers, not insurers. |
| Financial/actuarial audit | Out of scope | The $87 average value and 0.042% leakage baseline are accepted as given and not independently verified. |

---

## 7. Integration Contract

**Every section of the package must be consistent with all twelve assertions below.** They are numbered so later sections can cite rather than restate, and so Phase 6 can check them mechanically.

| ID | Canonical assertion |
|---|---|
| **SCOPE-01** | The system under assessment is the Insurance Claims Triage AI ("ClaimsTriage") **v2.4**, LLM-based and RAG-augmented, in production since approximately January 2026, trained on NeuralCore v3.3.2. |
| **SCOPE-02** | **Meridian Health Partners Ltd.** is the sole controller for every processing activity in scope. |
| **SCOPE-03** | **NovaTriage Analytics Ltd.** is an Article 28 processor. Microsoft is a processor for ICT infrastructure and telemetry. |
| **SCOPE-04** | The system intakes claims via a **public-facing web interface**, classifies clinical gravity, routes to adjudication queues, flags high-risk claims for human review, and **dispatches automated transactional instructions** to downstream Claims Management Systems. |
| **SCOPE-05** | It is the **sole pre-authorization screening layer. No secondary or legacy fallback screening runs in parallel.** |
| **SCOPE-06** | Processing involves **automated decision-making with significant effects**. A High-Risk Human Review Queue exists; whether the human involvement in it is *meaningful* for Article 22 purposes is **unevidenced** (OD-03). |
| **SCOPE-07** | **Special-category data under Article 9(1) is processed across P-01 to P-07**: health data in narratives, medical documentation, adjuster notes, RAG clinical records and derived clinical-gravity scores; racial/ethnic origin in demographic monitoring. |
| **SCOPE-08** | The architecture presents **four attack surfaces, AS-01 to AS-04**, at the tokenization, embedding, attention/RAG and output-decoding stages respectively. |
| **SCOPE-09** | Scale: **~4,200,000 claims/day**, peaking at ~700,000 across 11:00–15:00; average claim value **$87**; baseline leakage **0.042%**. |
| **SCOPE-10** | Performance envelope: v2.4 at **94.2% TPR**; contractual SLA floor **91.0%**; rollback target v2.3 at **90.4% TPR**; rollback trigger at **rolling 24h TPR below 89.5%**. No other threshold exists in this package. |
| **SCOPE-11** | The active vulnerability is **CVE-2025-XXXX** — NeuralCore v3.1.x–v3.4.x, CVSS 8.6 High, **training-time** optimizer poisoning with **no inference-time effect**. Assessment window: 90 days of Defender telemetry to July 3, 2026, plus the pre-audit red-team evaluation. |
| **SCOPE-12** | The package addresses **GDPR and secure operations**. The EU AI Act, DORA and NIS2 are considered and scoped out with recorded rationale (§6.4). |

### 7.1 Explicitly out of scope

Recorded so that later sections cannot quietly expand the frame:

- **CVE-2026-38291** — not an advisory for this deployment; discarded with the artifacts that carried it
- ClaimsTriage v2.3, except as the rollback contingency in Operational Security
- Any other AI system operated by Meridian
- Underwriting and pricing models (distinct systems, distinct Annex III analysis)
- Fraud detection beyond the stated leakage baseline
- Security of the web claim portal itself, except at the AS-01 ingestion boundary
- Employee and HR personal data
- EU AI Act conformity assessment (OD-06, separate exercise)
- Independent verification of the financial baselines

---

## 8. Source-authority ledger

Each assertion in this frame is graded. An auditor asking "where does this come from?" gets an answer for every line.

| Grade | Meaning | Examples |
|---|---|---|
| **Stated** | Explicit in the scenario package | Volumes, TPR figures, CVE profile, four-stage architecture, AML layer deficiencies, MDC-2026-8891, 14.2% injection success |
| **Derived** | Arithmetic on stated figures | $365.4M/day transaction value; ~$153,468/day leakage; ~48.6 claims/sec at peak; 159,600 backlog claims |
| **Determined** | Governance judgment applied to stated facts, recorded as judgment | P-03 and P-04 as Article 9; the four unregistered activities P-05 to P-09; third parties in narratives as data subjects; Art 9(2)(g) as the condition; the AI Act negative determination |
| **Unresolved** | Not in the package; must not be assumed | OD-01 to OD-06 |

### Open determinations

| Ref | Question | Blocks | Owner |
|---|---|---|---|
| OD-01 | Main establishment and lead supervisory authority | Art 56, Art 36 routing | Lead Data Privacy Officer |
| OD-02 | NovaTriage country of incorporation | Chapter V analysis, RoPA transfers column | Lead Data Privacy Officer |
| OD-03 | Evidenced human-review rate and reviewer authority to override | **Art 22 position; risk disposition verdict** | AI Governance Lead |
| OD-04 | Azure region and Microsoft sub-processor chain | Chapter V analysis | Chief Information Security Officer |
| OD-05 | Named "additional jurisdictions" | Non-EU regime mapping | Lead Data Privacy Officer |
| OD-06 | AI Act Annex III negative determination, documented | AI Act readiness (separate) | AI Governance Lead |

**OD-03 is the one that can change the verdict.** If human review is not meaningful, Article 22(1) bites, and the package's central claim — that outputs "influence but do not solely determine" outcomes — collapses. Everything downstream of that claim would need re-argument.

---

## 9. Scope-drift watchlist

The specific drift risks in this package, drawn from where the source artifacts already disagreed. Phase 6 checks each one.

| # | Drift risk | Correct framing |
|---|---|---|
| 1 | Entity name reverting to "Contoso Health Partners" | Meridian Health Partners Ltd. throughout (SCOPE-02) |
| 2 | CVE-2026-38291 reappearing via the rollback or decision-matrix language | CVE-2025-XXXX only (SCOPE-11) |
| 3 | Describing the CVE as inference-time or portal-facing | Training-time, no inference-time effect (SCOPE-11) |
| 4 | The 92.5% threshold reappearing | Only 94.2 / 91.0 / 90.4 / 89.5 exist (SCOPE-10) |
| 5 | Rollback described as "rejected" | Rejected as the *primary remediation path*; **retained as a triggered contingency** |
| 6 | Article 9 flags reverting to "No" or "Conditional" | P-01 to P-07 are Article 9 (SCOPE-07) |
| 7 | Human review asserted as meaningful without evidence | Queue exists; meaningfulness unevidenced pending OD-03 (SCOPE-06) |
| 8 | Attack surfaces renumbered or reduced to three | AS-01 to AS-04, four surfaces (SCOPE-08) |
| 9 | AS-03 described as "the training data pipeline" | AS-03 is **inference-time RAG retrieval**; training-pipeline poisoning is a distinct concern under C3.1 |
| 10 | Output-schema validation described as present | Deficient; C2.2 corrected to Non-Compliant |
| 11 | "No third-country transfers" stated as a finding | Undetermined pending OD-02 and OD-04 |
| 12 | Retention conflict between 5-year record and session-close purge | Distinguish **intermediate processing cache** from **system of record** in every mention |
| 13 | System described as advisory or decision-support | It **dispatches automated transactional instructions** (SCOPE-04) |
| 14 | A parallel or legacy fallback implied | There is none (SCOPE-05) |

---

## 10. Integration goal

The audit does not assess four documents. It assesses whether four views of one system agree.

The four sections view the same deployment from different angles, and each angle carries an obligation the others depend on:

- **GDPR Compliance** establishes what data is processed, on what basis, with what rights attached. Every security control downstream exists to protect *this* data.
- **Model Security** establishes where the architecture can be attacked. Each surface maps to a stage in §2 and to data categories in §4.
- **Operational Security** establishes what has actually happened in production. Telemetry evidence must map to surfaces named in Model Security, not to new ones introduced late.
- **Risk Disposition** aggregates every finding from the other three. **No risk may appear here that was not raised in a preceding section, and no material finding raised in a preceding section may be absent here.**

The coherence test is bidirectional: a risk without a source section is an orphan, and a finding without a risk line is a gap. Both fail the same check.

---

**Phase 1 complete.** Phases 2 through 6 build on this frame. Where a later phase needs a fact this document grades as *unresolved*, it must be raised as an open determination — not filled in.
