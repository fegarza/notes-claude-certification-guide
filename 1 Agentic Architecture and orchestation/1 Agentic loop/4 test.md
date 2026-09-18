Claude Certified Architect (CCAR-F) practice exam for [[1 resumen]], targeting **Task Statement 1.1: Design and implement agentic loops for autonomous task execution** from the official `examguide.pdf`. Each question presents a real-world case; pick the correct option before revealing the answer.

---

**Question 1**

Production logs show that your customer support agent works correctly for simple queries, but in 8% of complex cases — where the agent needs to call a tool and explain what it's doing at the same time — the agent returns a half-finished response like "Let me check the inventory system..." and never surfaces the final answer. The relevant part of the loop code is:

```python
if response.content[0].type == "text":
    return response.content[0].text
```

What change would most effectively fix this reliability issue?

A. Increase `max_tokens` so Claude has room to finish its explanation before calling the tool.

B. Replace the content-type check with a check on `stop_reason`: continue the loop when it is `"tool_use"`, terminate only when it is `"end_turn"`.

C. Add an iteration cap of 10 so the loop has more chances to reach a final answer.

D. Set `tool_choice` to `{"type": "any"}` so Claude is guaranteed to call a tool instead of returning text.

> [!question]- Reveal answer
> **Correct: B.** The code treats "there is text in the response" as "the agent is done," but Claude can return text alongside a `tool_use` block in the same response. `stop_reason` is the only deterministic, unambiguous signal for loop control. A does not address the actual cause. C is the classic exam distractor: iteration caps fix runaway loops, not premature termination. D would force a tool call even when the agent is genuinely finished, risking an infinite loop.

---

**Question 2**

An engineering team is debugging an agent that, on tasks requiring 12+ chained tool calls, sometimes stops early with an apologetic message even though the underlying task was solvable. The loop code is:

```python
for i in range(10):
    ...
    if i == 9:
        return "I wasn't able to complete this request."
```

The team wants to keep some upper bound on the loop for safety, without reintroducing premature termination. What is the correct fix?

A. Remove the iteration bound entirely and rely only on `stop_reason`.

B. Raise the fixed bound to a very large number, such as 10,000.

C. Make `stop_reason == "end_turn"` the actual exit condition, and keep a generous iteration count (e.g. 20-30) only as a safety net that triggers an error/alert, not a silent apologetic response.

D. Replace the iteration count with a wall-clock timeout instead.

> [!question]- Reveal answer
> **Correct: C.** Iteration caps are legitimate only as a safety net against a genuinely runaway loop, never as the primary mechanism deciding whether the agent is finished — that decision belongs exclusively to `stop_reason`. A removes a legitimate safeguard against real infinite loops. B and D keep the same conceptual error: something other than `stop_reason` is still deciding when the agent "gives up."

---

**Question 3**

A research agent occasionally enters a loop where it calls the same search tool repeatedly without stopping, consuming an unusually large amount of API budget on a single request. Reviewing the request configuration, the team finds:

```python
tool_choice={"type": "any"}
```

Why is this configuration a plausible root cause of the runaway loop?

A. `tool_choice: any` silently disables the `stop_reason` field in the API response.

B. `tool_choice: any` forces Claude to call a tool on every single turn, including turns where it has already gathered everything needed to finish with `end_turn` — so the loop's termination signal never occurs.

C. `tool_choice: any` causes Claude to ignore previously appended tool results.

D. `tool_choice: any` reduces the model's context window, causing it to "forget" that it already has an answer.

> [!question]- Reveal answer
> **Correct: B.** Forcing tool use on every turn removes Claude's ability to genuinely signal completion. Since the loop's continuation condition is "stop_reason is tool_use," and that value is now forced to occur every turn, the loop cannot reach its own exit condition. A, C, and D describe behavior `tool_choice` does not actually control.

---

**Question 4**

An agent for an internal support tool decides, turn by turn, whether to call `verify_identity`, `create_ticket`, or `escalate_to_human` based on what the user has already provided in the conversation, rather than following a fixed, pre-programmed order. A reviewer flags this as risky and suggests hard-coding a fixed call sequence instead. Is the reviewer's concern well-founded for this use case?

A. Yes — any agentic loop should always follow a fixed, pre-programmed tool sequence for predictability.

B. No — this is model-driven decision-making, the approach favored by the exam for its flexibility, and there is no stated requirement here for deterministic, auditable compliance that would justify overriding it.

C. Yes — model-driven decision-making should only be used when there is exactly one tool available.

D. No — but only because internal support tools are explicitly out of scope for reliability concerns.

> [!question]- Reveal answer
> **Correct: B.** Letting Claude choose the next tool based on context is the preferred, model-driven approach; it should only be overridden by fixed, programmatic sequencing when business logic demands deterministic compliance (e.g. financial, security, or regulatory requirements) — which is not described here. A and C state the anti-pattern as a rule. D gives the right answer for the wrong (and irrelevant) reason.

---

**Question 5**

A fintech company is building a transfer-approval agent that must comply with strict regulatory requirements. The team is deciding between (1) letting Claude decide, per transfer, which compliance checks to run based on context, or (2) hard-coding a fixed, auditable sequence of checks. Which approach is correct, and why?

A. Always option 1 (model-driven), because the exam favors flexibility in all cases.

B. Option 2 (fixed, programmatic sequence), because this is exactly the kind of case — financial/regulatory compliance — where deterministic enforcement must override the model's flexibility.

C. Neither — agentic loops should not be used for regulated financial workflows at all.

D. Option 1, as long as an iteration cap is added for safety.

> [!question]- Reveal answer
> **Correct: B.** The guide is explicit that model-driven decision-making is preferred in general, **except** when business logic requires deterministic compliance — financial, security, or regulatory contexts are the textbook exception. A ignores that explicit exception. C and D don't engage with the actual tradeoff being tested.
