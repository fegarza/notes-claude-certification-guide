Claude Certified Architect (CCAR-F) practice exam for [[1 resumen]], targeting **Task Statement 5.2: Design effective escalation and ambiguity resolution patterns** from the official `examguide.pdf`. Each question presents a real-world case; pick the correct option before revealing the answer.

---

**Question 1**

A support agent's logs show that whenever a customer message contains words like "furious," "unacceptable," or all-caps text, the agent immediately routes the conversation to a human queue — regardless of what the underlying issue actually is. Human agents report that many of these routed conversations are trivial (e.g., a late-delivery refund) that the AI agent could have resolved itself. What is the most likely root cause, and what should replace it?

A. The agent's tool timeout is too short; increase the timeout so the agent has time to resolve trivial issues before routing.

B. The agent is using sentiment as an escalation trigger. Replace it with the three valid triggers (explicit human request, policy exception/gap, inability to progress), and have the agent acknowledge frustration while resolving in-capability issues directly.

C. The agent needs a larger `max_tokens` budget to write a more thorough response before escalating.

D. The agent should escalate only when sentiment is extremely negative, using a stricter numeric threshold.

> [!question]- Reveal answer
> **Correct: B.** Frustration does not correlate with case complexity — a furious customer can have a trivial issue. Sentiment-based escalation is an explicitly called-out anti-pattern; the fix is to escalate only on the three valid triggers, and to acknowledge frustration while resolving in-scope issues directly. D just tunes the same broken mechanism instead of replacing it. A and C don't address the actual cause.

---

**Question 2**

A team adds a step where the model outputs a self-assessed `confidence_score` (0-1) after handling each ticket, and any ticket scoring below 0.6 is escalated. After a month, review shows the agent escalates many simple password-reset tickets (low self-reported confidence) while confidently mishandling several complex billing disputes (high self-reported confidence). What does this outcome illustrate?

A. The threshold of 0.6 was set too low and should be raised.

B. Self-reported confidence scores are a reliable signal but the training examples were insufficient.

C. Self-reported confidence scores are poorly calibrated — models are often confident on hard cases and unsure on easy ones — so confidence should not be used as an escalation trigger at all.

D. The agent needs a separate classifier model to compute confidence instead of self-reporting it.

> [!question]- Reveal answer
> **Correct: C.** This is exactly the anti-pattern the exam tests: LLM self-reported confidence is poorly calibrated, frequently inverted relative to actual case difficulty. Escalation should be driven by the three valid triggers instead. A only adjusts a broken signal. B misdiagnoses the problem as a data issue. D still relies on a confidence-style proxy rather than the valid triggers.

---

**Question 3**

A customer writes: "This is ridiculous, my package is three days late, fix it now!" The agent has full authority and tooling to issue a shipping refund for late deliveries. According to the guide's distinction between frustration and explicit escalation requests, what should the agent do?

A. Escalate immediately, since the customer is clearly upset.

B. Acknowledge the customer's frustration and resolve the issue directly (e.g., process the shipping refund), since this is within the agent's capability and the customer has not asked for a human.

C. Ask the customer to confirm whether they are "really" upset before taking any action.

D. Escalate only if the customer uses profanity in a follow-up message.

> [!question]- Reveal answer
> **Correct: B.** The customer is frustrated but has not explicitly requested a human, and the issue is within the agent's capability — the correct response is to acknowledge the emotion and resolve directly. Escalation only becomes appropriate if the customer *reiterates* a preference for a human agent after being offered help. A applies the sentiment-escalation anti-pattern. C and D are not meaningful decision criteria in this framework.

---

**Question 4**

Later in the same conversation from Question 3, after the agent offers the shipping refund, the customer responds: "I don't want a refund, I want to talk to an actual person." What should the agent do now?

A. Continue offering alternative resolutions (store credit, expedited replacement) before escalating, since the underlying issue is still solvable.

B. Escalate immediately — the customer has now made an explicit request for a human, which is honored without further investigation or resolution attempts.

C. Ask the customer one more clarifying question to confirm they understand the refund offer.

D. Decline to escalate, since the agent already demonstrated it could resolve the issue.

> [!question]- Reveal answer
> **Correct: B.** Once the customer reiterates an explicit preference for a human agent, that becomes trigger 1 (explicit human request) and is honored immediately, without further attempts to resolve. A, C, and D all continue trying to resolve after an explicit escalation request has been made, which contradicts the "honor immediately" rule.

---

**Question 5**

A customer asks the agent to match a competitor's advertised price on an item they already purchased. The company's documented policy only addresses price adjustments for price drops on the company's own site within 14 days; it says nothing about competitor price matching. What should the agent do?

A. Treat this as a policy violation and deny the request, since competitor price matching is not listed as something the company offers.

B. Approve the price match anyway, since the customer is asking in good faith.

C. Recognize this as a policy gap — not a documented violation — and escalate to a human for a judgment call, since the specific scenario falls outside what the policy addresses.

D. Escalate only if the customer explicitly asks for a human.

> [!question]- Reveal answer
> **Correct: C.** This is the textbook example of trigger 2: the policy is silent on competitor price matching (a gap), which is different from a documented violation with a clear answer. Gaps require human judgment and should be escalated. A incorrectly treats silence as a denial. B incorrectly assumes approval without documented support. D ignores that a policy gap is, on its own, a valid escalation trigger independent of an explicit request.

---

**Question 6**

A lookup tool for "John Smith" returns two customer records with different account histories. The agent's current code is:

```python
matches = buscar_cliente("John Smith")
cliente = matches[0]  # tomar el primer resultado
```

What is wrong with this approach, and what should replace it?

A. Nothing is wrong; picking the first match is an acceptable default when results are sorted by relevance.

B. The agent should pick the most recently active account instead of the first one.

C. Selecting between ambiguous matches heuristically risks acting on the wrong customer's account (exposing data or taking an incorrect action); the agent should instead ask the customer for an additional identifier (email, phone, or order number) to disambiguate.

D. The agent should escalate to a human any time more than one match is returned.

> [!question]- Reveal answer
> **Correct: C.** Heuristic selection among multiple customer matches — whether "first result" or "most recent" — risks exposing another person's data or acting on the wrong record. The correct pattern is to request an additional identifier and re-query to confirm the exact match. A and B both restate the anti-pattern with a different heuristic. D is unnecessary — this is a case for clarification, not escalation, since it doesn't match any of the three valid escalation triggers.

---

**Question 7 (Select the two best answers)**

Which two of the following are examples of the anti-patterns the exam explicitly warns against when designing escalation logic? (Select 2)

A. Escalating because the customer's message was flagged as highly negative by a sentiment-analysis step.

B. Escalating because the customer explicitly asked to speak with a person.

C. Escalating because the model's self-reported confidence score for the ticket was below a set threshold.

D. Escalating because a required backend system returned a connection timeout after the agent attempted to resolve the issue.

> [!question]- Reveal answer
> **Correct: A and C.** Sentiment-based escalation (A) and self-reported confidence scores (C) are the two explicitly called-out unreliable proxies for case complexity. B and D are both valid, legitimate escalation triggers (explicit request and genuine inability to progress, respectively), not anti-patterns.

---

**Question 8**

A team is deciding how to best calibrate their support agent's escalation behavior. Option 1: write explicit escalation criteria with few-shot examples directly into the system prompt. Option 2: build a separate sentiment-analysis microservice and a confidence-scoring classifier, then route based on their combined output. Which approach does the guide recommend as the most effective starting point, and why?

A. Option 2, because dedicated models are always more accurate than prompt-based instructions.

B. Option 1, because explicit criteria with few-shot examples in the system prompt is the most effective way to calibrate escalation, and should come before infrastructure changes — especially since Option 2 reintroduces the sentiment/confidence anti-patterns.

C. Both options should be built in parallel and their outputs averaged.

D. Neither — escalation logic should be hard-coded entirely in application code, bypassing the model.

> [!question]- Reveal answer
> **Correct: B.** The guide is explicit that the most effective way to calibrate escalation is adding explicit criteria with few-shot examples to the system prompt, before more complex infrastructure. Option 2 is a trap: it rebuilds the exact two anti-patterns (sentiment-based escalation, confidence scoring) already identified as unreliable. A and C treat those unreliable signals as valuable. D discards the model-driven flexibility the guide favors for this kind of judgment call.
