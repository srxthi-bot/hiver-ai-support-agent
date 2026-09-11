# Hiver AI Support Agent — AmazonHelp

An evaluation-focused AI customer-support prototype built for the **Hiver SDE Intern Take-Home Assignment** using real customer-support conversations from Twitter/X.

The system demonstrates three core capabilities:

1. **Intent Classification** — classifies incoming customer messages into a compact support-intent taxonomy.
2. **Grounded Reply Drafting** — retrieves historical AmazonHelp interactions and uses observed support resolutions as evidence for drafting a response.
3. **Escalation Decision** — determines whether a request can be auto-handled or should be escalated to a human, with an explicit reason.

The project emphasizes **evaluation, leakage prevention, grounding, safety, and failure analysis** rather than optimizing a single headline metric.

---

## Selected Brand

### AmazonHelp

The Customer Support on Twitter dataset was analyzed to identify a brand with sufficient interaction volume and diverse customer-support issues.

After preprocessing and constructing customer-to-support interaction pairs, the selected AmazonHelp subset contained:

**168,814 historical customer-support interactions**

AmazonHelp was selected because it provided:

- Large interaction volume
- Broad support issue coverage
- Repeated resolution patterns
- Sufficient historical responses for retrieval-based grounding

---

# 1. Problem Framing

The objective is to build a measurable prototype for AI-assisted customer support.

For each incoming customer message, the system follows this pipeline:

```text
Customer Message
       |
       v
Intent Classification
       |
       v
Historical Resolution Retrieval
       |
       v
Grounded Reply Draft
       |
       v
Escalation Policy
       |
       +---------------------+
       |                     |
       v                     v
  Auto-handle         Human Escalation
```

The system intentionally separates:

**Classification → Evidence Retrieval → Reply Drafting → Escalation**

This makes each stage independently measurable.

## What This Project Intentionally Does NOT Attempt to Build

This is an evaluation-focused prototype, not a production Amazon support system.

It does not attempt to provide:

- Direct access to customer accounts
- Real-time order information
- Payment processing
- Autonomous refunds
- Autonomous cancellations
- Authentication or identity verification
- Access to private customer information
- Production policy enforcement
- Full replacement of human support agents

High-risk or customer-specific cases should remain eligible for human review.

---

## 1.1 Intent Taxonomy

A compact 11-intent taxonomy was defined from the observed AmazonHelp conversations.

| ID | Intent |
|---|---|
| I01 | Order / Delivery |
| I02 | Missing Package |
| I03 | Return |
| I04 | Refund |
| I05 | Wrong / Damaged Item |
| I06 | Payment / Billing |
| I07 | Account |
| I08 | Prime Subscription |
| I09 | Cancellation / Modification |
| I10 | General Information |
| I11 | Other / Unclear |

The `I11 Other / Unclear` category is intentionally retained for ambiguous, multi-intent, or insufficiently specified customer messages rather than forcing them into an incorrect specific category.

---

# 2. Dataset and Golden Set

## Historical Dataset

The project uses the **Customer Support on Twitter** dataset and constructs customer-to-support interaction pairs for AmazonHelp.

The resulting AmazonHelp corpus contains:

**168,814 historical interactions**

These historical interactions are used for:

- Understanding support-intent patterns
- Training the intent classifier
- Retrieving similar historical customer issues
- Grounding draft responses

---

## Golden Set

A manually labelled evaluation set of **200 examples** was created.

Each golden-set example contains:

- Customer message
- Intent label
- Escalation decision
- Escalation reason

### Golden-set validation

| Check | Result |
|---|---:|
| Golden examples | 200 |
| Missing intent labels | 0 |
| Missing escalation labels | 0 |
| Missing escalation reasons | 0 |
| Duplicate IDs | 0 |

### Intent distribution

| Intent | Count |
|---|---:|
| I01 Order / Delivery | 52 |
| I02 Missing Package | 10 |
| I03 Return | 6 |
| I04 Refund | 5 |
| I05 Wrong / Damaged Item | 13 |
| I06 Payment / Billing | 9 |
| I07 Account | 12 |
| I08 Prime Subscription | 10 |
| I09 Cancellation / Modification | 5 |
| I10 General Information | 26 |
| I11 Other / Unclear | 52 |

### Escalation distribution

| Decision | Count |
|---|---:|
| Auto-handle | 108 |
| Human escalation | 92 |

The golden set was designed as a compact evaluation harness rather than a claim of complete coverage of all AmazonHelp support traffic.

---

# 3. Evaluation Methodology

The evaluation intentionally separates:

1. **Intent classification**
2. **Historical evidence retrieval**
3. **Reply quality**
4. **Escalation decisions**

This prevents a strong result in one stage from hiding failures in another.

For intent classification, a **5-fold out-of-fold (OOF)** evaluation was used to reduce training/evaluation leakage.

For retrieval, historical examples belonging to the same customer message were excluded during evaluation.

For reply quality, a human-reviewed sample of 20 examples was evaluated across four dimensions.

---

# 4. Intent Classification

## Selected Model

The final intent classifier is:

**Word-level TF-IDF + Logistic Regression**

The model was selected using weighted F1 on out-of-fold predictions.

### Model comparison

| Model | OOF Accuracy | Weighted F1 |
|---|---:|---:|
| Word TF-IDF + Logistic Regression | **43.50%** | **39.54%** |
| Word + Character TF-IDF + Logistic Regression | 44.50% | 39.04% |
| Word TF-IDF + Linear SVM | 42.00% | 36.92% |

Although the Word + Character model achieved slightly higher accuracy, Word TF-IDF + Logistic Regression achieved the best weighted F1 and was therefore selected.

---

## Baselines

Two baselines were used.

### Majority-class baseline

Always predicts the most frequent intent, `I11 Other / Unclear`.

- Accuracy: **26.00%**
- Weighted F1: **10.73%**

### Simple TF-IDF baseline

Initial TF-IDF + Logistic Regression baseline:

- Accuracy: **32.00%**
- Weighted F1: **29.42%**

The final OOF evaluation reaches:

- Accuracy: **43.50%**
- Weighted F1: **39.54%**

The classifier therefore provides a meaningful improvement over both the majority-class baseline and the initial simple baseline, while still leaving substantial room for improvement.

---

# 5. Historical Evidence Retrieval

The response-generation component retrieves similar historical AmazonHelp interactions.

The retrieval pipeline uses:

- Customer message representation
- TF-IDF similarity
- Historical customer-support interactions
- Similarity thresholding
- Leakage prevention

The retrieved historical support response acts as **evidence**, rather than treating the model as an unrestricted generative system.

---

## Leakage Prevention

An important issue was discovered during evaluation.

All 200 golden-set customer IDs were present in the historical corpus, and some golden examples had duplicate historical rows.

A naive retrieval implementation could therefore retrieve the exact evaluation interaction and artificially inflate performance.

To prevent this, the evaluation excludes the same `customer_tweet_id` from the retrieval candidate pool.

This makes the retrieval evaluation leakage-safe.

---

## Leakage-Safe Evidence Availability

At a cosine-similarity threshold of `0.20`:

- Evidence availability: **98.50%**
- Mean similarity: **0.4386**
- Median similarity: **0.3715**
- Minimum similarity: **0.0000**

The 98.50% figure should be interpreted as **leakage-safe evidence availability at the chosen threshold**, not as proof that every retrieved response is correct.

---

# 6. Reply Drafting

The prototype uses historical AmazonHelp support responses as grounding evidence.

The current implementation prioritizes:

- Traceability
- Historical grounding
- Avoiding unsupported policy claims
- Reusing observed resolution patterns

The system intentionally avoids inventing:

- Refund amounts
- Delivery dates
- Account information
- Private customer details
- Unsupported policies
- Unverified links

The current prototype uses the nearest historical support response as the draft response.

A production-quality version would extend this to **top-k retrieval followed by controlled synthesis**, while requiring every substantive claim to remain grounded in retrieved evidence.

---

# 7. Escalation Policy

The escalation component uses an explicit policy rather than relying only on model confidence.

A request is considered for human escalation when one or more of the following conditions apply:

1. The customer explicitly requests a human.
2. The issue involves sensitive account or payment information.
3. The product issue is severe.
4. The interaction indicates delivery failure.
5. The customer appears to have a repeated unresolved issue.
6. The customer makes a serious complaint.
7. No sufficiently relevant historical evidence is retrieved.
8. Retrieval similarity falls below the safety threshold of `0.20`.

Otherwise, the system may classify the interaction as eligible for auto-handling.

The escalation policy was intentionally frozen as **Escalation V2** before final evaluation.

---

# 8. Escalation Results

Evaluation was performed on the 200-example golden set.

| Metric | Result |
|---|---:|
| Accuracy | **69.50%** |
| Precision | **82.98%** |
| Recall | **42.39%** |
| F1 | **56.12%** |
| Auto-handle rate | **76.50%** |
| Missed escalations | **53** |
| Unnecessary escalations | **8** |

The high precision indicates that when the policy escalates a case, it is usually justified.

However, recall is substantially lower than precision. The major weakness is therefore **under-escalation**, especially for difficult cases that appear superficially similar to normal support requests.

---

# 9. Failure Analysis

The project explicitly analyzes failure cases rather than reporting only aggregate metrics.

## Failure Mode 1 — Order / Delivery Confusion

**Observed:** 19 missed escalation cases were associated with `I01 Order / Delivery`.

**Hypothesis:** Delivery-related conversations frequently contain short or ambiguous messages that resemble routine tracking questions even when the underlying issue requires human intervention.

**Improvement:** Add finer-grained delivery states such as:

- Delayed
- Lost
- Marked delivered but not received
- Tracking unavailable
- Delivery dispute

---

## Failure Mode 2 — Other / Unclear Intent

**Observed:** 8 missed escalation cases involved `I11 Other / Unclear`.

**Hypothesis:** The catch-all class absorbs ambiguous messages and therefore provides limited semantic separation.

**Improvement:** Introduce confidence-aware handling and consider splitting high-frequency ambiguous patterns into dedicated intents.

---

## Failure Mode 3 — Missing Package Cases

**Observed:** 7 missed escalation cases involved `I02 Missing Package`.

**Hypothesis:** Missing-package messages can resemble ordinary delivery-status questions.

**Improvement:** Add explicit rules/features for phrases indicating non-receipt, delivered-but-not-received status, and repeated delivery complaints.

---

## Failure Mode 4 — Account and Subscription Ambiguity

Missed escalation cases were also observed in:

- `I07 Account`
- `I08 Prime Subscription`
- `I09 Cancellation / Modification`

**Hypothesis:** These categories often require customer-specific account context that is unavailable in a public historical dataset.

**Improvement:** Escalate more aggressively when account-specific action is implied.

---

## Failure Mode 5 — Retrieval Similarity Is Not a Safety Guarantee

The missed-escalation cases did not simply have poor retrieval.

For missed escalation cases:

- Mean retrieval similarity: **0.355**
- Median retrieval similarity: **0.340**
- Maximum retrieval similarity: **0.746**

This demonstrates that a high similarity score does not necessarily mean the retrieved historical interaction is appropriate for the current customer.

**Improvement:** Evaluate retrieval using semantic correctness and resolution compatibility, not similarity alone.

---

# 10. Human Reply-Quality Evaluation

A human-reviewed sample of **20 replies** was scored on a 1–5 scale.

| Dimension | Mean Score |
|---|---:|
| Groundedness | **5.00 / 5** |
| Resolution Correctness | **4.00 / 5** |
| Relevance | **4.55 / 5** |
| Helpfulness | **3.70 / 5** |
| Overall | **4.31 / 5** |

The strongest result was groundedness, reflecting the retrieval-first design.

Helpfulness was lower than groundedness, indicating that simply copying a historically grounded response does not always produce the most useful response for a new customer.

This motivates the next iteration: **retrieve multiple relevant examples and synthesize a concise response while preserving evidence constraints.**

---

# 11. LLM-as-Judge Validation

An LLM-as-judge validation sample was prepared using 20 examples.

The intended judge evaluates:

- Groundedness
- Resolution correctness
- Relevance
- Helpfulness

The validation artifact contains both human scores and LLM-generated score fields.

However, the final notebook execution did not complete the LLM scoring pass reliably enough to report a defensible human–LLM agreement statistic.

Therefore, **no agreement number is claimed in this submission**.

This is an explicit limitation rather than an inferred or fabricated result.

The next iteration should complete the judge pass and report:

- Per-dimension agreement
- Exact agreement rate
- Spearman correlation
- Mean absolute score difference
- Disagreement examples

---

# 12. What Is Misleading About My Headline Number?

The most potentially misleading headline number is the **98.50% leakage-safe evidence availability** figure.

It does **not** mean:

- 98.50% of responses are correct
- 98.50% of customers are successfully resolved
- 98.50% of retrieved answers are relevant
- 98.50% of cases can be safely automated

It only means that, after preventing same-example leakage, a historical interaction above the selected cosine-similarity threshold was available for most evaluated messages.

Similarly, the **69.50% escalation accuracy** should not be interpreted as proof that the system is safe to deploy autonomously.

The more important safety signal is the **42.39% escalation recall**, which shows that the current policy still misses a substantial number of cases requiring escalation.

The project therefore treats headline metrics as diagnostics rather than deployment claims.

---

# 13. Decision Log

The following non-obvious decisions were made during development.

| # | Decision | Reason |
|---:|---|---|
| 1 | Select AmazonHelp | Large and diverse historical interaction volume |
| 2 | Construct customer-to-support pairs | Provides usable resolution evidence |
| 3 | Use 11 intents | Balances coverage and label consistency |
| 4 | Retain Other / Unclear | Avoids forcing ambiguous messages into incorrect classes |
| 5 | Use a 200-example golden set | Practical manual evaluation size |
| 6 | Use OOF intent evaluation | Reduces training/evaluation leakage |
| 7 | Select Logistic Regression | Best weighted F1 among tested models |
| 8 | Use historical retrieval | Provides grounded support evidence |
| 9 | Exclude same customer ID during retrieval | Prevents self-match leakage |
| 10 | Escalate explicit human requests | Safety-first behavior |
| 11 | Escalate sensitive account/payment cases | Public dataset lacks account context |
| 12 | Escalate serious/repeated issues | Protects against under-handling difficult cases |
| 13 | Use retrieval threshold 0.20 | Avoid unsupported responses when evidence is weak |
| 14 | Freeze Escalation V2 before evaluation | Prevents iterative tuning on the final test set |
| 15 | Include human reply-quality evaluation | Measures quality beyond classification metrics |

---

# 14. Final Results

| Component | Headline Result |
|---|---:|
| Golden set | **200 examples** |
| Intent model | **TF-IDF + Logistic Regression** |
| Intent OOF accuracy | **43.50%** |
| Intent OOF weighted F1 | **39.54%** |
| Leakage-safe evidence availability | **98.50%** |
| Escalation accuracy | **69.50%** |
| Escalation precision | **82.98%** |
| Escalation recall | **42.39%** |
| Escalation F1 | **56.12%** |
| Auto-handle rate | **76.50%** |
| Human reply quality | **4.31 / 5** |

---

# 15. Reproducibility

The repository contains the notebook, evaluation artifacts, golden set, report, and supporting result files.

### Environment

```text
Python 3.x
pandas
numpy
scikit-learn
matplotlib
seaborn
kagglehub
```

Install dependencies with:

```bash
pip install -r requirements.txt
```

The main notebook contains the complete experimental workflow:

1. Dataset loading
2. Preprocessing
3. AmazonHelp selection
4. Customer-support pair construction
5. Intent taxonomy
6. Golden-set evaluation
7. Intent model comparison
8. Out-of-fold evaluation
9. Historical evidence retrieval
10. Leakage-safe retrieval evaluation
11. Reply drafting
12. Escalation policy
13. Failure analysis
14. Human reply-quality evaluation

The project uses a subsample of the full dataset for evaluation, as expected for a take-home assignment.

---

# 16. Repository Structure

```text
hiver-ai-support-agent/
│
├── README.md
├── Hiver_SDE_Intern_Take_Home_—_AmazonHelp_AI_Support_Agent.ipynb
├── Hiver_AmazonHelp_Report.pdf
│
├── amazonhelp_golden_set.csv
├── final_metrics.json
├── oof_intent_results.csv
├── intent_model_comparison.csv
├── escalation_v2_results.csv
├── escalation_missed_cases.csv
├── leakage_safe_retrieval_results.csv
├── decision_log.csv
│
└── requirements.txt
```

---

# 17. One-Week Improvement Plan

If given another week, the next iteration would focus on improving **safety, intent quality, and reply usefulness** rather than simply increasing the auto-handling rate.

## Day 1 — Improve Intent Taxonomy

Review confusion patterns and split high-frequency ambiguous categories.

Potential additions:

- Delivery delayed
- Delivered but not received
- Tracking issue
- Account access problem
- Payment failure

---

## Day 2 — Improve Retrieval

Move from single-nearest-neighbor retrieval to top-k retrieval.

Evaluate:

- Recall@k
- MRR
- Semantic relevance
- Resolution compatibility

---

## Day 3 — Grounded Response Synthesis

Replace direct response copying with controlled synthesis.

The response generator should:

- Retrieve multiple examples
- Extract common resolution patterns
- Produce a concise response
- Avoid unsupported claims
- Preserve uncertainty when evidence is insufficient

---

## Day 4 — Improve Escalation

Optimize for **recall on high-risk cases**, not raw accuracy.

Introduce explicit escalation categories:

- Account-sensitive
- Payment-sensitive
- Delivery failure
- Repeated unresolved
- Serious complaint
- Policy-sensitive

---

## Day 5 — Complete LLM-as-Judge Validation

Run the full judge validation sample and compare LLM scores against human scores.

Report:

- Exact agreement
- Spearman correlation
- Mean absolute difference
- Per-dimension agreement
- Disagreement examples

---

## Day 6 — Robustness Testing

Test the system against:

- Typos
- Short messages
- Multiple intents
- Repeated complaints
- Contradictory messages
- Low-evidence cases
- Adversarial or ambiguous phrasing

---

## Day 7 — Final Evaluation

Freeze all policies and models before evaluation.

Produce:

- Final golden-set results
- Failure analysis
- Retrieval evaluation
- Reply-quality evaluation
- Escalation analysis
- Reproducible report

---

# 18. Limitations

This prototype has several important limitations.

### Dataset limitations

The dataset consists of historical Twitter/X support interactions and does not provide live customer-account context.

### Intent limitations

The 11-intent taxonomy is designed for this dataset and may not generalize directly to other brands.

### Retrieval limitations

Similarity does not guarantee that a retrieved response is correct for the current customer.

### Reply limitations

The current prototype relies heavily on historical support responses and does not yet perform robust multi-example response synthesis.

### Escalation limitations

The current escalation policy has high precision but relatively low recall, resulting in missed escalation cases.

### LLM judge limitation

The human–LLM judge agreement analysis was not completed sufficiently to report a defensible agreement statistic.

### Production limitations

The system should not be considered production-ready because it lacks:

- Authentication
- Live order/account APIs
- Real-time policy verification
- Transaction execution
- Privacy controls
- Production monitoring
- Human-in-the-loop infrastructure

---

# 19. Conclusion

This project demonstrates an evaluation-first approach to building an AI-assisted customer-support agent.

The system combines:

**Intent Classification + Historical Evidence Retrieval + Grounded Reply Drafting + Explicit Escalation**

The main engineering focus was not simply producing a chatbot, but making the system **measurable and auditable**.

Key findings include:

- Intent classification remains challenging on noisy customer-support text.
- Leakage prevention materially changes how retrieval results should be interpreted.
- Historical responses provide strong grounding but do not guarantee helpfulness.
- Escalation precision is strong, while escalation recall remains a major weakness.
- Similarity scores alone are not sufficient as a safety signal.
- Human evaluation reveals quality dimensions that classification metrics cannot capture.

The most important next step is therefore not maximizing the auto-handle rate. It is improving **high-risk case detection, retrieval quality, and grounded response synthesis** while maintaining transparent evaluation.

---

**Sruthi Ramesh Shinde**

GitHub: `srxthi-bot`

LinkedIn: `sruthishinde-2885srs`
