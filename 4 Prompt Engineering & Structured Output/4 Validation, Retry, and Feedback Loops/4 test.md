Practice test for **Task Statement 4.4** — *Implement validation, retry, and feedback loops for extraction quality* (Domain 4: Prompt Engineering & Structured Output). Based on `examguide.pdf`.

---

**Question 1** *(Official sample scenario, adapted from the study guide)*

Your extraction pipeline validates that line item amounts sum to the stated total. For Document A, the calculated sum is £450 but the stated total is £500. For Document B, the `department` field is missing entirely from the source text. Which retry strategy is correct?

A. Retry both documents with the validation errors, instructing the model to re-extract all fields.

B. Skip retries for both documents and flag them all for human review to ensure accuracy.

C. Retry Document A with the discrepancy error; flag Document B for human review since the information is absent from the source.

D. Retry both documents with the same prompt, since extraction is non-deterministic and may succeed on a second attempt.

> [!question]- Show answer
> **Correct Answer: C.** Document A has a fixable, retryable error — a numerical/structural discrepancy between the line items and the stated total, which retry-with-error-feedback can correct. Document B's failure is unfixable by retrying: the `department` value simply isn't in the source document, so no amount of re-prompting can produce a correct answer — routing it to human review is the only sound response. Option A wastes a retry on unfixable Document B and risks the model fabricating a department name. Option B needlessly sends Document A to human review when a targeted retry would resolve it. Option D ignores the distinction entirely and treats both failures as if extraction were simply flaky, which misdiagnoses Document B's root cause.

---

**Question 2**

A retry loop currently sends this follow-up message when an extraction fails schema validation: `"The previous extraction was incorrect. Please try again and be more careful."` Logs show the model frequently returns the exact same (incorrect) extraction on the retry. What is the most likely cause, and what is the fix?

A. The model has a context window limit that prevents it from re-reading the document; increase `max_tokens`.

B. The retry message doesn't include the specific validation error or the original document, so the model has no new information to correct its answer; include both.

C. The model is non-deterministic, so occasional repeated failures are expected and no fix is needed.

D. Switch `tool_choice` from a forced tool name to `"auto"` so the model has more freedom to self-correct.

> [!question]- Show answer
> **Correct Answer: B.** Retry-with-error-feedback requires three ingredients: the original document, the failed extraction, and the specific validation error. A vague "try again, be more careful" message supplies none of the third ingredient, so the model has no concrete signal to change its behavior and typically reproduces the same mistake. Option A misattributes the failure to a token limit rather than the actual missing feedback. Option C treats a fixable, diagnosable process gap as unavoidable randomness. Option D would make the failure worse, not better — allowing the model to skip calling the tool entirely removes the structural guarantee of getting a schema-compliant extraction at all.

---

**Question 3**

An invoice extraction schema includes both `calculated_total` (the sum of extracted line items) and `stated_total` (the total as written in the document), along with a `total_discrepancy` boolean. Why is this schema design preferable to extracting a single `total` field?

A. It reduces the number of tokens the model needs to generate per extraction.

B. It allows the mismatch between the line items and the document's stated total to be detected directly from the structured output, without a separate validation pass.

C. It guarantees that `tool_use` will eliminate all semantic errors in the extraction.

D. It is required because JSON Schema does not support single numeric fields.

> [!question]- Show answer
> **Correct Answer: B.** Extracting both values lets a downstream check (or the model itself) compare them directly; any mismatch is visible from the structured output itself, which is the essence of designing self-correction into the schema rather than relying solely on external validation logic. Option A is not the design's purpose and isn't necessarily true. Option C misunderstands the boundary covered in this task statement: `tool_use` eliminates syntax errors, not semantic ones like a sum mismatch — this schema pattern is specifically compensating for what `tool_use` does *not* guarantee. Option D is false; single numeric fields are fully valid in JSON Schema.

---

**Question 4**

A code-review pipeline attaches a `detected_pattern` field to every structured finding it returns (e.g., `"string concatenation in SQL query"`, `"variable shadowing in nested scope"`). Over several weeks, developers dismiss 85% of findings tagged with `"variable shadowing in nested scope"` but only 10% of findings tagged with `"string concatenation in SQL query"`. What is the best use of this data?

A. Remove the `detected_pattern` field, since it isn't part of the core finding and only adds noise to the schema.

B. Automatically suppress all findings with a dismissal rate above 50%, regardless of pattern.

C. Prioritize refining the prompt logic for the `"variable shadowing in nested scope"` pattern, since its high dismissal rate signals it is likely producing false positives.

D. Increase the model's `max_tokens` for findings tagged with high-dismissal patterns so it can provide more detail.

> [!question]- Show answer
> **Correct Answer: C.** `detected_pattern` exists to enable exactly this kind of systematic analysis: tracking which code constructs trigger findings so that patterns with high dismissal rates can be identified and prioritized for prompt refinement — turning developer dismissal behavior into an actionable feedback loop. Option A discards the mechanism this task statement is built around. Option B is a blunt, unconditional rule that could suppress genuinely valid findings without any human review of why they're being dismissed. Option D doesn't address the likely root cause (the pattern itself is poorly specified or overly broad) — more detail on a false positive is still a false positive.

---

**Question 5** *(Select the two best answers)*

A Python extraction pipeline uses `tool_use` with a strict JSON schema, so every response is guaranteed to be syntactically valid and schema-compliant. The team also layers a Pydantic model with a custom `model_validator` on top, which checks that line item amounts sum to the stated total before accepting an extraction. A teammate argues the Pydantic validator is redundant now that the schema is strict. Which two statements best explain why the Pydantic layer is still necessary?

A. `tool_use` with a strict schema only eliminates syntax errors (malformed JSON, missing required fields, wrong types) — it cannot express or enforce a cross-field rule like "line items must sum to the stated total."

B. Pydantic is required because the Anthropic API does not support JSON Schema natively.

C. When the `model_validator` raises a `ValidationError`, its per-field error messages can be formatted directly into a retry-with-error-feedback message, giving the model the specific correction signal it needs.

D. Strict tool schemas are deprecated in favor of Pydantic-only validation.

> [!question]- Show answer
> **Correct Answer: A and C.** Structural/syntactic guarantees (from `tool_use` and strict schemas) and semantic guarantees (from Pydantic validators) operate at different layers: the schema cannot express relationships between fields, so a validator is still needed to catch a sum mismatch (A) — and that validator's structured `ValidationError` output is exactly what feeds the retry loop's required "specific validation error" ingredient (C). Option B is factually wrong — the Anthropic API supports JSON Schema natively via `tool_use`, which is precisely what the strict schema in this scenario is using. Option D is also false; there is no such deprecation, and the two mechanisms are described as complementary, not competing.

---

> [!tip] Repasa la teoría en [[1 resumen]], memoriza con [[3 cuestionario]] y practica el código en [[2 example]]
