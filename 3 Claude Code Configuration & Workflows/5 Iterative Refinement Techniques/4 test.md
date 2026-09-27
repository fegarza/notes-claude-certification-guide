Claude Certified Architect (CCAR-F) practice exam for [[1 resumen]], targeting **Task Statement 3.5: Apply iterative refinement techniques for progressive improvement** from the official `examguide.pdf`. Each question presents a real-world case; pick the correct option before revealing the answer.

> [!info] About official sample questions
> `examguide.pdf` (Section 9) has no sample question that maps 1:1 to Task Statement 3.5, so all questions below are original, built in the same style.

---

**Question 1**

A developer asks Claude Code to "convert every API handler so that it returns a standardized envelope with data and error fields." Across three runs on different files, Claude Code produces three different shapes: one nests errors under `error.details`, one uses a top-level `errors` array, and one leaves successful responses unwrapped. The developer knows exactly what the envelope should look like. What should they try first?

A. Rewrite the instruction with more precise language, specifying field names, nesting rules, and the treatment of successful responses.

B. Provide 2-3 concrete before/after examples of a handler's return value, including one edge case, and then verify the pattern on a new file.

C. Use the interview pattern so Claude Code asks clarifying questions about the envelope design before continuing.

D. Write a full integration test suite for every handler before allowing Claude Code to make any further changes.

> [!question]- Reveal answer
> **Correct: B.** The symptom is inconsistent interpretation of a transformation the developer already knows precisely — the textbook case for concrete input/output examples, which the model generalizes from more reliably than from any prose. Verifying on a new file confirms generalization. A is the classic trap: more precise prose still relies on interpretation. C targets the wrong gap — the developer isn't missing domain considerations; the model is misreading a known transformation. D could eventually help, but it's heavier and isn't the first technique to reach for when the issue is simply inconsistent interpretation of a known pattern.

---

**Question 2**

A team is writing a script that migrates 4 million legacy customer records to a new schema. Requirements include preserving `null` values, handling empty strings and malformed dates, splitting full names into first/last, and completing within 10 minutes. Initial attempts from Claude Code handle the common case but keep breaking on different edge cases each iteration, and the team's prose corrections ("be careful with missing data") don't converge. Which approach best fits?

A. Write a test suite first covering the happy path, each edge case, and the performance requirement; then iterate by sharing the test failures with Claude Code.

B. Ask Claude Code to interview the team about data migration best practices before writing any more code.

C. Consolidate all prose corrections into one longer, more detailed specification document and resubmit it.

D. Switch to a model with a larger context window so it can keep all the requirements in mind at once.

> [!question]- Reveal answer
> **Correct: A.** A complex transformation with many edge cases and a performance requirement is the scenario for test-driven iteration. Test failures ("Expected: null preserved, Actual: empty string") give unambiguous feedback that vague prose ("be careful with missing data") cannot. B misapplies the interview pattern — the team knows the requirements; the problem is getting them implemented correctly. C is still prose and still relies on interpretation. D misdiagnoses the problem as capacity; the issue is ambiguous feedback, not context size.

---

**Question 3**

A backend developer with no experience in payments must add idempotent retry handling to a checkout service that calls a third-party payment provider. They plan to prompt: "Add retry logic with exponential backoff to the charge endpoint." A senior engineer suggests a different first step. Which is it?

A. Provide 2-3 before/after examples of a retry-wrapped function so Claude Code applies the pattern consistently.

B. Write tests for the retry behavior first, then share the failures.

C. Ask Claude Code to pose questions about requirements, edge cases, and constraints before implementing — e.g., duplicate-charge prevention, idempotency keys, which failures are safe to retry.

D. Provide all the retry requirements in a single batched message so Claude Code sees every constraint at once.

> [!question]- Reveal answer
> **Correct: C.** The developer is working in an unfamiliar domain where they are likely to miss critical considerations (retrying a charge can double-bill a customer). The interview pattern surfaces those considerations before implementation. A and B both assume the developer already knows the correct expected output — which is exactly the knowledge they lack; they can't write correct examples or tests for considerations they haven't identified. D is a feedback-delivery rule, not a way to discover missing requirements — you can't batch constraints you don't know exist.

---

**Question 4**

After a code review, a developer has three changes for Claude Code to make: (1) error responses must include a numeric `error_code` field, (2) structured logs must record that same `error_code`, and (3) the TypeScript client SDK types must reflect the new field. The developer sends them one at a time, waiting for each result. After the third iteration, the log format uses a string code, the SDK type declares `errorCode?: number`, and the API emits `error_code` as an integer. What went wrong?

A. The changes were too large; each should have been split into smaller sub-steps.

B. The developer should have used the interview pattern to let Claude Code discover the requirements.

C. Sequential iteration was correct, but the developer should have provided more precise prose in each step.

D. The three issues interact, so they should have been sent in a single message so Claude Code could see all constraints at once and produce a coherent fix.

> [!question]- Reveal answer
> **Correct: D.** When fixes interact — the shape of `error_code` determines how it's logged and typed — the model needs all constraints in one message. Sequencing them let each fix make a local decision that conflicted with the others. A would make the fragmentation worse. B is irrelevant: the developer already knows the requirements. C accepts the sequential approach, which is precisely the mistake; more precise prose in isolated steps still hides the coupling between them.

---

**Question 5**

A developer needs two unrelated cleanups in a module: rename all functions to camelCase, and change indentation from 4 spaces to 2. They also need to fix a bug where a date parser returns `undefined` for leap days. They send all three in one message, and Claude Code's result renames a variable inside the date parser incorrectly and leaves the leap-day bug partially fixed. What is the best way to handle this kind of feedback going forward?

A. Always batch all feedback into one message to minimize the number of iterations.

B. Fix independent issues sequentially, one per iteration, so each piece of feedback clearly applies to one part of the code.

C. Provide 2-3 before/after examples for each of the three issues in the same message.

D. Use the interview pattern so Claude Code can ask which issue to prioritize.

> [!question]- Reveal answer
> **Correct: B.** The three issues are independent — none affects how another should be fixed. Batching independent issues can confuse the model about which feedback applies to which part of the code, which is what happened. A inverts the rule: batching is for interacting issues, not a default for efficiency. C adds more material to the same overloaded message without addressing the mixing problem. D uses a technique meant for unfamiliar domains, which this isn't.

---

**Question 6**

A developer provided three examples to Claude Code showing how to convert callback-based functions to `async/await`. The model now converts standard functions correctly and consistently, but it mishandles functions whose callback receives multiple success arguments. The developer is considering adding 15 more standard examples "to reinforce the pattern." What should they do instead?

A. Add one or two examples that specifically show the multi-argument callback edge case and its expected output.

B. Add the 15 examples, since more examples always improve generalization.

C. Replace the examples with a detailed prose explanation of how multi-argument callbacks should be handled.

D. Restart with the interview pattern to have Claude Code ask about callback conventions.

> [!question]- Reveal answer
> **Correct: A.** The standard pattern has already generalized; the gap is a specific edge case. The documented step is to add examples specifically showing edge-case handling — two or three well-chosen examples are enough for the pattern itself. B piles on redundant examples that don't address the failing case. C reverts to prose, which reintroduces interpretation ambiguity. D is for unfamiliar domains; the developer knows exactly what output they want.

---

**Question 7** *(Select the two correct answers)*

A team lead is writing internal guidance on refining Claude Code output. Which **two** statements are correct?

A. When a prose description yields different results each run, the first fix is to add more technical detail to the prose.

B. Test failures in the form "Expected X, got Y" are effective feedback because they leave no room for interpretation.

C. The interview pattern is the best choice when the developer knows the exact transformation but the model interprets it inconsistently.

D. When one fix changes the constraints for another, both fixes should be provided in the same message.

E. Independent issues should always be batched together to reduce the number of iterations.

> [!question]- Reveal answer
> **Correct: B and D.** B describes why test-driven iteration works: failures are concrete and unambiguous. D is the batch rule for interacting issues. A is the "refine the prose" trap — examples come first. C swaps the purpose of the two techniques: that scenario calls for concrete examples; the interview pattern is for unfamiliar domains. E inverts the sequential rule for independent issues.

---

**Question 8**

A data engineer's migration script passes all happy-path checks but, in staging, converts `null` values in the `middle_name` column to the string `"null"`. They ask Claude Code: "The migration has some problems with missing middle names, please make it handle missing data properly." The next version converts nulls to empty strings instead. What is the most effective next message?

A. "Handle missing data properly — null values should stay null, not become strings or empty strings, and please be careful this time."

B. A test case with a sample input record where `middle_name` is `null` and the expected output JSON with `"middle_name": null`, plus the current failing test output.

C. Ask Claude Code to interview the engineer about data quality requirements before touching the script again.

D. Ask Claude Code to rewrite the entire migration from scratch with a more defensive coding style.

> [!question]- Reveal answer
> **Correct: B.** The Task Statement explicitly calls for providing specific test cases with example input and expected output to fix edge-case handling such as null values in migration scripts. The failing output ("Expected: null, Actual: \"\"") tells Claude Code exactly what to fix. A is more precise prose, but still prose — "handle properly" was already misread twice. C adds an interview step for a requirement the engineer already knows exactly. D discards working code and still leaves "defensive" open to interpretation.

---
> [!tip] Keep going
> Review the theory in [[1 resumen]], drill with [[3 cuestionario]], and see the code in [[2 example]].
