Claude Certified Architect (CCAR-F) practice exam for [[1 resumen]], targeting **Task Statement 1.5: Apply Agent SDK hooks for tool call interception and data normalization** from the official `examguide.pdf`. Each question presents a real-world case; pick the correct option before revealing the answer.

> [!info] No official Task 1.5 sample question in `examguide.pdf`
> Section 9 of `examguide.pdf` includes no sample question that tests `PreToolUse`/`PostToolUse` directly (its closest question, about blocking `lookup_order` until `get_customer` verifies identity, targets the general prerequisite-gate pattern under Task Statement 1.4). All questions below are original, built to the same scenario — "Customer Support Resolution Agent," using its documented tools `get_customer`, `lookup_order`, `process_refund`, `escalate_to_human` — and to the same style as the guide's real sample questions.

---

**Question 1**

Your support agent's `process_refund` tool has a business rule: refunds above $500 must be blocked and routed to human escalation, with zero exceptions. An engineer proposes adding a `PostToolUse` hook that inspects the refund amount in the tool's result and, if it exceeds $500, marks the transaction for manual reversal. Will this hook satisfy the "zero exceptions" requirement?

A. Yes — `PostToolUse` hooks can inspect any field in the tool result and take corrective action.

B. No — by the time `PostToolUse` fires, `process_refund` has already executed and the money has already moved; a `PreToolUse` hook is required to block the call before it runs.

C. Yes, as long as the hook also calls `escalate_to_human` immediately after flagging the transaction.

D. No — hooks cannot inspect tool results at all, only tool inputs.

> [!question]- Reveal answer
> **Correct: B.** `PostToolUse` runs after the tool has already executed — the refund has already been processed by the time the hook sees it. Marking it "for manual reversal" doesn't undo the real side effect. Meeting a zero-exception requirement means blocking the call before it runs, which only `PreToolUse` can do. A and C both assume `PostToolUse` can prevent something that already happened. D is factually wrong — `PostToolUse` hooks do inspect tool results; that's their entire purpose for normalization use cases.

---

**Question 2**

A payments agent has three MCP tools that each return transaction timestamps in a different format: one returns Unix epoch integers, one returns ISO 8601 strings, and one returns `DD/MM/YYYY` strings. Production logs show the agent occasionally misreads the day/month order in the third tool's dates when summarizing transaction history to customers. What is the most effective fix?

A. Add a `PostToolUse` hook on all three tools that converts every timestamp to a single ISO 8601 format before the model sees it.

B. Add few-shot examples to the system prompt showing correct interpretation of each date format.

C. Add a `PreToolUse` hook that blocks any tool call where the date format cannot be validated.

D. Instruct the model, via the system prompt, to always double-check date formats before summarizing.

> [!question]- Reveal answer
> **Correct: A.** This is a data-normalization problem, not a policy-enforcement problem — there's no call to block, just heterogeneous formats to standardize before the model has to interpret them. A `PostToolUse` hook eliminates the interpretation step entirely by guaranteeing one consistent format. B and D are prompt-based fixes that reduce but never eliminate the misread rate, since the model is still doing probabilistic interpretation. C misapplies `PreToolUse`: the problem isn't whether the call should happen, it's what format the result comes back in — `PreToolUse` fires before the tool runs and has no role in transforming its output.

---

**Question 3**

A compliance team requires that `transfer_funds` never execute unless `aml_check` has returned a passing result earlier in the same session. Audit logs show the agent currently completes AML checks 96% of the time before transferring, following a system-prompt instruction. Which implementation change closes the remaining gap to 100%?

A. Rewrite the system-prompt instruction to be more explicit about the required order of operations.

B. Add a `PreToolUse` hook on `transfer_funds` that checks whether `aml_check` passed for the current session and returns `permissionDecision: deny` if it has not.

C. Add a `PostToolUse` hook on `transfer_funds` that flags transfers missing a prior AML check for later audit review.

D. Add few-shot examples demonstrating the agent calling `aml_check` before `transfer_funds`.

> [!question]- Reveal answer
> **Correct: B.** A compliance requirement with a "never" clause needs a deterministic guarantee — a `PreToolUse` hook that blocks `transfer_funds` outright until AML state is verified in code, independent of what the model decides to do. A and D are still prompt-based and, by the guide's own decision framework, remain probabilistic no matter how explicit the wording. C only observes and flags after the transfer already executed, which doesn't prevent the compliance violation — it just documents it after the fact.

---

**Question 4** *(Multiple-response — select the two correct statements)*

A `PreToolUse` hook is attached to `escalate_to_human`, and a `PostToolUse` hook is attached to `get_customer`. Which two statements correctly describe what each hook can and cannot do?

A. The `PreToolUse` hook on `escalate_to_human` can return `permissionDecision: deny` to prevent the escalation call from running at all.

B. The `PostToolUse` hook on `get_customer` can prevent `get_customer` from running if the returned customer record looks invalid.

C. The `PostToolUse` hook on `get_customer` can return `updatedToolOutput` to reshape the customer record before the model sees it, without changing anything in the backend system.

D. The `PreToolUse` hook on `escalate_to_human` can retroactively cancel an escalation that has already been routed to a human queue.

> [!question]- Reveal answer
> **Correct: A and C.** `PreToolUse` fires before execution, so `permissionDecision: deny` (A) genuinely prevents the call from running. `PostToolUse` fires after execution but before the model reads the result, so it can only reshape the already-returned data via `updatedToolOutput` (C) — it cannot stop the underlying `get_customer` call, which makes B false. D is also false for the same reason as B, just in reverse direction: `PreToolUse` acts before execution, so there is nothing "already routed" for it to retroactively cancel — that would require intercepting the call before it ran in the first place, not after.

---

**Question 5**

A team wants every agent response involving a dollar amount to display exactly two decimal places (`$42.50`, not `$42.5` or `42.5`). Logs show the current system-prompt instruction achieves this format correctly 97% of the time. A staff engineer proposes writing a `PreToolUse` hook that blocks any response generation until the formatting is verified. Is this the right level of enforcement?

A. Yes — any documented formatting rule should be enforced with a hook, regardless of the consequence of a miss.

B. No — a formatting preference is not a financial, security, or compliance operation; a 3% occasional miss carries no real business risk, so prompt-based guidance (or a simple post-processing fix) is proportionate.

C. Yes, because `PreToolUse` hooks are specifically designed for output-formatting problems.

D. No — the fix should instead be a `PostToolUse` hook that intercepts the model's final response and reformats any incorrect numbers.

> [!question]- Reveal answer
> **Correct: B.** The decision framework is scoped to consequence, not to "any observed failure rate": a currency-formatting slip has no financial, security, or compliance consequence, so prompt-based guidance remains proportionate even at 97%. A misapplies the framework as if every rule deserved deterministic enforcement regardless of stakes. C is wrong on the mechanism — `PreToolUse`/`PostToolUse` intercept tool calls and their results, not the model's free-text response generation, which isn't a tool call at all. D repeats that same mechanism error.

---
> [!tip] Sigue con este tema
> Repasa la teoría en [[1 resumen]], refuerza con [[3 cuestionario]], y aplica el código en [[2 example]].
