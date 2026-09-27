Practice test for **Task Statement 4.6** — *Design multi-instance and multi-pass review architectures* (Domain 4: Prompt Engineering & Structured Output). Based on `examguide.pdf`.

---

**Question 1**

Your team generates a bug fix with Claude in a single conversation, then in the very next message of that same conversation asks Claude to "review your fix carefully and point out any remaining issues." The review consistently comes back clean, but a separate human reviewer later finds an off-by-one error in the same fix. What is the most likely explanation, and what should you do instead?

A. The model needs a more explicit review prompt, such as "be extremely critical and assume there are bugs."

B. The model retains the reasoning context from generating the fix in the same session, making it likely to confirm its own decisions rather than question them; use an independent instance with no prior reasoning context instead.

C. The model should be given extended thinking during the original generation step so it catches the bug before the review step even runs.

D. The conversation has grown too long for the model to track context accurately; start a new conversation but keep asking the same model to review its own output.

> [!question]- Show answer
> **Correct Answer: B.** Self-review within the same session carries forward the reasoning that produced the original output, so the model already "knows" why it made each decision and tends toward confirmation instead of critical examination. An independent instance, with no access to that prior reasoning, evaluates the output with fresh eyes and is substantially more effective at catching subtle defects. Option A treats a structural limitation as an instruction-wording problem — no amount of "be critical" phrasing removes the retained reasoning bias. Option C misapplies extended thinking, which happens during generation, not review, and doesn't introduce an independent perspective. Option D correctly starts a new conversation but still asks the *same* generating context indirectly by chaining review requests to the same output — the fix must be an independent instance, not just a fresh conversation window.

---

**Question 2** *(Official sample question from the exam guide — Question 12)*

A pull request modifies 14 files across the stock tracking module. Your single-pass review analyzing all files together produces inconsistent results: detailed feedback for some files but superficial comments for others, obvious bugs missed, and contradictory feedback — flagging a pattern as problematic in one file while approving identical code elsewhere in the same PR. How should you restructure the review?

A. Split into focused passes: analyze each file individually for local issues, then run a separate integration-focused pass examining cross-file data flow.

B. Require developers to split large PRs into smaller submissions of 3-4 files before the automated review runs.

C. Switch to a higher-tier model with a larger context window to give all 14 files adequate attention in one pass.

D. Run three independent review passes on the full PR and only flag issues that appear in at least two of the three runs.

> [!question]- Show answer
> **Correct Answer: A.** Splitting reviews into focused passes directly addresses the root cause: attention dilution when processing many files at once. File-by-file analysis ensures consistent depth, while a separate integration pass catches cross-file issues. Option B shifts burden to developers without improving the system. Option C misunderstands that larger context windows don't solve attention quality issues. Option D would actually suppress detection of real bugs by requiring consensus on issues that may only be caught intermittently.

---

**Question 3**

A reviewer proposes upgrading from the current model to a higher-tier model with a significantly larger context window, reasoning that "if it can hold all 14 files in context at once, it should be able to review them all with equal attention." Why is this reasoning flawed?

A. It is not flawed — a larger context window directly increases how much attention the model gives to each file.

B. The larger context window increases how much text the model can hold, but doesn't prevent uneven attention distribution across that text; the model can still spread cognitive effort unevenly across files regardless of model tier.

C. It is flawed only because larger models are always slower, making the pipeline too slow for CI/CD, not because of any attention issue.

D. It is flawed because larger context windows increase hallucination rates on long inputs.

> [!question]- Show answer
> **Correct Answer: B.** The problem attention dilution describes isn't storage capacity, it's attention quality — how consistently the model examines each part of the input. A bigger context window lets the model contain more text but says nothing about how evenly it distributes analytical depth across that text. Only structured, focused per-file passes ensure consistent attention depth. Option A is the exact misconception the exam targets: capacity and attention quality are not the same thing. Option C introduces an unrelated (and unsubstantiated) latency argument instead of the real root cause. Option D is an unrelated and unsupported claim not tied to the attention dilution concept.

---

**Question 4**

You are designing a multi-pass code review pipeline for large pull requests. After the per-file local analysis pass completes for all files, what should the integration pass specifically check for that the per-file passes cannot, by design, catch on their own?

A. Whether each individual file follows the team's naming conventions.

B. Whether any single file contains an obvious syntax error.

C. Whether data flows between files in incompatible formats, whether the same pattern is flagged inconsistently across files, and whether module boundaries violate API contracts.

D. Whether the pull request description accurately summarizes the code changes.

> [!question]- Show answer
> **Correct Answer: C.** Per-file passes only ever see one file at a time, so by construction they cannot detect problems that only become visible when comparing findings or data flow across files — that is exactly the job of the separate integration pass: cross-file data format mismatches, contradictory findings on the same pattern, and API contract violations at boundaries between modules. Options A and B are local, single-file concerns that the per-file pass already covers on its own. Option D is unrelated to code correctness review and isn't a stated goal of either pass.

---

**Question 5**

Your review pipeline has each finding include a model-reported confidence score (e.g., 0.65) so that high-confidence findings can be routed directly to developers while low-confidence findings go to a human review queue. Before wiring this routing into an automated CI/CD gate, what must happen first?

A. Nothing further is needed — the model's self-reported confidence score can be used directly as the routing threshold, since the model has full visibility into its own certainty.

B. The confidence threshold must be calibrated by running a labeled validation dataset (with known-correct answers) through the system and measuring how well reported confidence correlates with actual accuracy.

C. The confidence score should be replaced entirely with a rule-based severity tag, since confidence scores are never useful signals.

D. The pipeline should skip confidence scoring altogether and route every finding to human review to be safe.

> [!question]- Show answer
> **Correct Answer: B.** Raw, self-reported confidence is uncalibrated — it reflects the model's own sense of certainty, not verified accuracy, so using it directly for automated routing is unreliable. Calibration requires a labeled dataset where the correct answers are already known, run through the system to measure the actual correlation between reported confidence and verification results; only after that can thresholds be safely used for automated decisions. Option A is the uncalibrated-confidence trap the exam specifically tests. Option C discards a useful signal instead of validating it. Option D over-corrects, eliminating the efficiency benefit of confidence-based routing entirely instead of calibrating it properly.

---

**Question 6** *(Select the two best answers)*

A team is redesigning their automated code review system after noticing inconsistent results on large, multi-file pull requests. Which two changes are consistent with the architectural principles for multi-instance and multi-pass review?

A. Replace same-session self-review with an independent Claude instance that reviews generated code without access to the generator's reasoning context.

B. Instruct the same reviewing instance, within the same conversation as the generator, to "think step by step and double-check your own work before responding."

C. Split large multi-file reviews into focused per-file passes for local issues, followed by a separate integration pass for cross-file data flow and consistency.

D. Increase `max_tokens` on the single review call so the model has more room to write out a longer, more thorough-sounding review of all files at once.

> [!question]- Show answer
> **Correct Answer: A and C.** Using an independent instance without the generator's reasoning context directly addresses the self-review confirmation bias (A), and splitting large reviews into per-file passes plus a separate integration pass directly addresses attention dilution and contradictory findings (C) — these are the two core architectural fixes for this task statement. Option B is still same-session self-review with only a wording change; it does not remove the retained-reasoning bias regardless of how the instruction is phrased. Option D increases output length but does nothing to fix single-pass attention dilution across 14 files — a longer response is not the same as more evenly distributed attention.

---

> [!tip] Repasa la teoría en [[1 resumen]], memoriza con [[3 cuestionario]] y practica el código en [[2 example]]
