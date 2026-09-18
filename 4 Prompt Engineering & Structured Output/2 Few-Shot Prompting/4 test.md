Practice test for **Task Statement 4.2** — *Apply few-shot prompting to improve output consistency and quality* (Domain 4: Prompt Engineering & Structured Output). Based on `examguide.pdf`.

---

**Question 1**

A document extraction pipeline pulls structured fields from vendor contracts using a detailed system prompt with explicit field descriptions and formatting rules. After a month in production, the `payment_terms` field is consistently well-formatted when present, but comes back `null` on roughly 20% of contracts — manual review confirms the payment terms are stated in those contracts, just phrased as narrative prose ("payment due within thirty days of invoice") instead of the expected structured phrasing ("Net 30"). What is the most effective next step?

A. Rewrite the field description in the system prompt with more precise language about what counts as payment terms.

B. Add few-shot examples showing extraction of `payment_terms` from contracts with varied phrasing, including the narrative-prose case.

C. Change the field's schema type to a free-text string so the model has more flexibility in what it returns.

D. Add a validation step that flags any contract with a `null` payment_terms field for mandatory human review.

> [!question]- Show answer
> **Correct Answer: B.** This is the classic empty-extraction-field trigger for few-shot: the information exists but in an unexpected format (narrative prose instead of structured phrasing). A detailed system prompt already exists and isn't fixing it, so more prose instructions (A) are unlikely to help further — the model needs to see the pattern demonstrated. Option C loosens the schema without addressing why the model fails to recognize the information in the first place. Option D treats the symptom (null values) with a workaround rather than fixing the extraction itself.

---

**Question 2**

A code review agent classifies findings as either a genuine issue or an acceptable pattern. Logs show it frequently flags a specific idiom (a broad `except Exception:` block used intentionally at a top-level request handler to guarantee a clean error response) as a bug, even though the team's style guide explicitly allows this pattern in that specific location. The system prompt already states the general rule in prose. What is the most effective way to reduce this specific false positive while preserving the ability to catch genuine broad-exception misuse elsewhere?

A. Add a few-shot example showing this exact idiom at a top-level handler classified as acceptable, paired with a contrasting example showing a broad `except Exception:` deeper in business logic classified as a genuine issue, both with reasoning.

B. Remove the broad-exception-handling rule from the system prompt entirely so the model stops flagging it.

C. Add "be less strict about exception handling" to the system prompt.

D. Lower the confidence threshold required before the model reports a finding in this category.

> [!question]- Show answer
> **Correct Answer: A.** A single acceptable-pattern example paired with a contrasting genuine-issue example, each with reasoning, teaches the model the actual distinguishing principle (location and intent, not just the syntax pattern) so it generalizes correctly instead of flagging the syntax pattern everywhere. Option B loses legitimate detection ability elsewhere in the codebase. Option C is a vague instruction with no actionable decision boundary — the same failure mode tested in Task Statement 4.1. Option D relies on the model's self-reported confidence, which is poorly calibrated and does not reliably fix a systematic misclassification.

---

**Question 3**

Your team built a customer support agent with two tools: `billing_tool` ("Handles billing-related requests") and `technical_tool` ("Handles technical issues"). Tickets that blend both concerns (e.g., "I can't pay my invoice because the payment button errors out") are routed inconsistently — sometimes to one tool, sometimes to the other, across otherwise-identical tickets. You decide the fix is to add 3 few-shot examples showing this exact type of blended ticket routed correctly, each with a one-sentence reasoning explaining why one tool was chosen over the other. Why is this an appropriate use of few-shot examples here, rather than a case where a different technique should be used instead?

A. It isn't appropriate — descriptions this short mean the real fix is expanding each tool's description with input formats and boundary cases, not adding examples.

B. It is appropriate because the root cause is an ambiguous judgment call (routing) that produces inconsistent classification on similar inputs, and reasoning-rich examples teach the underlying decision principle rather than just the two literal tickets shown.

C. It is appropriate only if at least 5-8 examples are used, since 3 examples cannot establish a reliable pattern for tool routing specifically.

D. It isn't appropriate — routing decisions should always be enforced with a deterministic programmatic classifier rather than left to the model at all.

> [!question]- Show answer
> **Correct Answer: B.** Ambiguous judgment calls producing inconsistent classification on similar inputs are one of the three canonical triggers for few-shot examples, and reasoning-rich examples (why this tool over that one) teach a generalizable principle rather than memorizing the two tickets shown. Option A describes a real alternative fix (tool description enrichment) that applies when descriptions are the root cause of *general* misrouting to the wrong tool entirely — but here the described problem is inconsistency on a specific *ambiguous* blended case, which few-shot with reasoning directly addresses; 3 examples is within the effective 2-4 range, so A incorrectly assumes the wrong technique. Option C misapplies a number from a different scenario (tool-selection description gaps) to this one — 2-4 targeted examples is the correct range for this kind of ambiguous-case demonstration. Option D overcorrects to a rigid architecture where the ambiguity is genuinely a judgment call better suited to demonstrated reasoning, not a fixed rule.

---

**Question 4** *(Select the two best answers)*

Which two statements correctly distinguish when few-shot examples are the appropriate fix from when a different technique should be used instead?

A. Few-shot examples are the correct fix for an extraction field that returns `null` because the source information appears in an unexpected format, not because the information is actually missing.

B. Few-shot examples are the correct fix for a field that gets fabricated with a plausible-looking value when no such information exists anywhere in the source document.

C. Few-shot examples are the correct fix for a calculation that is internally inconsistent, such as a reported total that doesn't match the sum of its line items.

D. Few-shot examples are the correct fix for ambiguous judgment calls where similar inputs receive inconsistent classifications, provided the examples include reasoning rather than just input-output pairs.

> [!question]- Show answer
> **Correct Answer: A and D.** Empty/null extraction fields caused by an unexpected source format (A) and ambiguous judgment calls with inconsistent classification (D) are two of the three canonical triggers for few-shot examples — and D specifically requires reasoning in the examples, not bare input-output pairs, to teach a generalizable principle. Option B is incorrect: a fabricated value when no information exists is a schema problem, best solved by making the field explicitly nullable (Task Statement 4.3), not by adding examples. Option C is incorrect: an internally inconsistent calculation is caught and corrected by a validation-and-retry loop (Task Statement 4.4), not by demonstrating correct arithmetic via examples.

---

> [!tip] Repasa la teoría en [[1 resumen]], memoriza con [[3 cuestionario]] y practica el código en [[2 example]]
