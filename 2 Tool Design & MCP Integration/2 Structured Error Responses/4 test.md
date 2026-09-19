---
tags:
  - claude-cert/dominio-2
  - task-statement/2.2
---

# 4 test — Structured Error Responses (CCAR-F Practice)

Practice exam covering **Task Statement 2.2: Implement structured error responses for MCP tools** (Domain 2, `examguide.pdf`). Scenario-based questions in the style of the real exam. Answers are hidden — commit to an answer before revealing.

---

## Question 1 (Official — examguide.pdf Sample Question 8)

The web search subagent times out while researching a complex topic. You need to design how this failure information flows back to the coordinator agent. Which error propagation approach best enables intelligent recovery?

A. Return structured error context to the coordinator including the failure type, the attempted query, any partial results, and potential alternative approaches.

B. Implement automatic retry logic with exponential backoff within the subagent, returning a generic "search unavailable" status only after all retries are exhausted.

C. Catch the timeout within the subagent and return an empty result set marked as successful.

D. Propagate the timeout exception directly to a top-level handler that terminates the entire research workflow.

> [!question]- Show answer
> **Correct answer: A.**
> Structured error context gives the coordinator the information it needs to make intelligent recovery decisions — whether to retry with a modified query, try an alternative approach, or proceed with partial results.
> - **B is wrong** — the generic status hides valuable context from the coordinator, preventing informed decisions, even though retries did happen locally.
> - **C is wrong** — this suppresses the error by marking failure as success, which prevents any recovery and risks incomplete research outputs.
> - **D is wrong** — this terminates the entire workflow unnecessarily when recovery strategies could succeed.

---

## Question 2

A `check_inventory` tool queries a warehouse database for a given SKU. For a SKU that exists but currently has zero units in stock, the tool returns `{"isError": false, "content": "0 units available for SKU-4471", "structuredContent": {"resultCount": 0}}`. A teammate proposes changing this so that any `resultCount: 0` automatically sets `isError: true`, arguing that "zero results is still a kind of failure the agent should know about." What's the correct evaluation of this proposal?

A. It's correct — any result the agent needs to act on differently should be flagged as an error, regardless of whether the query itself succeeded.

B. It's incorrect — a query that runs successfully and legitimately finds nothing is a valid empty result, not an access failure; flagging it as an error invites the agent to retry a query that will keep returning the same correct answer.

C. It's correct, but only for inventory-related tools, since stock-outs are business-critical and warrant the retryable flag.

D. It's incorrect, but only because `isError: true` should be reserved exclusively for permission errors.

> [!question]- Show answer
> **Correct answer: B.**
> This is the "most testable concept" from the guide: distinguishing access failure from valid empty result. A successful query with zero matches should stay `isError: false`. Marking it as an error would cause the agent to retry a call that isn't broken — the empty result will recur identically every time, wasting effort.
> - **A is wrong** — "the agent should know about it differently" doesn't require the error flag; `resultCount: 0` in `structuredContent` already communicates that distinctly from a failure.
> - **C is wrong** — there's no tool-category exception to this distinction; it applies uniformly regardless of domain.
> - **D is wrong** — `isError: true` is used for any of the four execution-error categories (transient, validation, business, permission), not exclusively permission.

---

## Question 3

A `submit_refund` tool encounters a request for £750 against a store policy that caps automatic refunds at £500. The tool returns:

```json
{
  "isError": true,
  "content": "Refund exceeds policy limit",
  "structuredContent": {
    "errorCategory": "business",
    "isRetryable": false
  }
}
```

The agent receiving this response is unable to explain to the customer why the refund was denied beyond "a policy limit was exceeded." What is missing from this error response, and which category does it belong to conceptually?

A. A `resultCount` field, since business errors should be treated the same as valid empty results for customer communication purposes.

B. A human-readable `description` field with a customer-friendly explanation — business-rule violations need more than the category and retryable flag to let the agent communicate appropriately.

C. Nothing is missing — `errorCategory: "business"` and `isRetryable: false` are sufficient per the MCP structured error format, and further detail belongs in application logs, not the tool response.

D. The `errorCategory` should be changed to `permission`, since the customer effectively lacks the authority to receive this refund.

> [!question]- Show answer
> **Correct answer: B.**
> For business errors specifically, the guide's skills section calls out including "retriable: false flags and customer-friendly explanations for business rule violations so the agent can communicate appropriately." The category and boolean alone tell the agent *not to retry*, but not *why* — that's the job of `description`.
> - **A is wrong** — `resultCount` belongs to successful queries with no matches, not to a business-rule denial, which is a genuine execution error (`isError: true`).
> - **C is wrong** — the category and boolean are necessary but explicitly insufficient per the guide; the human-readable explanation is called out as a required skill, not an optional log-only detail.
> - **D is wrong** — `permission` describes access-credential failures (wrong account, insufficient scope), not a policy threshold being exceeded by an otherwise-authorized request; conflating the two miscategorizes the error and could misdirect recovery (e.g., trying different credentials instead of escalating for approval).

---

## Question 4

A tool integration wraps three failure scenarios uniformly as `isError: true` with `content: "Operation failed"` and no `structuredContent`: (1) the underlying API times out, (2) the caller passed a malformed date string, (3) the requesting user lacks permission for the resource. What is the direct consequence of returning identical, unstructured responses for these three distinct scenarios?

A. None — since all three set `isError: true`, the agent will correctly treat all three as non-retryable and escalate every time, which is the safe default.

B. The agent cannot make an appropriate recovery decision for any of them, because nothing distinguishes a transient issue worth retrying from a validation issue needing corrected input from a permission issue needing different credentials.

C. The agent will incorrectly treat all three as valid empty results and silently proceed as if the operation succeeded.

D. MCP will reject the response at the protocol layer because `structuredContent` is a required field for any `isError: true` response.

> [!question]- Show answer
> **Correct answer: B.**
> This is the core problem the guide opens with: uniform, generic error messages like "Operation failed" prevent the agent from making appropriate recovery decisions. Without `errorCategory` and `isRetryable`, the agent has no way to know the timeout should be retried while the malformed date and the permission denial should not be — nor what to do differently for each.
> - **A is wrong** — "always escalate" isn't correct for the timeout case, which is retryable and shouldn't necessarily be escalated to a human; treating it identically to the permission failure wastes the opportunity for automatic recovery.
> - **C is wrong** — `isError: true` unambiguously signals a failure, not a valid empty result; the confusion described in the guide runs the other direction (a valid empty result mistaken for a failure), not this one.
> - **D is wrong** — `structuredContent` is the recommended pattern for recoverable metadata, but nothing in the protocol layer enforces it as a hard requirement that would cause outright rejection; the failure mode here is a poor design choice, not a protocol violation.

---

## Question 5 (multiple-response — select the two correct options)

A coordinator delegates a multi-order refund lookup to a search subagent. The subagent hits a transient database timeout on the third of five orders. Which **two** of the following are consistent with how the guide says a subagent should handle this situation?

A. Attempt local recovery for the timeout (e.g., a bounded retry) within the subagent before deciding whether to involve the coordinator at all.

B. If local recovery doesn't resolve it, propagate to the coordinator the failure type, the partial results already gathered for the first two orders, and what was attempted for the third.

C. Immediately halt all processing and return only the string "error" to the coordinator, since any error should stop the subagent's work entirely.

D. Silently skip the failed order, return the two successful results as if they were the complete set, and let the coordinator assume all five orders were checked.

> [!question]- Show answer
> **Correct answers: A and B.**
> - **A** — the guide calls for "implementing local error recovery within subagents for transient failures," so the subagent should try to resolve a timeout on its own scope first.
> - **B** — only errors that can't be resolved locally should propagate to the coordinator, and they should come with partial results and what was attempted — not a bare failure notice.
> - **C is wrong** — halting entirely discards the two results already obtained and gives the coordinator nothing to work with, contradicting the "partial results" requirement.
> - **D is wrong** — silently dropping the failed order and presenting an incomplete set as complete hides a real failure, mirroring the "valid empty result vs. access failure" confusion the guide singles out as the most testable mistake in this area.

---

> [!tip] Repasa el tema completo en [[1 resumen]], [[2 example]] y [[3 cuestionario]]
