Claude Certified Architect (CCAR-F) practice exam for [[1 resumen]], targeting **Task Statement 5.5: Design human review workflows and confidence calibration** from the official `examguide.pdf`. Each question presents a real-world case; pick the correct option before revealing the answer.

> [!info] About official sample questions
> `examguide.pdf` has no sample question that maps 1:1 to Task Statement 5.5. The closest is Sample Question 3 (Task Statement 5.2), where the distractor "have the agent self-report a confidence score" fails because LLM self-reported confidence is poorly calibrated — the same principle tested here. All questions below are original, built on the Task Statement's "Knowledge of" and "Skills in" items, in the style of the official questions (scenario: *Structured Data Extraction*).

---

**Question 1**

Your structured data extraction system processes vendor invoices, handwritten expense receipts, scanned PDFs and international invoices. After three months, the dashboard reports 97% overall field accuracy. To cut review costs, the team proposes automating every extraction where the model's confidence exceeds 0.95. What is the critical risk in this proposal?

A. Automated extractions should always require human review regardless of confidence, so any automation is unsafe.

B. The 0.95 threshold is too low; it should be raised to 0.99 before automating.

C. Aggregate accuracy may mask poor performance on specific document types or fields, and the confidence scores need to be calibrated against labeled validation sets before being used as a threshold.

D. The model will become overconfident over time, so it needs regular retraining before automation is safe.

> [!question]- Reveal answer
> **Correct: C.** A 97% aggregate is dominated by the highest-volume document type and can hide segments far below acceptable accuracy (e.g., international invoices at ~45%). Raw confidence scores are relative, not absolute, so a 0.95 threshold has no validated meaning until calibrated per field. A rejects automation outright instead of validating it by segment. B keeps using uncalibrated raw scores — a stricter uncalibrated threshold doesn't fix the problem. D introduces an unrelated retraining concern and ignores both the aggregate-metrics and calibration issues. *(Adapted from the study guide's practice scenario for this topic.)*

---

**Question 2**

An invoice extraction pipeline reports these validation results:

| Document type | Share of volume | `totalAmount` accuracy |
|---|---|---|
| Standard invoices | 91% | 98.4% |
| Scanned PDFs | 5% | 69.8% |
| Handwritten receipts | 3% | 55.3% |
| International | 1% | 52.1% |
| **Aggregate** | 100% | **96.1%** |

Finance wants to stop reviewing `totalAmount` because "96% is good enough." What should you recommend?

A. Approve automation for `totalAmount` across all document types, since the aggregate exceeds 95%.

B. Keep human review for `totalAmount` across all document types until aggregate accuracy reaches 99%.

C. Retrain the model on more standard invoices to raise the aggregate further.

D. Consider reducing review only for segments that are consistently accurate (e.g., standard invoices, after threshold calibration and with stratified sampling in place), and keep human review for scanned, handwritten and international documents.

> [!question]- Reveal answer
> **Correct: D.** Accuracy must be analyzed by document type and field before reducing review; the aggregate is inflated by standard invoices (91% of volume) while three segments are far below acceptable levels. Automation should be scoped to segments that have been validated, calibrated, and are covered by ongoing sampling. A is the aggregate-metrics trap. B still decides with the aggregate — it would block automation for a segment that already performs well and doesn't address per-segment failures. C optimizes the already-strong segment and could raise the aggregate while leaving the weak segments untouched.

---

**Question 3**

Your extraction system outputs a confidence score per field. You run it against 2,000 labeled validation documents and find that when the model reports 0.90 on `invoiceDate` it is correct 94% of the time, but when it reports 0.90 on `totalAmount` it is correct only 82% of the time. The current routing logic auto-approves any field with confidence ≥ 0.90. What is the best change?

A. Raise the global threshold to 0.97 so that both fields clear the target accuracy.

B. Use the labeled validation set to build calibration curves per field (and document type), and set a separate calibrated threshold for each so that auto-approved items meet the target accuracy.

C. Ask the model in the prompt to be more conservative when reporting confidence on amount fields.

D. Drop field-level scores and use a single document-level confidence score to simplify routing.

> [!question]- Reveal answer
> **Correct: B.** Raw confidence is not calibrated: the same 0.90 maps to different real accuracies per field. Calibrating against labeled validation data and setting per-field thresholds is exactly what the Task Statement requires. A is still a single uncalibrated threshold — it over-restricts dates and may still be wrong for amounts in some document types. C tries to fix calibration via prompting; the output remains uncalibrated and unvalidated. D removes the granularity needed to route review attention to the fields that are actually unreliable.

---

**Question 4**

Six months after automating high-confidence extractions for standard invoices, a downstream accounts-payable reconciliation starts failing. Investigation shows a large new vendor sends invoices in DD/MM/YYYY format, and the model has been extracting `invoiceDate` as MM/DD/YYYY with 0.97 confidence. Your current QA process sends a random 5% of **low-confidence** extractions to an auditor each week. Which change would have caught this earliest?

A. Lower the auto-approval threshold so fewer extractions qualify as high confidence.

B. Increase the weekly audit from 5% to 20% of low-confidence extractions.

C. Implement stratified random sampling across document type, field, and confidence band — including the high-confidence automated extractions — and compare each stratum's error rate to its validated rate.

D. Add a JSON schema validator that rejects any date that isn't a valid calendar date.

> [!question]- Reveal answer
> **Correct: C.** Low-confidence items already go to human review; novel error patterns in high-confidence automated extractions only surface if those extractions are sampled. Stratified sampling by document type, field and confidence band measures ongoing error rates and detects new failure modes like an unfamiliar date format. A and B both keep looking only where errors are already expected. D can't catch the error: swapped day/month (e.g., 03/04) is still a valid date and passes schema validation.

---

**Question 5**

A team of 4 reviewers can handle about 600 field reviews per day, but the pipeline flags ~1,500 fields daily. Today the queue is processed in arrival order, and reviewers report spending most of their time confirming obviously-correct vendor names from standard invoices. Which redesign best uses the limited reviewer capacity?

A. Review a uniform random 40% of flagged fields so every document type gets equal attention.

B. Order the queue dynamically by uncertainty — low calibrated confidence, ambiguous or contradictory source documents, and segments with poor historical accuracy first — and have each reviewer pull the highest-uncertainty remaining item.

C. Assign each reviewer a fixed document type so they become specialists, and keep arrival order within each type.

D. Hire more reviewers until the queue can be processed fully in arrival order.

> [!question]- Reveal answer
> **Correct: B.** Reviewer capacity should be prioritized on the highest-uncertainty items, where human judgment adds the most value, and the prioritization should be dynamic — the queue reorders as new extractions arrive. A spreads capacity evenly, which is the anti-pattern: time is still wasted on easy items. C keeps chronological order within types and allocates capacity by type rather than by uncertainty. D addresses cost rather than the design flaw and ignores prioritization.

---

**Question 6**

An extraction for a purchase order returns:

```json
{
  "vendorName":  { "value": "Northwind Ltd", "confidence": 0.99, "ambiguous": false },
  "totalAmount": { "value": "4,820.00",      "confidence": 0.96, "ambiguous": true,
                   "alternatives": ["4,280.00"] },
  "dueDate":     { "value": "2025-07-01",    "confidence": 0.93, "ambiguous": false }
}
```

The source PDF shows one total in the summary box and a different total in the line-item footer. The calibrated auto-approval threshold for `totalAmount` on this document type is 0.94. How should `totalAmount` be routed?

A. Auto-approve it, because 0.96 is above the calibrated threshold of 0.94.

B. Auto-approve it but add it to the stratified sample, since it is above threshold.

C. Discard the extraction and re-run the model until it returns `ambiguous: false`.

D. Route it to human review with high priority, because the source document is contradictory regardless of the confidence score.

> [!question]- Reveal answer
> **Correct: D.** Extractions from ambiguous or contradictory source documents should be routed to human review and prioritized, even when the score clears the threshold — the contradiction is genuine uncertainty that the model cannot resolve. A and B trust the score while ignoring an explicit contradiction in the source; sampling is for ongoing measurement, not for handling known-ambiguous items. C hides the ambiguity by re-sampling until the model commits to one reading, which doesn't make the answer correct.

---

**Question 7** *(Select the TWO best answers)*

A team is preparing to reduce human review for their extraction pipeline. Which TWO steps must happen **before** they reduce human review for any segment?

A. Measure accuracy separately for each document type and field segment on a labeled validation set.

B. Confirm that aggregate accuracy across all documents exceeds 97%.

C. Calibrate field-level confidence scores against labeled validation data and set calibrated thresholds for automation versus review.

D. Instruct the model in its system prompt to only report confidence above 0.9 when it is certain.

E. Switch from field-level to document-level confidence so each document gets a single automation decision.

> [!question]- Reveal answer
> **Correct: A and C.** The validation-before-automation sequence is: measure accuracy by document type and field, calibrate confidence with labeled validation sets, set calibrated thresholds, implement stratified sampling, and only then reduce review on consistently accurate segments. B is the aggregate-metrics trap — the aggregate can be high while specific segments fail. D tries to calibrate through prompting; scores still need empirical calibration. E removes the field-level granularity required to route review attention to unreliable fields.

---

**Question 8**

You review the following QA code in an extraction pipeline:

```python
AUTO_THRESHOLD = 0.95

def route(field):
    if field["confidence"] >= AUTO_THRESHOLD:
        return "auto"
    return "review"

def weekly_audit(items):
    reviewed = [i for i in items if route(i) == "review"]
    return random.sample(reviewed, k=int(len(reviewed) * 0.05))
```

Which statement best describes the main weaknesses of this design?

A. It uses an uncalibrated global threshold for all fields and document types, and its audit samples only items that are already reviewed, so automated high-confidence extractions are never measured.

B. The threshold should be `>` instead of `>=` to avoid borderline cases being auto-approved.

C. The audit sample of 5% is too small; it should be at least 20% to be statistically meaningful.

D. `random.sample` is not cryptographically secure and should be replaced with `secrets.choice`.

> [!question]- Reveal answer
> **Correct: A.** Two Task Statement failures appear together: a single raw threshold assumes confidence means the same thing for every field and document type (it needs per-segment calibration on labeled validation data), and the audit only samples the review path, so novel error patterns in auto-approved extractions go undetected. The fix is calibrated per-field thresholds plus stratified random sampling that includes high-confidence automated items. B is a trivial boundary change. C increases sample size but keeps sampling the wrong population. D is irrelevant to measuring accuracy.

---
> [!tip] Sigue con este tema
> Vuelve a la teoría en [[1 resumen]], repasa con [[3 cuestionario]] y revisa el código en [[2 example]].
