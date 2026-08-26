# Phase 3 output — Model Security section, drafted for transfer

Destination: **Section 3 of the Compliance Package Report**. Finished text.

**Revision note:** reconciled against the Adversarial Red-Teaming & Security Test Report (MHP-ADV-REP-2026-v1, engagement 15–30 June 2026), received after the first draft. Three changes resulted: the hardening priorities now follow the red team's selection, three gap severities were revised **down** to the red team's direct-impact scoring, and the C2.2 correction gained independent corroboration.

---

## 3. Model Security Posture Assessment

### 3.1 Transformer attack surface mapping

Five distinct attack surfaces exist in the ClaimsTriage architecture. **Three are designated priority surfaces**, ordered by the severity the adversarial engagement assigned. All five are ranked, so the selection can be checked rather than taken on trust.

#### AS-04 — The inference output layer: decoding and outbound transactional dispatch · CRITICAL

**Stage 4.** From logit distribution through token selection to serialisation and dispatch of transactional instructions into the Automated Claims Adjudication Queue, the High-Risk Human Review Queue and Medical Underwriting Systems.

Threat vectors: adversarial instructions surviving the decoder serialised and dispatched as transactional calls to backend financial queues without structural verification; unreviewed model output executing unauthorised state changes; routing manipulation directing high-risk files into standard queues, defeating human-in-the-loop oversight at the final hop.

Control coverage: **C2.2 Non-Compliant — status corrected** · **C2.5 Non-Compliant** (no routing integrity verification).

**Directly evidenced.** Red-team finding **ADV-03 (CRITICAL)**: exploiting token logit distributions forced the output layer to append standard-processing routing tags to high-risk files, bypassing human review triggers. The AML assessment records the same outcome reached from the claim form at a **14.2%** success rate.

Feasibility HIGH × Impact HIGH = **CRITICAL**. The most severe verified finding in the deployment.

#### AS-01 — The model API endpoint: public claim ingestion and tokenization boundary · HIGH

**Stage 1.** The public-facing web claim interface through to the tokenizer. The only surface reachable by an unauthenticated member of the public.

Threat vectors: token-splitting and character-obfuscation strings defeating signature-matching filters; malformed multi-byte Unicode homoglyphs bypassing string-length filters and forcing out-of-vocabulary indices; long-form sequences engineered for context-window exhaustion; malicious content in uploaded PDF medical documents, which enter through a parsing path distinct from the text field and are subject to the same absent controls.

Control coverage: **C1.1 Partially Compliant** (45-minute demonstrated time-to-throttle; per-source thresholds defeated by IP-block distribution) · **C1.3 Non-Compliant** (no LLM-specific WAF signatures) · **C2.1 Partially Compliant** (keyword blacklist only).

**Directly evidenced twice — the only surface carrying both adversarial-test and production-telemetry evidence.** Red-team finding **ADV-01 (HIGH)** proved the gateway bypass is achievable. Alert **MDC-2026-8891** proved it is being attempted at volume against production: a sustained four-hour campaign from an external IP block, ~1,200 adversarial queries executing through the entire stack before throttling engaged.

Feasibility HIGH × Impact **MEDIUM** (instruction override, prompt extraction) = **HIGH**.

> Rated on **direct** impact. An earlier draft rated this CRITICAL by crediting its enablement of AS-03 and AS-04. That inflates every upstream surface to the severity of the worst thing downstream of it, and the red team's discipline is adopted instead.

#### AS-03 — The shared context window: attention layer and RAG assembly boundary · HIGH

**Stage 3.** Where untrusted claimant text, retrieved policy and clinical documents, and system routing instructions are assembled into a single context window.

Threat vectors: instruction-override sequences the model resolves as instructions, because nothing in the assembled prompt distinguishes instruction from data; adversarial formatting injected into the RAG repository forcing system prompt leakage; attention-weight manipulation down-weighting retrieved policy limits.

Control coverage: **C2.4 Non-Compliant and absent from the supplied checklist entirely** · **C2.1 Partially Compliant** · **C2.5 Non-Compliant**.

**Two distinct entry paths, one surface.** The AML assessment records instruction override at 14.2% via the claim form. Red-team finding **ADV-02 (MEDIUM)** compromised the same surface through the RAG ingestion channel, leaking the system prompt.

Feasibility HIGH via claim form / LOW via RAG repository × Impact MEDIUM–HIGH = **HIGH**.

#### Two surfaces ranked below the priority set

**AS-05 — the training data pipeline (HIGH).** Ranked fourth on **attacker feasibility alone**; impact equals any surface above it. Exploitation requires vendor-feed or build-environment access, not the public boundary. Impact is systemic rather than per-transaction: a poisoned corpus misclassifies every input matching an attacker-controlled pattern for the life of that model version. AS-01 through AS-04 compromise transactions; AS-05 compromises the classifier. Controls C3.1, C3.3, C3.4 all attach and all are Non-Compliant. **Assessed in Section 4**, where CVE-2025-XXXX is handled.

**AS-02 — the vector representation space (MEDIUM).** Not independently reachable; closing AS-01 substantially closes it. Its exploit degrades classification rather than hijacking it. Retained because MDC-2026-8891 is classified *"Anomalous Token Sequences & Repeated High-Entropy Inputs"* — this surface's exploit signature. It appeared on no map in the source pack.

#### The structural correction to the supplied map

The supplied Transformer Architecture report labelled the Stage 3 attention annotation *"the training data pipeline."* RAG retrieval happens at inference, against a live claim, reachable by anyone who can submit one. Corpus ingestion happens offline, during a build. Different attacker, different window, different controls. The consequence of merging them: **the deployment's actual training pipeline had no surface of its own**, which is why a training-time CVE appears in no attack surface map anywhere in the source pack.

---

### 3.2 Adversarial test findings ledger

Three exploits verified by the Principal Adversarial Engineer, 15–30 June 2026.

| ID | Verified exploit | Severity | Surface | Reconciliation |
|---|---|---|---|---|
| **ADV-03** | Output token hijacking and queue redirection — logit distribution exploitation forced standard-processing routing tags onto high-risk files, bypassing human review triggers | **CRITICAL** | AS-04 | Accepted as scored |
| **ADV-01** | Direct prompt injection via obfuscated medical narrative strings — token-splitting bypassed web gateway filters, allowing system instruction overrides | **HIGH** | AS-01 | Accepted as scored |
| **ADV-02** | Indirect prompt injection via vector DB / RAG ingestion — adversarial formatting in historical claims repositories forced the attention window to leak the system prompt | **MEDIUM** | **AS-05 (write) → AS-03 (read)** | Severity accepted; **surface attribution corrected** |

#### Why ADV-02 validates the AS-03 / AS-05 separation

The red team recorded ADV-02 against *"the training data pipeline."* The exploit has two halves: a **write** into the claims repository — a corpus ingestion event, control C3.1 — and a **read** at inference, where poisoned content enters the context window, control C2.4. The attacker writes once and the payload is read back later, against unrelated claims and unrelated claimants.

Neither control substitutes for the other, **and a map with one surface can only name one of them.** That is precisely how this exploit came to be labelled a training-pipeline issue in a package whose training pipeline had no surface at all.

#### A methodology note on the rubric

The rubric returns two different answers for the same finding, and the package should state which method governs.

ADV-02 is system prompt extraction. **Section 2** classifies by consequence alone and lists "complete core framework Prompt Leakage (system prompt extraction)" under **HIGH**. **Section 3** crosses feasibility with impact, placing prompt leakage in the *Medium Impact* column — so at the low feasibility of write access to internal repositories, it returns **LOW to MEDIUM**.

The red team applied Section 3, and its three ratings are internally consistent with that matrix. **This package follows Section 3 throughout, and records the choice** so a reviewer arriving at a different figure from Section 2 can see why rather than treating it as an error. Where they disagree, Section 3 governs: feasibility is a real constraint on risk, and a classification ignoring it over-rates every theoretical finding.

---

### 3.3 Hardening priorities

The adversarial engagement isolated two strategic priorities, carried forward unchanged. A third is added.

**The red team's selection is not the obvious one, and it is worth stating why it is right.** It prioritised the two **ends** of the attack chain — the boundary an attacker must enter through, and the boundary where damage lands — rather than the attention layer in the middle where the instruction override actually occurs. AS-03 is the model's own reasoning; no deterministic control can be placed inside attention, and encapsulation reduces the probability of override without enforcing an outcome. AS-01 and AS-04 are the two points in this architecture where a control can **refuse a request outright**. Harden what can be enforced, then mitigate what can only be influenced.

#### HP-01 — Output token schema sanitisation and structural validation · AS-04 · CRITICAL

From **ADV-03**. Feasibility HIGH × Impact HIGH = CRITICAL; matches the rubric's CRITICAL worked example almost verbatim. **Mandated SLA: 24-hour hotfix — deliverable in window**, since this is an interposed validation layer, not a change to model behaviour, and carries no retraining or regression exposure against the accuracy floor.

Control: strict pydantic output data contracts and a hard structural validation layer between decoder and dispatch. Any token sequence attempting to overwrite internal claims-routing tags is dropped before reaching adjudication queues. **Fail closed** — a validation failure holds the claim for human handling rather than dispatching a best-effort payload. Enforce separation between generated response content and routing control tokens. **Owner: Application Security Lead.**

> **Three-way corroboration of the C2.2 correction.** (1) The AML configuration report states the decoder does not validate output against outbound JSON schemas. (2) The adversarial report *recommends deploying pydantic output contracts* as its first hardening priority — a remediation no red team proposes for a control already in place. (3) ADV-03 exploited the absence directly. Against this, the supplied checklist scored the control Compliant on the basis that schemas are "hard-validated against strict pydantic definitions." **Two independent primary sources and a verified exploit against one checklist entry.**

#### HP-02 — LLM input guardrails and token-splitting heuristic validation · AS-01 · HIGH

From **ADV-01**. Feasibility HIGH × Impact MEDIUM = HIGH. **Mandated SLA: 72-hour mitigation cycle — deliverable in window.**

Control: an input guardrail layer ahead of the tokenizer — deterministic token-length constraints, strict Unicode normalisation with homoglyph folding to defeat token-splitting, and a semantic pre-classification model that identifies and drops injection patterns rather than matching known strings. Extend the WAF with OWASP Top 10 for LLM signatures. **Apply the same controls to the PDF document parsing path**, which enters through a different route and is subject to the same absent controls. **Owner: Principal AI Engineer.**

#### HP-03 — Context window structural encapsulation · AS-03 · HIGH · added

From **ADV-02** and the AML processing-layer finding. Added because **control C2.4 was absent from the supplied checklist entirely** — the deployment scored nine controls without one covering the mechanism by which instruction override occurs. A missing control scores as nothing at all, which is how a 55.6% compliance figure coexisted with this gap. Ranking it third respects the red team's judgement while ensuring the control is still built.

Control: encapsulate untrusted claimant text within cryptographically random per-request delimiters, regenerated each request so the boundary cannot be guessed. Establish an explicit instruction hierarchy in which system directives and retrieved content outrank anything originating in a claimant field. Constrain RAG retrieval scope to the claimant's own records absent a documented basis — a privacy boundary and a security boundary simultaneously. **Owner: Principal AI Engineer.**

#### The remediation window, stated rather than buried

Two of the three priorities meet their SLA without difficulty. **HP-03 does not**: context window encapsulation is a prompt architecture change requiring regression testing against a 4.2 million claim per day system with a 91.0% accuracy floor and 3.2 points of headroom. Seventy-two hours is not an honest estimate.

The supplied report template assigned model-security remediations target dates of 12 and 15 July against a 3 July package date — nine and twelve days out — with no acknowledgement that a rubric in the same package mandates 24 and 72 hours.

| Window | Action | Priority / Surface |
|---|---|---|
| **24 hours** | Pydantic output contracts and structural validation, failing closed to human handling; raised alerting on routing anomalies | HP-01 · AS-04 — **SLA met** |
| **24 hours** | Interim compensating controls for HP-03: block known injection structural variants at input; force human review on every settlement-queue routing above a value threshold | AS-01, AS-03 |
| **72 hours** | Input guardrail layer — token-length caps, Unicode normalisation with homoglyph folding, semantic pre-classification; WAF extended with OWASP LLM signatures | HP-02 · AS-01 — **SLA met** |
| **7 days** | Full context window encapsulation with per-request random delimiters and instruction hierarchy, after regression testing against the 91.0% TPR floor | HP-03 · AS-03 — **SLA exceeded by 4 days** |
| **Throughout** | Documented risk acceptance for the HP-03 interval, signed by the AI Governance Lead, recording residual exposure and the compensating controls relied on | AS-03 |

**One priority exceeds its SLA, by four days, for a stated engineering reason, with compensating controls in place and a named owner accepting the gap.** That is defensible in front of an auditor. Nine days with no acknowledgement is not.

---

### 3.4 Defence-in-depth configuration gaps

Twelve controls assessed. **Systemic compliance score: 25.0%** — three compliant, two partially compliant, seven non-compliant.

The supplied checklist reported 55.6% over nine controls. The posture did not deteriorate. **It was previously measured against a control set that omitted the controls this deployment most needs**: one status corrected against primary sources (C2.2), three absent controls added (C2.4, C2.5, C3.4), one downgraded on production evidence (C1.1).

| Layer | Gap | Weak or missing control | Exposure | Severity | ADV |
|---|---|---|---|---|---|
| Output | DID-02 (C2.2) | No outbound JSON schema validation before transactional dispatch | Adversarial instructions surviving the decoder execute state changes in adjudication systems | **CRITICAL** | ADV-03 |
| Output/Processing | DID-05 (C2.5) | No reconciliation of intended routing against actual queue placement | A successful routing bypass produces no signal; the system cannot evidence that human review occurred | **CRITICAL** | ADV-03 |
| Processing | DID-01 (C2.4) | No structural encapsulation of untrusted input in the context window | Model cannot distinguish instruction from data; policy limits overridden, human review bypassed | HIGH | ADV-02 |
| Input | DID-03 (C1.3) | WAF carries generic layer-7 rules only, no OWASP LLM signatures | Injection payloads are well-formed HTTP; nothing in a generic ruleset distinguishes hostile from legitimate | HIGH | ADV-01 |
| Input | DID-09 (C2.1) | Keyword blacklist only; no semantic anomaly detection | A blacklist enumerates known bad strings; the attack class is defined by intent. Live during the 14.2% result | HIGH | ADV-01 |
| Input | DID-08 (C1.1) | Threshold-based per-source rate limiting; 45-minute demonstrated time-to-throttle | The observed attack came from an IP *block*; per-source thresholds are structurally defeated by distribution across one | HIGH | MDC-2026-8891 |
| Data & Governance | DID-04 (C3.1) | No cryptographic hash-chain verification of corpus or RAG integrity at ingestion | No way to demonstrate the v2.4 corpus was untampered, so CVE-2025-XXXX cannot be excluded. Also the **write-side control for ADV-02** | HIGH | ADV-02 (write) |
| Data & Governance | DID-06 (C3.3) | No DP-SGD; no epsilon-delta budget | Membership inference and text memorisation over Article 9 narratives. **Blocks the de-identification recommendation at §2.3** | HIGH | — |
| Data & Governance | DID-07 (C3.4) | No signed model bill of materials | Advisory triage becomes manual archaeology under time pressure and may return no answer | MEDIUM | — |

> Severities for DID-01, DID-03 and DID-09 were revised **down** from CRITICAL after reconciliation. The earlier ratings credited each surface with enabling exploits downstream of it. Two CRITICAL gaps remain, both attached to ADV-03.

#### The structural finding

**Four of nine gaps sit in a layer the AML security configuration report never examined.** That report assesses Input, Processing and Output — the runtime inference path only. Training-time integrity, corpus provenance, differential privacy and supply-chain attestation have no place in its taxonomy and were never assessed.

That is where CVE-2025-XXXX lives, **and it is also where the write-side control for ADV-02 lives**. A reader of the AML report alone would conclude the deployment has three deficient layers; the checklist shows four, and the unexamined one carries both the CVE exposure and half of a verified exploit. A defence-in-depth review scoped to the inference path is not a defence-in-depth review.

#### Layer crosswalk

| AML report layer | Verdict | Checklist controls | Net status |
|---|---|---|---|
| Input | Deficient | C1.1 (Partial), C1.3 (Non-Compliant), C2.1 (Partial) | **Deficient — confirmed.** All three fail against the same attack class for the same reason: each enumerates known-bad patterns rather than modelling intent |
| Processing | Deficient | C1.2 (Compliant), C2.3 (Compliant), C2.4 (Non-Compliant, added) | **Deficient — confirmed.** The two compliant controls protect the network and observe the runtime; neither addresses prompt structure |
| Output | Deficient | C2.2 (Non-Compliant, corrected), C2.5 (Non-Compliant, added) | **Deficient — confirmed.** Verifying output is well-formed is not verifying it went where intended |
| *(none)* | **Not assessed** | C3.1, C3.2, C3.3, C3.4 | **3 of 4 Non-Compliant** |

One control deserves credit. **C2.3 real-time telemetry is Compliant and performed** — it generated MDC-2026-8891. Detection is not prevention: the alert fired and 1,200 queries still completed. But telemetry is the reason this deployment knows about the exposure at all.

---

### 3.5 What Model Security hands forward

| Output | Destination | Constraint |
|---|---|---|
| AS-01 primary, AS-02 contributory | §4.1 telemetry | MDC-2026-8891 maps to AS-01 — the surface HP-02 addresses. The adversarial report **requires** its priority surfaces map directly to Defender telemetry. No new surface may be introduced in §4 |
| AS-05, controls C3.1 / C3.3 / C3.4 | §4.2 CVE response | The CVE attaches here, named so §4 cites rather than invents it. C3.1 is also ADV-02's write-side control |
| HP-01, HP-02, HP-03 | §5 risk disposition | Severities and SLAs carried, with the four-day HP-03 exception and its named risk acceptance |
| ADV-01, ADV-02, ADV-03 | §5 risk disposition | Rubric §4 and the adversarial report §4 both require CRITICAL and HIGH findings logged with a role owner and a compliant close date |
| DID-01 to DID-09 | §5 risk disposition | Nine gaps; none may be dropped |
| DID-06 (C3.3) | §2.3 cross-reference | Technical precondition of the de-identification recommendation |
| DID-05 (C2.5) | §2.2 cross-reference | The evidential control DPIA GAP-05 requires for the Article 22 position |

Two are deliberate cross-links back into GDPR Compliance. **C3.3 and C2.5 are not only security gaps** — the first blocks the de-identification control recommended in §2.3, the second is the only control that would evidence the human review the Article 22 position depends on. ADV-03 makes that second link concrete: the red team bypassed human review triggers and nothing detected it.
