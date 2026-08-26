# Step 3 — Prioritization Analysis and FAIR Scenario Selection

**Initiative:** Contoso Financial Service GenAI Knowledge Assistant (Azure OpenAI)
**Source:** AI Risk Register, R-01 to R-11 (Step 2). No risk is introduced here that is not already in the register.
**Purpose:** Rank the top five risks, document the overlay factors that move them away from raw score order, and select the single scenario carried into FAIR quantification.

---

## 1. Raw Score Extract

The five highest inherent scores, before overlay:

| Raw rank | Risk ID | Risk | Domain | L | I | Inherent |
|---|---|---|---|---|---|---|
| 1= | R-01 | Ingestion boundary failure — excluded content enters the index | Data | 4 | 4 | **16** |
| 1= | R-04 | Ungrounded answer on a regulated procedure | Model | 4 | 4 | **16** |
| 1= | R-09 | Staff over-reliance on assistant output | Usage | 4 | 4 | **16** |
| 4= | R-02 | Permission-trimming failure — above-entitlement retrieval | Data | 3 | 5 | **15** |
| 4= | R-05 | Prompt injection at the inference endpoint | Model | 3 | 5 | **15** |

Raw scoring produces a three-way tie at 16 and a two-way tie at 15 and cannot, on its own, tell the committee what to act on first. Overlay factors resolve the ties and, in two cases, reorder the list.

**Immediately below the line:** R-03, R-08 and R-10 all score 12. R-10 (auditability and non-repudiation) is flagged as a watch item — supervisory expectations around reconstructing AI-influenced decisions are tightening, and a shift in that expectation would move it above the line without any change to the system.

---

## 2. Overlay Factors

Five factors are applied. Each is rated High / Medium / Low, and each is defined so the rating can be challenged.

| Factor | Definition | Why it matters to this committee |
|---|---|---|
| **Regulatory sensitivity (RS)** | Does realisation create a notifiable, reportable, or supervisory-attention event? | Determines whether the risk becomes the regulator's business as well as Contoso's |
| **External harm reach (EH)** | Does the harm reach customers or become externally visible? | Separates internal control failures from conduct exposure |
| **Detection lag (DL)** | Time between occurrence and reliable detection. **High = long lag = worse** | A risk with no natural detection signal accumulates silently |
| **Control readiness gap (CR)** | How much mitigation is still to be built, versus already running in production | Determines what the committee must fund or condition now |
| **Interaction effect (IE)** | Does the risk enable, bound, or amplify another register risk? | Distinguishes generative risks from dependent ones and prevents double-counting |

### Overlay assessment

| Risk | RS | EH | DL | CR | IE | Nature of interaction |
|---|---|---|---|---|---|---|
| R-01 | High | Medium | **High** | **High** | High | **Enables** the R-05 loss path |
| R-02 | High | Low | **High** | **High** | High | **Bounds** R-05's blast radius |
| R-05 | High | Medium | Medium | Medium | High | **Depends on** R-01; bounded by R-02 |
| R-04 | Medium | **High** | Medium | Low | Medium | Amplified by R-09 |
| R-09 | Medium | **High** | High | Medium | High | **Amplifies** R-04 and R-06 |

---

## 3. Adjusted Priority Ranking

| Priority | Risk ID | Raw | Movement | Adjusted rationale |
|---|---|---|---|---|
| **1** | **R-01** | 16 | — | Holds first place on overlay as well as raw score. It is the only risk that is simultaneously regulator-notifiable, silent in accumulation, and almost entirely unbuilt — the existing controls inventory carries input regex validation, which is not a data loss prevention capability. It is also the precondition that makes Contoso's largest modelled financial exposure possible. |
| **2** | **R-02** | 15 | **▲ 3** | Carries the highest single impact rating in the register (5). It is the only risk with **no natural detection signal** — a user shown more than their entitlement does not complain, and nothing in normal operation surfaces the drift. It is also the containment control that caps R-05: if entitlement filtering holds, a successful injection cannot exceed the caller's own access. Its failure therefore compounds rather than stands alone. |
| **3** | **R-05** | 15 | **▲ 2** | A deliberate-adversary risk with immediate exploitation and a published, low-barrier technique base. Chained with R-01 it constitutes the organisation's largest modelled financial exposure, and it is the risk on which the CISO will be questioned most directly. Ranked below R-02 because permission-trimmed retrieval is what bounds its consequence — fixing the container matters more than fixing one route into it. |
| **4** | **R-04** | 16 | **▼ 2** | Reaches customers directly, which is why it stays in the top five. Ranked below the three above because its harm is per-answer and bounded rather than systemic, existing quality assurance and four-eyes checks provide independent detection, and its mitigations — temperature, groundedness threshold, abstention, restricted-topic routing — are configuration rather than build. Lower cost to close, lower claim on committee attention now. |
| **5** | **R-09** | 16 | **▼ 2** | Ranked fifth **at the inherent stage only**, because it is a dependent amplifier rather than an independent source of loss: it converts R-04 and R-06 errors into customer harm rather than generating harm of its own. Ranking it higher would count the same loss event twice. **This ordering reverses after controls** — R-09 carries the highest residual score in the register (9) and becomes the leading residual concern in Section 6. |

### Two observations the committee should take from this ranking

**The top three are all Data-and-access risks, not model-behaviour risks.** The instinctive concern with a GenAI assistant is that the model will say something wrong. The analysis says the more material exposure is what the model is allowed to see and who is allowed to see it back. That should shape where pre-go-live effort is spent.

**Raw scoring and overlay scoring disagree in both directions.** Two 15s outrank three 16s. This is recorded openly so the committee can challenge the overlay logic rather than inherit a ranking whose derivation is invisible.

---

## 4. FAIR Scenario Selection

### Selected scenario

> **Confidentiality breach of restricted content extracted from the knowledge assistant through prompt injection against the Azure OpenAI inference endpoint.**
>
> **Primary risk: R-05.** **Enabling precondition: R-01.**

### Why this scenario

| Criterion | Assessment |
|---|---|
| **Discrete loss event** | FAIR requires a countable event with a frequency. A prompt-injection extraction is a bounded incident with a start, a scope, and an end — unlike a diffuse behavioural risk. |
| **Relatable financial impact** | Loss lands in categories a financial services executive prices routinely: incident response and forensics, regulatory penalty, legal cost, notification and remediation. |
| **Defensible loss data** | Published breach-cost benchmarks and regulatory penalty ranges give both loss magnitude components an external anchor, so the estimate can survive challenge. |
| **Single identifiable attack path** | One threat actor, one mechanism, one endpoint. Frequency can be reasoned about rather than guessed. |
| **Directly answers the committee's question** | The committee explicitly asked how the inference endpoint is protected. Quantifying the endpoint's principal attack produces the number that question implies. |

### Why the scenario is chained rather than single-risk

The supplied FAIR calculator carries a secondary loss band reaching $11,000,000 in regulatory and legal exposure. **On R-05 alone, that figure is not defensible.** The scope record excludes customer personal data from the index (X-2), so a prompt injection against a correctly-bounded corpus extracts internal policy and procedural content — commercially sensitive, but not a notifiable personal-data breach and nowhere near a fine of that scale.

The figure becomes credible only when R-01 has already failed: excluded content is present in the index despite the boundary rule, and injection is the mechanism that extracts it. Modelling the two together is therefore not a convenience — it is what makes the loss magnitudes honest.

This has a governance consequence worth stating to the committee: **the largest single lever on Contoso's modelled financial exposure is R-01's ingestion controls, not R-05's endpoint defences.** If nothing sensitive is in the index, the injection succeeds and the loss is modest.

### Candidates considered and rejected

| Risk | Why not quantified |
|---|---|
| **R-01** alone | A control failure, not a loss event. It creates exposure but produces no loss until something extracts the data. FAIR needs an event to count; R-01 supplies a condition. It enters the model as the precondition instead. |
| **R-02** alone | Same structural problem — an access failure whose loss depends entirely on what the over-entitled user subsequently does. The consequence is also intra-organisational and resists external cost benchmarking. |
| **R-04** | The loss event is a single wrong answer: very high frequency, very low individual magnitude, with a long tail where one answer causes a large remediation. FAIR handles that distribution poorly at Contoso's current data maturity, and the resulting estimate would be indefensible under challenge — precisely the failure mode the method warns against. |
| **R-09** | Behavioural, with no discrete event boundary. There is no defensible way to state how often "over-reliance occurred." Managed through control design and assurance, not quantification. |

---

## 5. Carried Into Step 4

| Item | Value |
|---|---|
| FAIR scenario label | Confidentiality breach of restricted content via prompt injection against the Azure OpenAI inference endpoint |
| Primary risk ID | R-05 |
| Enabling precondition | R-01 |
| Loss form — primary | Incident response, forensics, containment |
| Loss form — secondary | Regulatory penalty, legal cost, notification and remediation |
| Supplied bounds to be validated | LEF 0.10 / 0.25 / 0.50 per year; ALE $50,000 / $625,000 / $6,000,000 |
| Priority risks that must reappear downstream | R-01, R-02, R-05, R-04, R-09 |

---

**Status:** Awaiting confirmation. On approval, Step 4 populates the FAIR calculator against this scenario.
