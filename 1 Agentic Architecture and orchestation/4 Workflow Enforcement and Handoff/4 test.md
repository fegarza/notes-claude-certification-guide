Claude Certified Architect (CCAR-F) practice exam for [[1 resumen]], targeting **Task Statement 1.4: Implement multi-step workflows with enforcement and handoff patterns** from the official `examguide.pdf`. Each question presents a real-world case; pick the correct option before revealing the answer.

---

**Question 1** *(Official sample question — examguide.pdf, Section 9, "Customer Support Resolution Agent")*

Production data shows that in 12% of cases, your agent skips `get_customer` entirely and calls `lookup_order` using only the customer's stated name, occasionally leading to misidentified accounts and incorrect refunds. What change would most effectively address this reliability issue?

A. Add a programmatic prerequisite that blocks `lookup_order` and `process_refund` calls until `get_customer` has returned a verified customer ID.

B. Enhance the system prompt to state that customer verification via `get_customer` is mandatory before any order operations.

C. Add few-shot examples showing the agent always calling `get_customer` first, even when customers volunteer order details.

D. Implement a routing classifier that analyzes each request and enables only the subset of tools appropriate for that request type.

> [!question]- Reveal answer
> **Correct: A.** When a specific tool sequence is required for critical business logic (like verifying customer identity before processing refunds), programmatic enforcement provides deterministic guarantees that prompt-based approaches cannot. B and C rely on probabilistic LLM compliance, which is insufficient when errors have financial consequences. D addresses tool availability rather than tool ordering, which is not the actual problem.

---

**Question 2**

A payments team is deciding how to enforce that `transfer_funds` can only execute after `verify_two_factor_code` has succeeded for the current session. The current system prompt already states this requirement clearly, but audit logs show 6% of transfers happen without a preceding successful 2FA check. The team proposes rewriting the system prompt with more explicit, emphatic language ("CRITICAL: you MUST verify 2FA before ANY transfer, no exceptions"). Will this fix the underlying problem?

A. Yes — emphatic language and capitalization measurably increase instruction-following reliability to effectively 100%.

B. No — this is still prompt-based guidance, which remains probabilistic regardless of wording; a financial operation like fund transfer requires a programmatic prerequisite gate.

C. Yes, as long as the emphatic instruction is paired with two or three few-shot examples demonstrating the correct order.

D. No — the real fix is to remove `transfer_funds` from the agent's tool list entirely and route all transfers through a separate, non-agentic system.

> [!question]- Reveal answer
> **Correct: B.** Rewording a prompt — however emphatic — does not change its fundamental nature: it is still instructions the model probabilistically follows, not a deterministic block. For a financial operation, the exam's decision rule requires programmatic enforcement (a gate that physically blocks `transfer_funds` until 2FA state is verified in code), not stronger phrasing. A overstates what prompt engineering can guarantee. C is the classic few-shot distractor — it improves the odds but not to 100%. D overcorrects: removing the tool eliminates the capability entirely rather than fixing the ordering problem, which is not what the scenario asks for.

---

**Question 3**

A team is designing enforcement for a content-formatting rule: all currency amounts in agent responses must be written as `$X.XX` rather than a raw float. Logs show this rule is followed correctly 97% of the time, with occasional formatting slips. A staff engineer suggests building a `PreToolUse`-style prerequisite gate that blocks any response containing an unformatted number. Is this the appropriate level of enforcement?

A. Yes — any deviation from a stated rule should be enforced programmatically regardless of the rule's stakes.

B. No — this is a low-stakes formatting preference, not a financial, security, or compliance operation; prompt-based guidance (or a lightweight lint/fix step) is an acceptable and proportionate approach.

C. Yes, because 3% is still a non-zero failure rate and the exam's decision rule treats any non-zero failure rate as unacceptable.

D. No — formatting issues should be fixed by upgrading to a larger context window so the model can "see" more examples of correct formatting.

> [!question]- Reveal answer
> **Correct: B.** The exam's decision rule is scoped to stakes, not to "any non-zero failure rate" in the abstract: financial, security, and compliance operations get programmatic enforcement because a single failure there causes real harm. A currency-formatting inconsistency is not a business risk, so prompt-based guidance (or a simple post-processing fix) is proportionate. A and C misapply the decision rule as if it were stakes-agnostic. D is an unrelated and irrelevant fix — context window size has nothing to do with output formatting consistency.

---

**Question 4** *(Multiple-response — select the two best answers)*

An escalation summary produced by a customer support agent reads: *"Customer wants a refund for order #8871, but I couldn't approve it because it exceeds my authorization limit. Please review."* The human agent who receives this has no access to the conversation transcript. Which two additions would most improve this handoff summary's usefulness, per the structured handoff protocol?

A. The specific refund amount requested, stated as a concrete figure rather than left implicit.

B. A recommended action for the human agent to take (e.g., "approve if the damage photo attached matches policy, otherwise deny").

C. A restatement of the agent's system prompt, so the human agent understands how the agent was configured.

D. A count of how many tool calls the agent made before escalating.

> [!question]- Reveal answer
> **Correct: A and B.** The structured handoff protocol requires the summary to be self-contained: a concrete refund amount and a recommended action are two of the five required fields (alongside customer ID, conversation summary, and root cause analysis), and both are missing from this example. C is irrelevant to resolving the customer's issue — the human agent needs the case's facts, not the agent's configuration. D is operational trivia that doesn't help a human decide what to do next.

---

**Question 5**

A support agent handles a customer message covering three separate concerns: a return request, a billing dispute, and a question about a promotional code. Logs show the agent resolved the return request, then ended its turn — the billing dispute and promo code question were never addressed, and the customer had to send a follow-up message repeating them. What is the most likely root cause, and what is the correct fix?

A. Root cause: the agent's tools are misconfigured for billing disputes. Fix: add a dedicated `billing_dispute` tool.

B. Root cause: the agent failed to decompose the multi-concern request into all of its distinct items before investigating and responding. Fix: decompose the request into all concerns, investigate each in parallel using shared context, and synthesize one unified resolution.

C. Root cause: the agent's context window was too small to hold all three concerns. Fix: increase the context window.

D. Root cause: the customer phrased the request ambiguously. Fix: have the agent ask a clarifying question before doing anything.

> [!question]- Reveal answer
> **Correct: B.** This matches the exam's multi-concern handling pattern exactly: the failure mode described (only the first item addressed, the rest silently dropped) is a decomposition failure, not a tool, context-size, or ambiguity problem. The fix is to decompose the request into all distinct items, investigate each in parallel with shared context (the same customer account is relevant to all three), and produce one unified response covering everything. A invents a tooling gap not supported by the scenario. C is implausible — three short concerns in one message don't approach context limits. D adds an unnecessary round trip when the request was clear enough to decompose without clarification.

---

**Question 6**

A team is deciding how to fix a compliance workflow where an AML (anti-money-laundering) check is sometimes skipped before a large transaction is approved. Someone proposes: "Let's add a routing classifier that detects large transactions and sends them to a specialized 'compliance-first' agent pipeline." Why does this not directly solve the described problem, per the exam's framing of routing versus per-agent enforcement?

A. It does solve the problem — routing classifiers are the standard mechanism for enforcing sequential compliance steps within an agent's execution.

B. A routing classifier determines which agent or pipeline handles a request; it doesn't guarantee that, once routed there, the AML check will actually execute before approval — that ordering still needs enforcement within the agent's own execution sequence.

C. It doesn't solve the problem because routing classifiers are deprecated in favor of few-shot prompting.

D. It doesn't solve the problem because AML checks cannot be automated at all and must always involve a human reviewer.

> [!question]- Reveal answer
> **Correct: B.** This mirrors the exam trap about routing classifiers: they solve *which agent handles what*, not *whether a required step happens in the right order within that agent's own execution*. Even after correct routing, the compliance-first pipeline still needs its own programmatic prerequisite gate to guarantee the AML check runs before approval — routing alone doesn't provide that guarantee. A misapplies routing to a problem it doesn't address. C and D are both unsupported claims not found in the guide.

---
> [!tip] Sigue con este tema
> Repasa la teoría en [[1 resumen]], refuerza con [[3 cuestionario]], y aplica el código en [[2 example]].
