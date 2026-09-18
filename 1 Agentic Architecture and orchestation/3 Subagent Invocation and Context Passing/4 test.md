Claude Certified Architect (CCAR-F) practice exam for [[1 resumen]], targeting **Task Statement 1.3: Configure subagent invocation, context passing, and spawning** from the official `examguide.pdf`. Each question presents a real-world case; pick the correct option before revealing the answer.

---

**Question 1**

A coordinator agent is configured with `allowedTools=["WebSearch", "Read", "Bash"]` and a fully-defined set of `AgentDefinition` objects under `options.agents`, including a well-scoped web search subagent and a document analysis subagent. When the coordinator tries to delegate research to these subagents, nothing happens — no subagent is ever invoked, and the coordinator falls back to answering directly. What is the most likely cause?

A. The `AgentDefinition` objects are missing a `description` field, so the coordinator cannot decide when to invoke them.

B. `"Task"` (or `"Agent"`) is missing from the coordinator's `allowedTools`, so the coordinator has no way to spawn subagents at all.

C. The subagents' `tools` lists are too restrictive, so they refuse to run when invoked.

D. The coordinator's model does not support multi-agent orchestration.

> [!question]- Reveal answer
> **Correct: B.** The Task tool (current name `Agent`) is a hard, binary gate: without it present in the coordinator's `allowedTools`, the coordinator cannot invoke any subagent regardless of how well those subagents are defined. A well-written `description` (A) affects *which* subagent gets picked, not whether spawning is possible at all. C and D describe failure modes that don't match "no subagent is ever invoked" — a restrictive tool list would cause a subagent to fail mid-task, not prevent invocation entirely, and model support for orchestration isn't a real constraint here.

---

**Question 2**

A research coordinator has three subagents: `web_search`, `doc_analysis`, and `synthesis`. QA testing shows that `synthesis` consistently produces well-written reports, but none of the claims in those reports include a source URL, document name, or page number — even though `web_search` and `doc_analysis` are independently verified to return well-sourced, complete results. What is the most likely root cause?

A. The `synthesis` subagent's system prompt does not explicitly instruct it to cite sources.

B. The coordinator is passing only the claim content to `synthesis`, without the structured metadata (source URLs, document names, page numbers) that `web_search` and `doc_analysis` returned.

C. `synthesis` should be given direct access to `WebSearch` so it can independently verify and cite each claim.

D. The `doc_analysis` subagent's output format is incompatible with what `synthesis` expects, so metadata is silently dropped during parsing.

> [!question]- Reveal answer
> **Correct: B.** This is the signature exam pattern for this task statement: when `web_search` and `doc_analysis` are verified correct but `synthesis` produces unsourced output, the root cause is almost always that the coordinator stripped metadata before passing content to `synthesis`. `synthesis` cannot cite what it never received, so A misdiagnoses a context-passing failure as a prompt-design failure. C over-provisions `synthesis` with tools it doesn't need, violating separation of concerns, and doesn't fix the actual break (the coordinator's context-passing step). D introduces an unverified parsing-compatibility explanation not supported by the scenario, which states both upstream subagents return correct, complete results.

---

**Question 3**

A coordinator needs to research a topic using both a `pricing_search` subagent and a `review_search` subagent. The two subagents have no dependency on each other's output. The current implementation invokes `pricing_search` in one coordinator turn, waits for its full result, and only then invokes `review_search` in a separate turn. A teammate flags this as a performance issue. What change should the coordinator make?

A. Merge `pricing_search` and `review_search` into a single subagent so only one invocation is needed.

B. Emit both Task tool calls for `pricing_search` and `review_search` within the same coordinator response, so they run in parallel instead of sequential turns.

C. Keep the sequential pattern, but reduce the `max_tokens` of each subagent's response to lower latency.

D. Let `pricing_search` call `review_search` directly once it finishes, skipping the round trip back through the coordinator.

> [!question]- Reveal answer
> **Correct: B.** Independent subagent tasks should be spawned in parallel by emitting multiple Task tool calls in a single coordinator response — sequential invocation across separate turns adds latency for no benefit when the tasks don't depend on each other. A discards the value of having two specialized, separately-scoped subagents and doesn't address the real issue (invocation pattern, not subagent count). C reduces output size but does nothing about the unnecessary sequential wait. D violates the hub-and-spoke rule that all subagent communication routes through the coordinator — direct subagent-to-subagent calls break observability and centralized error handling regardless of latency motivation.

---

**Question 4**

A coordinator has completed an initial analysis of a legacy codebase — reading key modules, mapping dependencies, and identifying two candidate refactoring strategies. The team wants to explore both strategies independently, using the same codebase analysis as a shared starting point, without either exploration affecting the other's session state. Which approach correctly achieves this?

A. Call `query()` twice with `resume=session_id` and no `fork_session` flag, once for each strategy.

B. Call `query()` twice with `resume=session_id, fork_session=True` for each strategy, creating two independent branches from the same analysis baseline.

C. Start two entirely new sessions from scratch, re-running the codebase analysis in each one.

D. Use a single session and alternate between the two strategies within the same conversation thread.

> [!question]- Reveal answer
> **Correct: B.** `fork_session` is a modifier on `resume`: passing both together branches off the named session's history instead of appending to it, giving each exploration an independent copy of the shared analysis baseline. A is the classic trap — `resume` alone appends to the same session, so the two calls would interfere with each other's state rather than staying independent. C wastes the already-completed analysis and risks inconsistency between the two re-runs. D loses the independence requirement entirely, since both strategies would share and potentially pollute the same ongoing conversation.

---

**Question 5**

A coordinator's `AgentDefinition` for a "refund verification" subagent includes a `description`, a `system prompt` instructing it to "verify refund eligibility using account history," and a `tools` list containing `get_account_history`, `process_refund`, and `send_email`. During a review, someone flags that this subagent's tool list is broader than its stated role. Which tool(s) should most likely be removed to properly scope this subagent, per the principle that tool restrictions should match a subagent's specific role?

A. `get_account_history`, since verification doesn't require reading account data directly.

B. `process_refund` and `send_email`, since a *verification* subagent's role is to check eligibility, not to execute the refund or notify the customer.

C. All three tools, since a subagent's `tools` list should always be empty unless it fails during testing.

D. Nothing should change; broader tool access simply gives the subagent more flexibility to handle edge cases.

> [!question]- Reveal answer
> **Correct: B.** `AgentDefinition` tool restrictions should scope each subagent to what its stated role actually requires. A subagent described as doing "refund verification" needs read access to account history to make its determination, but executing the refund (`process_refund`) and notifying the customer (`send_email`) belong to a downstream step the coordinator should route separately — granting them here violates least-privilege scoping. A is wrong because reading account history is exactly what verification requires. C is an overcorrection with no basis in the principle being tested. D ignores that unscoped tool access is precisely the anti-pattern the exam tests against — flexibility is not a valid reason to over-provision a subagent.

---

**Question 6 — Multiple response (Select the two)**

A coordinator repeatedly produces synthesis reports with unattributed claims, even though its `web_search` and `doc_analysis` subagents are independently confirmed to return complete, well-sourced findings. Select the **two** statements that correctly diagnose the problem and describe a valid fix.

A. The root cause is most likely that the coordinator is forwarding only claim text to the `synthesis` subagent, dropping `source_url`, `document_name`, and `page_number` in the process.

B. The fix is to update the coordinator's context-passing logic to forward the complete structured `Finding` objects — content and metadata together — to `synthesis`.

C. The fix is to rewrite the `synthesis` subagent's system prompt to say "always include a citation for every claim."

D. The fix is to grant `synthesis` direct access to the same search and document-reading tools that `web_search` and `doc_analysis` use.

> [!question]- Reveal answer
> **Correct: A and B.** This is the core context-passing failure pattern for this task statement: the root cause sits in what the coordinator forwards to `synthesis` (A), and the fix is to preserve the full structured findings — including metadata — when passing context downstream (B). C is the classic distractor: no prompt instruction can produce citation data that was never included in the subagent's input. D over-provisions `synthesis` with tools that duplicate work already done upstream, violating separation of concerns and not addressing the actual context-passing gap.

---

**Question 7**

During testing, the `synthesis` subagent frequently needs to verify simple facts (dates, names, statistics) while combining findings from other agents. Currently, each verification requires `synthesis` to return control to the coordinator, which invokes `web_search`, then re-invokes `synthesis` with the result — adding several round trips per task. Evaluation shows 85% of these verifications are simple lookups, while 15% require deeper investigation. What is the most effective way to reduce this overhead while preserving the hub-and-spoke communication pattern where it matters?

A. Give `synthesis` a narrowly-scoped `verify_fact` tool for simple lookups, while complex verifications continue to route through the coordinator to `web_search` as before.

B. Give `synthesis` full access to `web_search` so it can resolve any verification need without going back through the coordinator.

C. Have `synthesis` batch all its verification needs and send them to the coordinator only at the end of its pass.

D. Have `web_search` proactively cache extra context around every source during initial research, anticipating what `synthesis` might later need to verify.

> [!question]- Reveal answer
> **Correct: A.** This applies least-privilege scoping from `AgentDefinition`: `synthesis` gets exactly the narrow tool it needs for the common case (85% of verifications), while the coordinator continues to mediate the harder 15% through the existing hub-and-spoke pattern. B over-provisions `synthesis`, letting it bypass the coordinator entirely and losing centralized observability and error handling. C introduces blocking dependencies, since later synthesis steps may depend on facts verified earlier in the same pass. D relies on speculative caching that cannot reliably predict what will actually need verification.

---

**Question 8**

Two engineers disagree about how to configure a coordinator that needs to spawn a `web_search` subagent and a `doc_analysis` subagent. Engineer 1 writes `allowedTools=["Task", "WebSearch", "Read"]` on the coordinator and gives each subagent's `AgentDefinition` its own restricted `tools` list (`web_search` gets `WebSearch` only, `doc_analysis` gets `Read` only). Engineer 2 argues the coordinator's `allowedTools` should list only `"Task"`, since the coordinator itself never calls `WebSearch` or `Read` directly. Which configuration correctly reflects the requirement in the exam guide?

A. Engineer 1 is correct — the coordinator must list every tool any subagent might use, in addition to `"Task"`.

B. Engineer 2 is correct — the coordinator's `allowedTools` only needs `"Task"` (or `"Agent"`) to spawn subagents; each subagent's own tool access is governed separately by its `AgentDefinition`'s `tools` restriction, not the coordinator's `allowedTools`.

C. Neither is correct — `allowedTools` is not required at all as long as `AgentDefinition.tools` is set for each subagent.

D. Both configurations are functionally identical, so the distinction doesn't matter for the exam.

> [!question]- Reveal answer
> **Correct: B.** The binary requirement is specifically that `"Task"` (or `"Agent"`) must be in the *coordinator's* `allowedTools` to permit spawning subagents at all — it says nothing about the coordinator needing every tool its subagents use. Each subagent's own tool access is scoped independently through its `AgentDefinition`'s `tools` field. A conflates coordinator-level spawning permission with subagent-level tool scoping, which are separate mechanisms. C ignores the hard gate the exam guide states explicitly: without `Task`/`Agent`, spawning is impossible regardless of how `AgentDefinition.tools` is configured. D is wrong because the distinction (spawning permission vs. per-subagent tool scope) is exactly what the exam tests.
