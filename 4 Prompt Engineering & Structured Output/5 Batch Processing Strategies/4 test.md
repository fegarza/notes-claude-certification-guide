Practice test for **Task Statement 4.5** — *Design efficient batch processing strategies* (Domain 4: Prompt Engineering & Structured Output). Based on `examguide.pdf`.

---

**Question 1** *(Official sample question from the exam guide — Question 11)*

Your team wants to reduce API costs for automated analysis. Currently, real-time Claude calls power two workflows: (1) a blocking pre-merge check that must complete before developers can merge, and (2) a technical debt report generated overnight for review the next morning. Your manager proposes switching both to the Message Batches API for its 50% cost savings. How should you evaluate this proposal?

A. Use batch processing for the technical debt reports only; keep real-time calls for pre-merge checks.

B. Switch both workflows to batch processing with status polling to check for completion.

C. Keep real-time calls for both workflows to avoid batch result ordering issues.

D. Switch both to batch processing with a timeout fallback to real-time if batches take too long.

> [!question]- Show answer
> **Correct Answer: A.** The Message Batches API offers 50% cost savings but has processing times up to 24 hours with no guaranteed latency SLA. This makes it unsuitable for blocking pre-merge checks where developers wait for results, but ideal for overnight batch jobs like technical debt reports. Option B is wrong because relying on "often faster" completion isn't acceptable for blocking workflows — status polling doesn't change the lack of an SLA. Option C reflects a misconception: batch results can be correlated using `custom_id` fields, so ordering is a non-issue. Option D adds unnecessary complexity when the simpler solution is matching each workflow to its appropriate API.

---

**Question 2**

Your organization requires a 30-hour SLA for a batch-generated compliance report. The Message Batches API has a processing window of up to 24 hours with no guaranteed latency. How should you design the submission schedule?

A. Submit one batch right at the 30-hour deadline minus 24 hours, since that leaves the maximum possible processing time.

B. Submit a new batch every 4 hours, so that a fresh batch is always in flight within the 6-hour buffer left after accounting for the 24-hour worst case.

C. Submit batches every 24 hours exactly, matching the processing window to minimize the number of submissions.

D. Submit continuously in real time using the synchronous API instead, since batch cannot reliably meet any SLA.

> [!question]- Show answer
> **Correct Answer: B.** The 24-hour window is a maximum, not a guarantee, so the schedule must be built against that worst case: 30-hour SLA minus 24-hour worst case leaves a 6-hour buffer for collecting requests, validating inputs, and absorbing delays. Submitting every 4 hours keeps a fresh batch continuously in flight within that buffer, so a single slow or expired batch doesn't blow the SLA. Option A leaves zero margin for any delay or an expired batch — a single worst-case run would breach the SLA with nothing to fall back on. Option C also leaves no buffer at all (24-hour cadence against a 24-hour worst case). Option D abandons the cost savings the batch API exists for; the correct move is scheduling around the constraint, not avoiding batch entirely.

---

**Question 3**

You submit a batch of 500 invoice documents for extraction. When the batch completes, you parse the results and find that 40 requests returned with `result.type` of either `"errored"` or `"expired"`. What is the correct next step?

A. Resubmit the entire batch of 500 documents to guarantee consistent results across all items.

B. Discard the 40 failed documents entirely, since a 92% success rate is already high enough for most use cases.

C. Resubmit only the 40 documents identified by their `custom_id`, applying targeted modifications such as chunking documents that exceeded context limits.

D. Retry the 40 failed documents using the exact same batch request, since Claude's non-determinism means a second identical attempt will likely succeed.

> [!question]- Show answer
> **Correct Answer: C.** The `custom_id` field exists precisely to correlate each result with its original request, enabling a surgical retry of only the failed items — resubmitting with targeted modifications (like chunking oversized documents) addresses the likely root cause instead of blindly repeating the same request. Option A wastes cost reprocessing 460 documents that already succeeded. Option B silently drops data without any human review of whether those extractions matter. Option D misdiagnoses the failure as random noise rather than a fixable, diagnosable cause (e.g., a document exceeding the context limit will fail identically on a second unmodified attempt).

---

**Question 4**

A document analysis pipeline needs to, for each document, call an internal `lookup_vendor_database` tool your own backend code executes, use the tool's result mid-conversation, and then have the model continue generating the final structured output based on that result — all within a single request. The team wants to run this over 300 documents using the Message Batches API to save cost. What is the correct guidance?

A. This works fine in the Message Batches API as long as `custom_id` is set correctly on each request.

B. The Message Batches API does not support multi-turn tool calling for a client-executed tool within a single request; this step must run on the synchronous API instead.

C. This works in the Message Batches API only if `max_tokens` is increased enough to accommodate the tool call and its result.

D. This works in the Message Batches API as long as `tool_choice` is set to force the specific tool by name.

> [!question]- Show answer
> **Correct Answer: B.** The batch API cannot execute a client tool mid-request and use its result to continue processing within that same request — `lookup_vendor_database` is a client tool because the team's own backend code runs it, not Claude's infrastructure. A workflow requiring this pattern must use the synchronous API for that step. Option A is wrong because `custom_id` only affects request/response correlation, not what happens inside a single request's processing. Option C and D both misattribute the limitation to a token or tool-selection configuration issue, when it is a structural constraint of the batch API itself that no parameter can work around.

---

**Question 5** *(Select the two best answers)*

A team is evaluating whether to move a nightly test-generation workflow to the Message Batches API. Which two statements are accurate considerations for this decision?

A. Because the workflow is not blocking (results are reviewed the next morning), batch processing is an appropriate fit and can yield roughly 50% cost savings.

B. Before submitting the full volume of source files, the team should refine its prompt against a small representative sample to maximize first-pass success and reduce resubmission costs.

C. Since results are only reviewed the next morning, the team does not need to handle failed requests at all, because there is no urgency.

D. The team should design the workflow assuming batch results will typically arrive within an hour, since that is the common case in practice.

> [!question]- Show answer
> **Correct Answer: A and B.** A nightly, non-blocking workflow is exactly the kind of latency-tolerant use case the batch API is designed for, and the 50% cost savings is a direct, correct benefit (A). Refining prompts on a small sample before scaling to the full batch is the single most cost-effective strategy in this task statement, since it maximizes first-pass success and minimizes expensive resubmissions (B). Option C is wrong: failures still need to be identified by `custom_id` and resubmitted with targeted modifications, regardless of how relaxed the timeline is — ignoring them means silently losing data. Option D is the "assumes fast because it's often fast" trap: the API gives no latency SLA, so designs must account for the 24-hour maximum, not the typical case.

---

> [!tip] Repasa la teoría en [[1 resumen]], memoriza con [[3 cuestionario]] y practica el código en [[2 example]]
