# Phase 4 output — Operational Security section, drafted for transfer

Destination: **Section 4 of the Compliance Package Report**. Finished text.

---

## 4. Operational Security and Telemetry Interpretation

### 4.1 Telemetry analysis — incident MDC-2026-8891

**One high-severity alert appears in the 90-day Defender for Cloud window.**

| Field | Value |
|---|---|
| Alert ID | MDC-2026-8891 |
| Severity | High |
| Classification | Anomalous Token Sequences & Repeated High-Entropy Inputs Detected on Claims Inference Endpoint |
| Duration | 4 continuous hours |
| Source | Automated scripts from an external IP **block** |
| Target | Primary web text input fields |
| Signature | Patterns designed to induce context-window resource exhaustion and trigger application-layer routing bypasses |
| Mitigation | Network edge rate-limiting throttled source IPs after **45 minutes** |
| Outcome | Approximately **1,200 adversarial queries executed through the entire transformer stack** before mitigation |

#### What it indicates

The alert classification maps directly onto attack surfaces named in Section 3, and it maps onto **three of them at once** — which is what makes it a chain rather than a probe.

- *"Anomalous token sequences"* is **AS-01**, the model API endpoint. This is the surface HP-02 addresses, and the surface red-team finding ADV-01 independently proved bypassable via token-splitting.
- *"Repeated high-entropy inputs"* is the exploit signature of **AS-02**, the vector representation space: repetitive high-entropy token configurations generate extreme vector magnitudes that distort positional encoding. AS-02 appeared on no attack surface map in the source pack, so the deployment held telemetry evidencing a surface it had not mapped.
- *"Application-layer routing bypasses"* is the intended outcome at **AS-03 and AS-04** — the same outcome ADV-03 achieved and the same outcome the AML assessment measured at a 14.2% success rate.

**No new surface is introduced here.** Every element of this alert was named in Section 3 before the telemetry was read.

#### The escalation outcome, and what it actually demonstrates

Three controls were exercised. Their performance is not uniform, and the package should say so rather than reporting a clean save.

**C2.3 real-time telemetry performed.** The alert exists because this control works. It is the reason the deployment knows about the exposure at all, and it deserves credit in a section otherwise full of failures.

**C1.1 rate limiting partially performed, and the timing is the finding.** Forty-five minutes elapsed before throttling engaged against a sustained automated campaign. But the more revealing number is the arithmetic:

> 1,200 queries across the 45-minute pre-mitigation window averages roughly **27 requests per minute in aggregate** — well below the configured 100-per-minute threshold, and far below it once distributed across an IP *block*.

**Whatever tripped the throttle, it was not these queries.** The control fired on high-volume noise and never applied to the traffic that actually completed the stack. A per-source threshold is structurally defeated by distribution across a block, and this incident is the demonstration.

**C1.3 LLM-aware WAF inspection did not perform, because it does not exist.** This is the control that should have raised the alert in minutes on signature rather than in 45 on volume. Its absence is why edge rate-limiting — the wrong layer, responding to the wrong property of the traffic — was the only thing standing between an external actor and the transformer stack.

#### The question the source pack does not ask

**No forensic analysis was performed on the 1,200 queries that completed. No Article 33 breach assessment exists.**

That is the most consequential gap in the operational security posture, and it is entirely absent from the supplied package.

The reasoning is direct. Approximately 1,200 adversarial queries executed through the full transformer stack of a system whose two demonstrated failure modes are **routing bypass** (ADV-03, CRITICAL — high-risk claims diverted from human review) and **system prompt leakage** (ADV-02). Either outcome, if achieved, engages Article 4(12):

- **Routing bypass** on Article 9 claims is unauthorised alteration of processing — claims concerning health diverted from the human review that constitutes the Article 22(3) safeguard.
- **Prompt leakage** discloses system configuration, and the context window at the moment of leakage contains RAG-retrieved clinical records — potentially a confidentiality breach of Article 9 data.

Applying the red team's 14.2% rate to 1,200 queries yields roughly **170 potentially successful bypasses**. That figure is a bounding illustration and nothing more: red-team payloads were purpose-built and tested under controlled conditions, and observed traffic may be more or less effective. **The point is not the estimate. The point is that the actual number is unknown, is knowable only through forensic analysis that was never performed, and that zero is not the safe default.**

Article 33(1) requires notification within 72 hours of awareness unless the breach is unlikely to result in a risk to rights and freedoms. **Article 33(5) requires every breach to be documented — including those assessed as not notifiable — together with the facts, effects and remedial action.** Neither the assessment nor the documentation exists. Even a well-founded conclusion of "no notifiable breach" would need to be written down, and it has not been.

This is a standalone regulatory failure, independent of the security posture that produced the incident.

#### Two determinations the alert record cannot support

**OD-08 — was MDC-2026-8891 red-team traffic?** The adversarial engagement ran **15–30 June 2026**; the alert falls inside the 90-day window ending at the package date. A pre-authorised engagement's traffic is normally excluded from alerting or annotated as such. The package does neither.

Both answers matter and both change the conclusion. If this was red-team traffic misclassified as an external attack, the detection is good news and the incident is not an incident. If it was genuine external traffic, an unknown actor was probing precisely the weaknesses the red team found, in the same period — and the forensic question above becomes urgent rather than prudent. **An incident responder asks "is this us?" first. The package gives no answer.**

**OD-09 — is one alert the whole 90 days?** The package describes "the last 90 days of alert summaries" and documents a single high-severity alert. Either the window genuinely contained one, or what was supplied is an excerpt. Without the alert-volume baseline and the medium and low severity records, this incident cannot be placed in context: a one-off and the visible instance of a sustained pattern look identical when you have one data point. An auditor asking to see the other eighty-nine days needs an answer.

#### Implications for current security posture

The incident demonstrates, in production, what Section 3 established from configuration review: **the perimeter is not a security boundary for this deployment.**

Network isolation (C1.2, Compliant) protects the subnet. The attack arrived through the legitimate public claim interface, which network isolation is not designed to stop. Edge rate-limiting (C1.1, Partial) responds to volume. The attack's effective component was quiet. The only control positioned to distinguish a hostile claim narrative from a legitimate one is LLM-aware application-layer inspection, and it is Non-Compliant.

**Detection is not prevention.** The alert fired, and 1,200 queries still completed. Until HP-02 deploys the input guardrail layer and HP-01 deploys output validation, a repeat of this incident produces the same result with the same telemetry — the deployment would watch it happen again.

---

### 4.2 CVE-2025-XXXX — patch-versus-retrain decision

#### Vulnerability profile

| Field | Value |
|---|---|
| CVE | CVE-2025-XXXX |
| Component | NeuralCore Library v3.1.x – v3.4.x, gradient computation and weight initialization modules |
| Type | Improper input validation in the gradient descent optimizer |
| CVSS v3.1 | 8.6 (High) · Scope Changed / Network · Attack Complexity High · Privileges Required None |
| **Exposure class** | **Training-time. No effect on inference-time behaviour in models trained without exploitation** |
| Attack surface | **AS-05** — the training data pipeline (Section 3) |

**Meridian's exposure.** ClaimsTriage v2.4 was trained six months ago on **NeuralCore v3.3.2** — inside the affected range. Training data was ingested from an external vendor over a network connection across a 72-hour window. Control **C3.1 (cryptographic ingestion hashing) is Non-Compliant** and has never existed, so no integrity evidence for that corpus is available.

The consequence is precise: **the absence of C3.1 is what converts an advisory into an unfalsifiable exposure.** Meridian cannot demonstrate the corpus was tampered with, and cannot demonstrate it was not. The production model must therefore be treated as potentially carrying an attacker-controlled backdoor.

#### Decision matrix, rescored

The supplied matrix was calibrated for an **inference-time** vulnerability. Two criteria change materially for a training-time exposure:

| Criterion | Supplied | Rescored | Why |
|---|---|---|---|
| Speed to protection, patch-only | 5.0 | **1.0** | Patching the inference environment closes no exposure that has already occurred — the scenario package states this directly. Time-to-protection is not 24 hours; it is never |
| Long-term robustness, patch-only | 2.0 | **1.0** | Patch-only does not "mask a deeper flaw." It leaves a model that cannot be shown free of a backdoor running as the sole screening layer, indefinitely |

| Strategy | Raw | **Weighted** | Gates failed |
|---|---|---|---|
| S1 — Environment patch only | 16 | 2.85 | **2** |
| S2 — Full retrain only | 13 | 2.60 | 0 |
| **S3 — Hybrid: patch + retrain** | **19** | **3.90** | **0** |

**Recommendation: Strategy 3 — Hybrid.** Immediate environment patch to NeuralCore v3.5.0 as a compensating control, plus a full model retrain on clean verified data.

Two methodological points, both fixing defects in the supplied matrix:

**The recommendation is now the highest-scoring strategy.** The supplied matrix recommended Hybrid at 4.05 while Patching Only scored 4.25, and reported a "Matrix Verification Score" of 2.3 — which was Retraining Only's score. Three numbers, none of them agreeing with the verdict.

**A mandatory gate test was added, because the weighted score alone gives the wrong answer.** Note that S1 outscores S2 (2.85 against 2.60) despite delivering no protection whatever. That is an artefact of weighting: speed, disruption and cost together carry 0.65, so a fast, cheap, non-disruptive strategy ranks well even when it achieves nothing. **S1 fails two knockout gates** — it does not remove the exposure that already occurred, and it leaves Meridian unable to demonstrate to a regulator that the production model is free of a backdoor. A strategy failing a gate is excluded regardless of rank. The matrix is a decision aid, not the decision.

#### Business impact

**The hybrid path.** Retraining takes 3–4 weeks. Throughout that window the deployment continues at full volume:

| | 21 days | 28 days |
|---|---|---|
| Claims processed | 88.2 million | 117.6 million |
| Transaction value | **$7.67 billion** | **$10.23 billion** |

These figures state **exposure scope, not expected loss.** No loss is asserted and none may be inferred — the exposure is unproven. The numbers state what is riding on the acceptance.

**The rollback alternative.** v2.3 operates at 90.4% TPR, **0.6 percentage points below** the 91.0% contractual floor — a breach effective at cutover, not at end of day. The 3.8-point drop from v2.4 produces **159,600 additional manual review claims per day**, accumulating to **3.35 million over a 21-day retrain window**. That exceeds manual capacity by a wide margin and compounds daily.

**The trade being made.** The hybrid path carries a large but **unproven and probabilistic** exposure. The rollback path carries a smaller but **certain and immediate** harm. Choosing hybrid is choosing probabilistic exposure over certain harm — a defensible trade, but only if recorded as a deliberate risk acceptance with compensating controls and a named owner, rather than presented as though it were cost-free.

> **Accepting owner:** Director of AI Production Infrastructure Operations, countersigned by the AI Governance Lead and the CISO. Reviewed weekly during the retrain window and immediately on any shadow-comparator divergence alert.

#### A correction to the inherited compensating control

The scenario package specifies that during the remediation window *"input layer filters will block all claimant submissions containing structural text variants associated with known backdoor triggers."*

**This control is not implementable as written.** CVE-2025-XXXX causes misclassification of inputs matching a *specific attacker-controlled pattern*. If the corpus was poisoned, that pattern was chosen by the attacker and is unknown to us; the advisory publishes no indicators of compromise. **A filter cannot block what cannot be enumerated.** Carried forward unexamined, it would have appeared in the package as a mitigation while providing nothing.

Four implementable controls replace it:

| | Control | Why it works where filtering does not |
|---|---|---|
| **CC-1** | **Shadow comparator (primary).** Run ClaimsTriage v2.3 in shadow against live traffic, scoring every claim and routing none. Alert on statistically significant divergence from v2.4 on identical input | v2.3 was compiled on NeuralCore v2.9.1, outside the affected range — it is clean of this vulnerability. A backdoor is invisible to a filter because its trigger cannot be named, but visible as **disagreement between a suspect model and a clean one on the same claim**. This detects what cannot be filtered, using an asset the deployment already holds |
| **CC-2** | Output distribution monitoring against the 90-day pre-incident baseline | A backdoor routing attacker-pattern claims to automated approval shows as an unexplained shift in the automated-approval share, even when the trigger is unknown |
| **CC-3** | Enhanced human review sampling on automated-approval routings, weighted to high-value claims | Converts an undetectable classifier defect into a detectable review-disagreement signal |
| **CC-4** | Patch the build environment to v3.5.0 within the 4-hour window; freeze training on the affected library; **implement C3.1 before the retrain corpus is assembled** | Closes forward exposure, and ensures the replacement model can be demonstrated clean in a way v2.4 never can |

CC-4's last clause matters beyond this CVE: without ingestion hashing, the retrained model inherits exactly the same unfalsifiable position, and the next advisory produces the same crisis.

#### Rollback conformance reference

A validated rollback procedure is in place: **MHP-OPSC-RECOVERY-v2**, which satisfies gate G5 of the decision matrix. Target ClaimsTriage v2.3 on NeuralCore v2.9.1, 45-minute containerised redeployment, decision owner the Director of AI Production Infrastructure Operations.

Four corrections were made to the inherited plan, one of them structural:

- It addressed **CVE-2026-38291**, a tokenizer vulnerability not applicable to this deployment. Rewritten against CVE-2025-XXXX.
- Target version corrected from "v1.4.2-stable" to **v2.3**; recovery time from 12 minutes to **45 minutes**.
- **The accuracy trigger was self-defeating.** It fired below 92.5% while the rollback target operates at 90.4% — executing the rollback would immediately re-satisfy the condition mandating it. Corrected to **89.5%**, set below the target's performance so reverting does not re-trigger.
- **A third trigger was added, and it is the only one that can detect this CVE's harm.** Triggers 1 and 2 detect degraded aggregate accuracy and known-signature exploitation. A targeted backdoor produces neither — it misclassifies only the attacker's pattern, leaving aggregate accuracy intact, and leaves no signature to match. **Trigger 3 fires on shadow-comparator divergence.** Without it, the deployment could be actively exploited throughout the retrain window with no trigger firing.

**The 24-hour limit does not survive contact with this CVE.** The inherited plan caps the rollback state at 24 hours "while patching is finalized" — framing that belongs to an inference-time vulnerability. If rollback is triggered here by confirmed exploitation or shadow divergence, **returning to v2.4 is precisely what must not happen**: v2.4 is the suspect artefact. The earliest safe return is the retrained replacement, 3–4 weeks away. A security-triggered rollback therefore persists for weeks at 159,600 additional manual claims per day. An escalation path is now defined in the plan rather than left to be improvised on the day it fails.

One caveat is stated rather than glossed: **v2.3 is clean of CVE-2025-XXXX, not clean generally.** C3.1 has never existed, so v2.3's corpus provenance is as unverified as v2.4's. Rollback removes a specific known exposure, not the class of exposure.

---

### 4.3 What Operational Security hands forward

| Output | Destination | Requirement |
|---|---|---|
| **CVE-2025-XXXX** | §5 Risk Disposition | **Named risk, mandatory.** The package fails the traceability check without it. Owner, target date, and the hybrid-path risk acceptance must all appear |
| Article 33 breach assessment gap | §5 Risk Disposition | A regulatory failure independent of the security posture. Also a GDPR finding, cross-referenced to §2 |
| MDC-2026-8891 findings | §5 Risk Disposition | Maps to AS-01 and AS-02; drives DID-03, DID-08, DID-09 |
| C3.1 ingestion hashing | §5 Risk Disposition | The control whose absence makes the CVE unfalsifiable, and the precondition for the retrained model being demonstrably clean |
| CC-1 shadow comparator | §5 Risk Disposition | Compensating control for the accepted risk, and rollback Trigger 3 |
| OD-08, OD-09 | §5 and §1 | Open determinations affecting the confidence of this section |

#### Two cross-links back into GDPR Compliance

**The Article 33 gap is a privacy finding, not only a security one.** It belongs in Section 2's rights-and-freedoms analysis as much as here, and it compounds DPIA GAP-05: the same 1,200 queries that may have breached Article 33 were attacking the routing that constitutes the Article 22(3) human review safeguard.

**The retrain interacts with the de-identification recommendation.** The clean-data retrain required by CC-4 is also the moment DP-SGD (C3.3, Non-Compliant) must be integrated, per §2.3. Running the retrain without it means running it twice — and each retrain consumes privacy budget under residual risk R-DEID-05, which the de-identification matrix records as failing silently.
