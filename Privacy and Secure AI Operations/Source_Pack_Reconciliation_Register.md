# Source Pack Reconciliation Register
**Insurance Claims Triage AI (v2.4) — Privacy & Secure Operations Compliance Package**
Prepared by: AI Governance Lead, Meridian Health Partners
Purpose: establish one authoritative canon before Phase 1 drafting, and record every source-artifact conflict overridden to reach it.
Status: Pre-Phase 1 (read-and-check complete)

---

## 0. Canon locked for this package

| Item | Canonical value | Overrides |
|---|---|---|
| Entity | **Meridian Health Partners Ltd.** | "Contoso Health Partners" in the scenario header, RoPA controller field, and all three .xlsx titles |
| Vulnerability | **CVE-2025-XXXX** — NeuralCore v3.1.x–v3.4.x, improper input validation in the gradient descent optimizer, CVSS 8.6 High, **training-time** exposure | CVE-2026-38291 (tokenizer RCE, CVSS 9.1) in the Patch-vs-Retrain matrix and Rollback Plan is discarded entirely |
| Package effective date | **July 3, 2026** (assumed — matches all seven completed artifacts) | — |
| Report reference | MHP-AI-GOV-2026-PKG01 | — |
| DPIA reference | **DPIA-VTA-2024-003** (+ `-MitigationPlan` for the plan) | "DPIA-MHP-003" in scenario §2.2 |
| Rollback target | **ClaimsTriage v2.3** on NeuralCore v2.9.1, 45 min execution | v1.4.2-stable / 12 min in the Rollback Plan template |
| Appendix | **GenAI Audit Trail** | Template's pre-written Non-Use Governance Attestation |
| Scope of rebuild | All inconsistent artifacts re-issued against this canon | — |

**Canonical ID schemes** (to be used identically in every artifact and every report section):

- Attack surfaces: `AS-01`…`AS-04`
- Risks: `RSK-01`…`RSK-nn`
- DPIA gaps: `GAP-01`…`GAP-nn`
- AML controls: `C1.1`…`C3.3` (existing), extended where controls are missing

---

## Class A — Audit-fatal contradictions

These are defects an auditor finds by reading two of our own documents side by side. Each one on its own is sufficient to trigger a remediation order.

### A1. The RoPA workbook contradicts itself on Article 9

The same file gives two opposite answers for the same data element:

| Source | Claim Narrative Free-Text Body — Art 9 flag |
|---|---|
| RoPA .xlsx, Tab 1 "RoPA — Partial", row 2 | **Yes** — Article 9(2)(g) |
| RoPA .xlsx, Tab 2 "Lawful Basis Classification", row 2 | **No** — N/A |
| Scenario package §2.2 | **No** — N/A |
| Compliance Report template, Table 2 | **Yes** — Article 9(2)(g) |

**Resolution: Yes, Article 9(2)(g).** The system is described as processing "claim narratives and supporting medical documentation" and determining "clinical gravity." That is data concerning health under Art 9(1) on any reading. Every "No" entry is wrong and must be corrected in both tabs and in §2.2.

### A2. Two different vulnerabilities are documented as "the" CVE

| Artifact | CVE | CVSS | Nature | Verdict recorded |
|---|---|---|---|---|
| Scenario package §6 | CVE-2025-XXXX | 8.6 High | Training-time optimizer poisoning; **no inference-time effect** | Full retrain on v3.5.0; patching = compensating control only |
| Patch-vs-Retrain matrix | CVE-2026-38291 | 9.1 Critical | Tokenizer buffer overflow → RCE at the public portal | Hybrid: immediate gateway patch + retrain next sprint |
| Rollback Plan | CVE-2026-38291 | — | "High-Severity Base Library Exploit" | 12-min reversion to v1.4.2 |

Resolved to CVE-2025-XXXX. Consequence: the matrix must be **rescored**, not merely relabelled — see B1.

### A3. The rollback plan trips its own trigger on arrival

- Rollback Plan trigger: execute rollback if accuracy falls **below 92.5%**.
- Rollback target v2.3 operates at **90.4% TPR**.

Rolling back to v2.3 lands 2.1 points below the threshold that mandates rolling back. The procedure is self-defeating as written. The scenario package's trigger (**TPR below 89.5%**) is the coherent one and becomes canon. Three thresholds are in play across the pack — 89.5% (trigger), 91.0% (SLA floor), 92.5% (rollback plan) — and only the first two can coexist.

### A4. The defense-in-depth checklist contradicts the AML report on the output layer

| Source | Output-layer schema validation |
|---|---|
| Scenario §5.2 (AML config report) | **Deficient** — "does not validate generated output strings against outbound JSON schemas" |
| Defense-in-Depth checklist, C2.2 | **Compliant** — "hard-validated against strict pydantic definitions" |
| Compliance Report §3.3 + RSK-03 | Treated as a **gap** requiring remediation by July 12 |

We cannot simultaneously score this control compliant and open a risk to remediate it. **Resolution: C2.2 → Non-Compliant.** Recomputed systemic compliance score: **4/9 = 44.4%** (was 55.6%), and 4/10 = 40% once the missing control in C2 is added (see C3).

### A5. Three different DPIA gap sets

| Source | Gap count | Gap 1 classification | Cache purge window |
|---|---|---|---|
| Scenario §3.2 | 2 | Present-but-Weak | At session close |
| DPIA Mitigation Plan (completed) | 3 | **True Gap** | Within 24 hours |
| Compliance Report §2.2 | 2 | Present-but-Weak | Within 5 minutes of DB write |

Same gap, three classifications and three windows. The DPIA plan's own self-check confirms "at least three gaps," so **three is the target count**, and one canonical purge window must be chosen and repeated verbatim everywhere.

### A6. Three incompatible attack-surface ID schemes

| Scenario §4 | Transformer doc | Report §3.1 | Layer |
|---|---|---|---|
| AN-SURF-01 Unicode Normalization Boundary | "the model API endpoint" | VUL-TOK-01 | Tokenization |
| AN-SURF-02 Vector Representation Space | *(absent)* | *(absent — numbering gap)* | Embedding |
| AN-SURF-03 Shared Context Window Assembly | "the training data pipeline" | VUL-ATT-03 | Attention / RAG |
| AN-SURF-04 Outbound Transactional API | "the inference output layer" | VUL-DEC-04 | Output decoding |

Two substantive problems beyond the naming: the Transformer doc labels the **Attention/RAG ingest** as "the training data pipeline," conflating inference-time retrieval with training-time ingestion — these have different threat models and different controls. And AN-SURF-02 was dropped from every downstream artifact despite being the surface the Defender alert actually evidences (see C1).

### A7. Empty mandatory fields carried into the submission file

| Location | Field left blank |
|---|---|
| Report §1, priority #2 | "Enforce Third-Party Trust Boundaries:" — no body text |
| Report §2.2, Gap 2 | Effectiveness Criterion — empty (also empty in scenario §3.2) |
| Report §2.3 | "Recommendation Verdict:" — empty |
| Report §2.3 | "Stated Residual Risks for Product Owner Acceptance:" — empty |
| Report §5, RSK-01 | Assigned Closed Control Mitigation — empty |

RSK-01 being blank directly falsifies the sentence immediately above the table: "No unmapped risks are included."

---

## Class B — Substantive technical and legal defects

### B1. The Patch-vs-Retrain matrix cites a score that is not its own verdict

Arithmetic checks out (weights sum to 1.00; weighted scores 4.25 / 2.30 / 4.05 against raw sums 21 / 12 / 20). But:

- Recommended strategy: **Hybrid** (4.05)
- "Matrix Verification Score" recorded: **2.3** — that is Strategy 2, Retraining Only
- Highest-scoring strategy: **Patching Only** (4.25), which is not what was recommended

Worse, the scoring is calibrated for an inference-time vulnerability. For a training-time backdoor, "Speed to Implement & Time-to-Protection = 5.0" for Patching Only is wrong on the scenario's own logic: *"Patching the production inference environment closes no training-time exposure that has already occurred."* That criterion should score 1. Rescoring is mandatory, not cosmetic.

### B2. Differential privacy is applied to the wrong object

Scenario GAP-02 control: *"noise injection and structural differential privacy transformations on all historical claim corpora prior to exporting datasets to the external processor."*

DP-SGD perturbs **gradients during training**. It does not sanitise a corpus being exported. If raw narratives leave our boundary for NovaTriage to fine-tune, DP applied inside their training run protects the resulting model — not the corpus in transit or at rest on their infrastructure. The stated control does not close the stated gap.

**Correct split:** NER masking (+ Art 28 DPA + Chapter V transfer mechanism) protects the **export**; DP-SGD protects the **model** against membership inference. The De-Identification Matrix's hybrid recommendation is right; the scenario's Gap 2 wording and the report's §2.3 are both wrong, in different directions.

### B3. The anonymisation claim is overstated

Report §2.3 asserts DP "satisfies the EDPB anonymisation tests for singling out, linkability, and inference, providing a legally defensible stance for exporting data." A DP-SGD-trained model is not thereby anonymous data, and the guarantee is meaningless without a stated epsilon-delta budget — which appears nowhere in the pack. This paragraph as written invites direct challenge. It needs the epsilon budget stated and the claim scoped to membership-inference resistance rather than legal anonymisation.

### B4. Article 22 exposure is unaddressed anywhere in the package

The pack asserts meaningful human review. The same pack states:

- "the **sole** primary screening layer" and "**no** secondary legacy fallback screening layer runs in parallel" (§1)
- "automatically determining clinical gravity and dispatching **automated transactional instructions**" (§1)
- ADM indicator: "**Yes — Significant Effects**" (RoPA row 4)
- Red team: 14.2% of variations routed unverified cases **directly to settlement queues**, bypassing review (§5.1)

If human involvement is not meaningful — performed by someone with authority and competence to override — Art 22(1) applies, requiring an Art 22(2) ground, Art 22(3) safeguards, and under **Art 22(4)** special-category data may only be processed by such decisions under Art 9(2)(a) or 9(2)(g). This is the single largest legal exposure in the deployment and it appears in **no** risk line. It needs an evidenced human-review rate, not an assertion.

### B5. "No transfers identified" is an assertion, not a finding

All four RoPA rows record "None identified at this stage" — while the scenario describes a **multinational** operating "across the EU and additional jurisdictions," a processor styled **NovaTriage Analytics Ltd.**, and Microsoft Defender for Cloud / Azure Monitor telemetry. Chapter V requires a positive determination: adequacy, or Art 46 safeguards plus a transfer impact assessment. "None identified at this stage" tells an auditor we have not looked.

### B6. The RoPA is incomplete against Article 30(1)

Columns present cover most of Art 30(1). Missing outright:

- **Art 30(1)(a)** — controller and **DPO contact details**
- **Art 30(1)(d)/(e)** — identification of third-country recipients and **documentation of suitable safeguards**
- **Art 30(1)(g)** — general description of **technical and organisational security measures**

The last is a straight completeness failure: there is no column in which our defense-in-depth posture connects to the processing register.

### B7. DPO routing contradicts the register's own triggers

Rows 2 and 3 route to "DPO Required" on Article 9 grounds. Row 4 (Automated Triage Output) is marked "Controller Only" **despite** ADM with significant effects and a live DPIA reference — the strongest DPO trigger present. Row 5 (Historic Adjuster Notes) is "Controller Only" despite acknowledged possible incidental health disclosure.

### B8. "Conditional" is not a valid Article 9 value

Historic Adjuster Notes are flagged "Conditional — Post-Redaction Review Required." Article 9 either applies to a category of data or it does not; redaction is a **minimisation control**, not a basis for de-flagging. Given that NER false negatives are an acknowledged residual risk in our own De-Identification Matrix, the honest entry is **Yes**, with redaction recorded as the control.

### B9. Derived triage scores are classified as non-Article 9

Both the RoPA (row 4) and Report Table 2 flag the automated triage output as "No — N/A (Derived Internal Operational Sorting Index)." A score expressing **clinical gravity**, derived from health narratives, reveals health status. Recommended reclassification to Article 9 — flagging this as a judgment call an auditor will probe either way, so it needs a reasoned position rather than a bare "No."

### B10. Retention and purge windows are mutually exclusive as written

Narratives carry a **5-year** retention (RoPA) alongside a control promising "**zero** free-text content retained beyond the active triage routing window" (scenario GAP-01). Both can be true only if the register distinguishes the **intermediate processing cache** from the **system of record**. As drafted they read as a direct contradiction.

### B11. The prompt-injection rubric was never actually applied

Running the 14.2% Adversarial Hardship Narrative finding through our own rubric:

- Attacker Feasibility: **High** — public web claim boundary
- Maximum Potential Impact: **High** — queue hijacking, human review bypassed
- Matrix cell: **CRITICAL SEVERITY**
- Mandated SLA: **Immediate triage; 24-hour hotfix**

The finding also matches the CRITICAL row's worked example almost verbatim ("forcing the system to dispatch automated transactional API calls… bypassing all human review workflows").

**RSK-02 is dated July 15, 2026 — twelve days out.** The risk register breaches the SLA of the rubric held in the same package. RSK-03 (unvalidated output → backend execution) is likewise CRITICAL under the rubric and dated July 12. Either the dates move to within 24 hours, or we document explicit compensating controls and a named risk acceptance.

---

## Class C — Coverage gaps

### C1. The telemetry evidence maps to a surface nobody tracks

Alert MDC-2026-8891 is classed "Anomalous Token Sequences & **Repeated High-Entropy Inputs**." That is the precise exploit note for **AN-SURF-02** (Vector Representation Space): *"highly repetitive, high-entropy token configurations generate extreme vector magnitudes that distort positional encoding."* AN-SURF-02 was dropped from the Transformer doc and from the report. We have live telemetry evidencing an attack surface absent from our own mapping.

### C2. Risks required by our own artifacts are missing from the register

The De-Identification Matrix §3 states its three residual risks "**must** be explicitly listed as active line items inside the Risk Disposition section" and warns that failure to cross-reference "will cause an audit failure." None of the three appears. The register is also silent on:

| Missing risk | Source |
|---|---|
| Semantic re-identification via quasi-identifiers | DeID Matrix §1 |
| Utility/privacy degradation on rare conditions | DeID Matrix §1 |
| NER false negatives leaving identifiers unmasked | DeID Matrix §1 |
| No cryptographic ingestion hashing (poisoning) | Checklist C3.1 — Non-Compliant |
| No DP-SGD (membership inference) | Checklist C3.3 — Non-Compliant |
| WAF lacks LLM-specific rules | Checklist C1.3 — Non-Compliant |
| Article 22 / meaningful human review | B4 above |
| Chapter V transfer determination absent | B5 above |
| ABAC + encryption for flagged claim queues | DPIA GAP-03 |

Four risk lines cannot carry a package with this many open findings.

### C3. The checklist has no control for the deployment's largest gap

Nine controls are scored across three layers. **None** covers context-window encapsulation / structural prompt isolation — the exact deficiency behind the 14.2% injection success rate and the scenario's "Processing Layer (Deficient)" finding. A missing control scores as nothing at all, which is how a 55.6% compliance score coexists with a critical unmitigated vulnerability. Recommend adding **C2.4 — Context Window Structural Encapsulation (Non-Compliant)**.

### C4. Two incompatible "three layers"

The scenario's AML report uses **Input / Processing / Output**. The checklist uses **Network & Infra / Model Application / Data & Governance**. Report §3.3 uses the first, the checklist evidence uses the second, and no crosswalk exists — so §3.3's findings cannot be traced to the control they came from. A mapping table is required.

### C5. Processing activities absent from the RoPA

Not registered anywhere: **supporting medical documentation** (the Transformer doc's Stage 1 explicitly ingests "multi-format PDF narrative documents"), the **RAG knowledge base** of historical clinical records queried at inference, the **high-risk human review queue**, **inference/prompt telemetry logging**, and the **vendor export** itself as a distinct transfer activity.

### C6. Missing legal-basis row

Report Table 2 covers three data categories. The RoPA covers four — **Historic Adjuster Notes** has no row in the report's legal basis matrix.

---

## Class D — Editorial and precision defects in the submission file

| Location | Defect |
|---|---|
| §1 | "card-network/regulatory SLA floor" — card-network is a payments artifact, wrong industry |
| §1 | Rollback described as "explicitly rejected" while §7 keeps a live triggered rollback plan. Correct framing: rejected as the *primary remediation path*, retained as a *triggered contingency* |
| §1 | "trigger mandatory regulatory notifications within 48 hours" — basis unstated; not a GDPR breach, so presumably contractual |
| §1 | "schema schema-checking" |
| §4.2 | "91.0% SLA **ceiling floor**" |
| §4.2 | "fully satisfying the rollback contingency tracking requirements" — vague, no referent |
| Attestation | Signed "Meridian Health Partners" while the header reads "Contoso Health Partners" |
| DPIA plan header | "Insurance Claims Triage AI (Virtual Triage Assistant)" — VTA naming leftover; reference dated 2024 for a system deployed January 2026 |
| Rollback §5 | "passes 100% of the Prompt-Injection Triage Rubric baseline test suits" — the rubric is a severity classification framework, not a test suite |
| Rollback §2 | Triggers scoped "within 48 hours post-deployment" for a system deployed six months ago; needs scoping to the patch deployment |
| Rollback | Owner "Lead AI Infrastructure Engineer" vs scenario's "Director of AI Production Infrastructure Operations" |

---

## Arithmetic and internal-consistency verification

| Check | Result |
|---|---|
| 4,200,000 × 3.8% = 159,600 claims/day | ✓ |
| 94.2% − 90.4% = 3.8 pp | ✓ |
| 91.0% − 90.4% = 0.6 pp below floor | ✓ |
| $0.42 per $1,000 = 0.042% | ✓ |
| Matrix weights sum to 1.00 | ✓ |
| Weighted scores 4.25 / 2.30 / 4.05; raw 21 / 12 / 20 | ✓ arithmetic — but verdict cites the wrong figure (B1) |
| Compliance score 5/9 = 55.6% | ✓ as scored — invalid once C2.2 corrected (A4) |
| Daily transaction value: $87 × 4.2M = $365.4M | ✓ derived |
| Baseline leakage at 0.042% = ~$153,468/day | ✓ derived |
| **Peak throughput vs rate limit** | ⚠ 700,000 claims / 4h = ~48.6/sec, against C1.1's "100 requests/min" (1.67/sec). Reconcilable only if the cap is per-source-IP or per-instance — needs stating, or the control reads as incompatible with the SLA |

---

## Sequencing note

The Class A items are not stylistic. A1 (Article 9), A4 (output-layer control status) and B11 (rubric SLA) each determine content in three or more report sections, so they must be settled in Phase 1–2 rather than patched during the Phase 6 consistency check. B4 (Article 22) may change the risk disposition verdict itself and should be evidenced early.
