Practice test for **Task Statement 4.3** — *Enforce structured output using tool use and JSON schemas* (Domain 4: Prompt Engineering & Structured Output). Based on `examguide.pdf`.

---

**Question 1**

Your team's document extraction pipeline currently prompts Claude with: `"Extract the fields below and respond only with valid JSON: {...}"`. In production, roughly 2% of responses fail `json.loads()` with errors like unterminated strings or trailing commas. What is the most effective structural fix?

A. Lower `max_tokens` so the model has less room to produce malformed output.

B. Replace the prompt-based JSON instruction with a `tool_use` call whose `input_schema` defines the required fields.

C. Add a regex-based JSON repair step that attempts to fix common syntax issues before parsing.

D. Add "Respond with ONLY valid JSON, no other text" to the prompt in bold at the top.

> [!question]- Show answer
> **Correct Answer: B.** `tool_use` with a JSON schema is the most reliable mechanism for guaranteed schema-compliant output — it eliminates JSON syntax errors structurally, rather than relying on the model reliably following a text instruction. Option A doesn't address the root cause (prompt-based extraction has no structural guarantee) and risks truncating legitimate output. Option C papers over the underlying reliability problem with a brittle patch instead of fixing it. Option D is a stronger version of the same prompt-based approach that already produces the 2% failure rate — restating the instruction more forcefully doesn't add a structural guarantee.

---

**Question 2**

A document classification pipeline processes incoming files that could be invoices, receipts, or contracts — the type is unknown until the model reads the content. You define three separate extraction tools, one per document type, and want to guarantee the model always calls exactly one of them rather than replying with text. Which `tool_choice` configuration should you use?

A. `{"type": "auto"}`

B. `{"type": "any"}`

C. `{"type": "tool", "name": "extract_invoice"}`

D. Omit `tool_choice` and rely on the tool descriptions alone to guide selection.

> [!question]- Show answer
> **Correct Answer: B.** `{"type": "any"}` forces the model to call a tool while still letting it choose which one — exactly what's needed when the document type is unknown ahead of time but structured output must be guaranteed. Option A (`"auto"`) allows the model to return plain text instead of calling any tool, which would break the guarantee. Option C forces one specific tool (`extract_invoice`), which is wrong here because the document might actually be a receipt or contract. Option D removes the guarantee entirely — without a `tool_choice` constraint, the model can decide not to call a tool at all.

---

**Question 3**

An invoice-extraction tool's JSON schema marks `purchase_order` as a required string field. Log review shows that for documents that don't mention a purchase order anywhere in the text, the model still returns a plausible-looking but incorrect PO number instead of indicating the data is missing. The extracted JSON always validates successfully against the schema. What change best addresses this?

A. Add a validation step after extraction that flags any `purchase_order` value not found verbatim in the source text.

B. Change `purchase_order` to an optional/nullable field so the model can return `null` when the document doesn't contain that information.

C. Add a prompt instruction telling the model "do not invent purchase order numbers."

D. Switch `purchase_order` from a string type to a number type to restrict the format of fabricated values.

> [!question]- Show answer
> **Correct Answer: B.** Required fields pressure the model to produce *some* value to satisfy the schema, even when the source document has no such information — making the field optional/nullable is the primary defence against this kind of fabrication, since it gives the model a valid, honest way to say the data isn't present. Option A treats the symptom after the fact rather than removing the pressure to fabricate in the first place, and adds an extra verification pass that a schema fix would make unnecessary. Option C relies on probabilistic prompt compliance layered on top of a schema that still structurally requires a value — the same class of fix the exam guide rejects elsewhere (vague instructions over structural fixes). Option D doesn't address the root cause at all; a fabricated number is still fabricated whether it's typed as a string or a number.

---

**Question 4** *(Select the two best answers)*

A finance team's extraction pipeline uses `tool_use` with a strict JSON schema. Every response parses successfully and matches the schema. However, spot-checks reveal that in some extracted invoices the `total_amount` field doesn't match the sum of the extracted `line_items`, and in a few cases a vendor's tax ID was extracted into the `purchase_order` field instead. Which two statements correctly explain this situation?

A. This indicates a bug in `tool_use` itself, since schema-compliant output should also be factually and internally consistent.

B. `tool_use` with JSON schemas eliminates syntax errors but does not prevent semantic errors such as values summing incorrectly or landing in the wrong field.

C. The pipeline needs a separate semantic validation step (e.g., checking that line items sum to the total) in addition to the schema-based extraction.

D. Switching `tool_choice` from `"any"` to a forced, specific tool name would resolve the sum-mismatch and field-placement errors.

> [!question]- Show answer
> **Correct Answer: B and C.** `tool_use` guarantees structure, not correctness — sum discrepancies and field-placement errors are exactly the class of semantic error the schema cannot catch on its own (B), so a separate validation step is required to catch them (C). Option A is a misunderstanding: this is expected, documented behavior of `tool_use`, not a defect — the schema was never designed to verify cross-field consistency. Option D confuses two unrelated concerns: `tool_choice` controls *whether/which* tool is called, not the semantic correctness of the values the model places inside that tool's fields — forcing a specific tool name would not fix a sum mismatch or a misplaced tax ID.

---

> [!tip] Repasa la teoría en [[1 resumen]], memoriza con [[3 cuestionario]] y practica el código en [[2 example]]
