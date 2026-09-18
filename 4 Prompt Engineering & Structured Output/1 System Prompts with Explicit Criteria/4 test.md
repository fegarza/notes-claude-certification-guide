Practice test for **Task Statement 4.1** — *Design prompts with explicit criteria to improve precision and reduce false positives* (Domain 4: Prompt Engineering & Structured Output). Based on `examguide.pdf`.

---

**Question 1**

Your team's automated code review system currently uses this system prompt: `"Review this pull request. Be conservative. Only flag issues you are highly confident about."` After two weeks in production, reviewers report that the flagged issues are inconsistent — the same recurring pattern (a missing null check) gets flagged in some PRs and ignored in others. What is the most effective change to the system prompt?

A. Lower the model's temperature to 0 so that identical patterns are always classified the same way.

B. Replace "be conservative" and "highly confident" with explicit categorical criteria defining which issue types to report and which to skip.

C. Add "only report issues you are 100% certain about" to further narrow the confidence threshold.

D. Instruct the model to review each PR three times and only report findings that appear in all three passes.

> [!question]- Show answer
> **Correct Answer: B.** Vague instructions like "be conservative" and "highly confident" provide no consistent, actionable decision boundary, which is exactly why the same pattern gets classified differently across runs. Explicit categorical criteria (what to flag, what to skip, with a verifiable trigger) produce consistent classification. Option A treats this as a randomness problem, but the root cause is ambiguous instructions, not sampling variance — lowering temperature won't fix an undefined criterion. Option C tightens the same flawed confidence-based approach instead of replacing it. Option D adds cost and latency without addressing the underlying ambiguity; a model applying an undefined criterion three times can still disagree with itself three different ways.

---

**Question 2**

A CI/CD code review pipeline reports findings in three categories: `bugs`, `security`, and `documentation mismatch`. After a false-positive audit, `documentation mismatch` findings are wrong 40% of the time, while `bugs` and `security` are accurate 95%+ of the time. Developers have started ignoring the entire review report, including accurate security findings. What is the most effective fix?

A. Temporarily disable the `documentation mismatch` category while refining its criteria with concrete code examples, then re-enable it once its false positive rate improves.

B. Add "only report high-confidence documentation issues" to the system prompt so the model filters its own weaker findings before they reach developers.

C. Keep all three categories active, but add a disclaimer to the report noting that `documentation mismatch` findings may be less reliable than the others.

D. Increase the sampling temperature to generate more varied documentation findings, then discard any finding that doesn't appear in at least two of three runs.

> [!question]- Show answer
> **Correct Answer: A.** High false positive rates in one category destroy developer trust in *all* categories — trust does not compartmentalize. Temporarily disabling the problematic category restores immediate trust in the accurate categories (`bugs`, `security`) while the team iterates on `documentation mismatch` using concrete code examples, per the recommended workflow. Option B relies on confidence-based filtering, and LLM self-reported confidence is poorly calibrated — it would not reliably suppress the false positives. Option C leaves the noisy category active, which does nothing to stop it from eroding trust in the whole report; a disclaimer doesn't change developer behavior once trust is already broken. Option D adds cost without addressing the root cause: an underspecified criterion for what counts as a documentation mismatch.

---

**Question 3**

You are writing severity definitions for a code review prompt. Your first draft reads: `"Critical: issues that could cause system failures or data loss. Minor: issues that affect readability but not functionality."` Reviewers report inconsistent severity assignment on similar issues. What should you change?

A. Add more adjectives to each severity tier (e.g., "extremely critical," "very minor") to sharpen the distinction.

B. Replace the prose descriptions with concrete code examples illustrating an actual pattern at each severity level.

C. Remove severity levels entirely and have the model report all findings at a single priority.

D. Ask the model to assign a numeric severity score from 1-10 instead of a category label.

> [!question]- Show answer
> **Correct Answer: B.** Prose descriptions like "could cause system failures" require the model to interpret an abstract standard, which produces inconsistent classification across invocations. Concrete code examples for each severity tier (e.g., an unsanitised SQL query for Critical, an inconsistent variable name for Minor) give the model an actual pattern to match against, eliminating that ambiguity. Option A adds more subjective, uncalibrated language on top of an already-ambiguous standard — it doesn't fix the core problem. Option C removes useful signal instead of fixing the underlying criteria. Option D swaps one uncalibrated, self-reported judgment (severity label) for another (a numeric score) without adding any concrete reference point.

---

**Question 4** *(Select the two best answers)*

Which two statements correctly describe why confidence-based filtering should not be the primary mechanism for reducing false positives in a review prompt?

A. LLM self-reported confidence is poorly calibrated — the model can express high certainty about an incorrect finding and low certainty about a correct one.

B. Confidence-based filtering is only useful for routing already-valid findings (e.g., to human review), and should be applied after explicit criteria have already determined validity.

C. Confidence-based filtering is slower to compute than categorical filtering, making it impractical for CI/CD pipelines.

D. Confidence scores cannot be represented in structured JSON output, making them incompatible with tool-use-based review pipelines.

> [!question]- Show answer
> **Correct Answer: A and B.** LLM self-reported confidence is poorly calibrated, so filtering by it does not reliably separate true findings from false ones (A). The correct hierarchy is explicit criteria first, confidence-based routing second — confidence is legitimately useful only after criteria have already established what counts as a valid finding (B). Option C is false: computational cost is not the reason confidence-based filtering is discouraged. Option D is false: confidence scores can be included as an ordinary field in structured output; that's not the limiting factor here.

---

> [!tip] Repasa la teoría en [[1 resumen]], memoriza con [[3 cuestionario]] y practica el código en [[2 example]]
