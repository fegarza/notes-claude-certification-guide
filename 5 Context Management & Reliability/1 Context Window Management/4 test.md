---
tags:
  - claude-cert/dominio-5
  - task-statement/5.1
---

# 4 test — Context Window Management (CCAR-F Practice)

Practice exam covering **Task Statement 5.1: Manage conversation context to preserve critical information across long interactions** (Domain 5, `examguide.pdf`). Scenario-based questions in the style of the real exam. Answers are hidden — commit to an answer before revealing.

---

## Question 1 (Official — examguide.pdf Practice Scenario, Section 5.1)

A customer support agent handles a multi-issue session. After several turns, the agent refers to "your recent refund request" instead of the specific $247.83 refund for order #8891. The conversation history is being summarised between turns to manage context length. What is the most effective fix?

A. Instruct the model to preserve all numerical values verbatim whenever it summarises the conversation history.

B. Extract transactional facts (amounts, dates, order numbers) into a persistent case facts block included in every prompt, outside summarised history.

C. Store the full conversation history in an external database and retrieve the relevant turns on demand whenever the agent needs to recall an earlier detail.

D. Increase the context window size so the full conversation history fits and summarisation never needs to run.

> [!question]- Show answer
> **Correct answer: B.**
> The persistent case facts block is the pattern designed exactly for this failure mode: it holds transactional facts outside the summarised history, so they survive regardless of what happens to the conversational narrative.
> - **A is wrong** — instructing the model to "preserve everything verbatim" is a prompt-based fix for a systematic behavior; summarisation reliably degrades numerical precision regardless of instructions, so this cannot be relied on.
> - **C is wrong** — this adds retrieval infrastructure to solve a problem that a much simpler structural pattern (the case facts block) already solves; it also doesn't specify how the agent knows *when* to retrieve, leaving the same blind spot.
> - **D is wrong** — a bigger context window delays the problem but doesn't eliminate the underlying risk that summarisation, whenever it does run, destroys transactional precision; it also does nothing for sessions that already exceed the (larger) limit.

---

## Question 2

An order-lookup tool returns 42 fields per call: internal audit timestamps, warehouse codes, carrier IDs, fulfilment centre identifiers, and five fields actually relevant to processing a return (order ID, order date, total amount, return eligibility, item description). The engineering team currently appends the full 42-field response to the conversation history on every call, then only displays the 5 relevant fields to the customer-facing UI. After 15 turns, response latency has grown noticeably. What is the root cause and the correct fix?

A. The root cause is model latency scaling with tool call count; the fix is to batch tool calls together to reduce the number of round trips.

B. The root cause is the full untrimmed result accumulating in conversation history across turns, which must be resent on every stateless request; the fix is to trim the tool result to relevant fields before it enters the history, not just before display.

C. The root cause is the UI rendering layer being slow to filter 42 fields; the fix is to precompute the filtered view once and cache it client-side.

D. The root cause is the model re-reasoning over the tool schema on every call; the fix is to shorten the tool's JSON schema definition.

> [!question]- Show answer
> **Correct answer: B.**
> Trimming at display time doesn't help: because the Claude API is stateless, the full 42-field result — already sitting in conversation history — gets resent on every subsequent request. Latency and token cost grow with every turn as the untrimmed data accumulates. The fix has to happen before the result enters history (e.g., in a `PostToolUse` hook), not at the point of display.
> - **A is wrong** — nothing in the scenario points to call volume or round-trip count as the driver; the described growth pattern (accumulation across turns) points specifically to context bloat.
> - **C is wrong** — the UI-side filtering already works correctly (only 5 fields are shown); the token cost problem is upstream, in what's stored in the model-facing history, not in rendering.
> - **D is wrong** — trimming a tool's schema definition doesn't address data that already accumulated in the conversation history from past tool *results*, which is what's actually growing.

---

## Question 3

A research pipeline runs three subagents in parallel, then feeds their combined output to a synthesis agent to produce a final report. The subagents' raw outputs are concatenated in the order they complete, each running several thousand tokens of prose. Reviewing the final report, the team finds that findings from the subagent that finished second — buried in the middle of the combined input — are consistently omitted from the synthesis, while findings from the first and third subagents are well represented. Engineers add an instruction to the synthesis agent's prompt: "Make sure to consider findings from all three sources equally." The omission persists. What does this outcome indicate, and what is the correct structural fix?

A. The instruction should be stronger and more specific, naming each subagent explicitly rather than referring to "all three sources."

B. This confirms the "lost in the middle" effect is a positional processing phenomenon that prompt-based instructions don't reliably fix; the correct fix is to place a key-findings summary at the beginning of the aggregated input, followed by detailed results under explicit section headers.

C. The synthesis agent's context window is too small for three subagents' worth of output; the fix is to route to a model with a larger context window.

D. The second subagent's output must contain lower-quality findings than the others; the fix is to review and improve that specific subagent's prompt.

> [!question]- Show answer
> **Correct answer: B.**
> The persistence of the omission after adding an explicit instruction is itself diagnostic: it shows the failure is positional (a well-documented effect where models process the beginning and end of long inputs more reliably than the middle), not a matter of insufficient emphasis. The guide is explicit that the fix here is structural — reordering the input with a summary section first and clear section headers — not a stronger prompt reminder.
> - **A is wrong** — the team already tried a targeted instruction ("consider findings from all three sources equally") and it didn't work; making it more specific doesn't address a positional processing effect.
> - **C is wrong** — nothing in the scenario indicates the model is truncating or refusing to process content due to length; the findings are present in the input but underweighted due to position, which a larger context window doesn't fix.
> - **D is wrong** — the pattern described (the *middle* item is consistently penalized regardless of which subagent finishes second) points to position, not to that specific subagent's content quality; swapping which subagent runs second would likely just move the omission to whichever output lands in the middle.

---

## Question 4

A developer is debugging why a multi-turn agent seems to "forget" facts from early in a long conversation even though the team's summarisation logic runs correctly and compresses old turns as designed. A teammate proposes: "Since the Claude API is stateless anyway, let's just stop sending turns older than the last 10 exchanges — that's effectively what summarisation was approximating anyway, and it's simpler." What is the correct evaluation of this proposal?

A. The proposal is sound — since the API doesn't retain state between requests, omitting older turns is equivalent in effect to summarising them, just cheaper to implement.

B. The proposal is flawed — omitting older turns entirely (truncation) is not equivalent to summarisation; it discards conversational coherence outright rather than compressing it, and neither approach protects transactional facts unless paired with a persistent case facts block.

C. The proposal is sound, but only if the discarded turns are logged externally for audit purposes.

D. The proposal is flawed only because 10 exchanges is an arbitrary cutoff; a larger fixed window (e.g., 30 exchanges) would resolve the issue.

> [!question]- Show answer
> **Correct answer: B.**
> Truncation and summarisation are not interchangeable. Because the API is stateless, coherence depends entirely on what's included in the request — truncating older turns removes them outright, with nothing left to reference them, while summarisation at least preserves a compressed trace of what happened. Neither approach, on its own, protects transactional details (amounts, dates, order numbers); that requires a persistent case facts block held outside whichever compression strategy is used.
> - **A is wrong** — "stateless" describes the server, not a license to discard history arbitrarily; the model still needs enough of the actual conversation content in the request to remain coherent, which truncation doesn't guarantee.
> - **C is wrong** — external logging solves an audit/observability problem, not the agent's in-conversation coherence problem, since the agent itself never sees the logged turns again.
> - **D is wrong** — the flaw isn't the specific cutoff value; any fixed-size truncation window can silently drop a fact when a conversation runs longer than that window, so a larger fixed window narrows but doesn't eliminate the failure mode.

---

## Question 5 (multiple-response — select the two correct options)

A multi-agent research system has a web-search subagent that returns its full reasoning trace, unfiltered source text, and a verbose narrative summary to a synthesis agent operating under a tight context budget. Which **two** of the following changes are consistent with the guide's recommended approach to upstream agent optimisation?

A. Modify the web-search subagent to return structured findings — claims, source, relevance score, publication date — instead of its verbose reasoning chain and raw content.

B. Require the subagent's structured output to include metadata such as publication dates and source locations, so the synthesis agent can produce accurate downstream synthesis without re-fetching context.

C. Keep the verbose reasoning trace in the payload, but instruct the synthesis agent's prompt to "ignore the reasoning section and focus only on conclusions."

D. Route the subagent's full raw output through a second summarisation pass performed by the synthesis agent itself before synthesis begins.

> [!question]- Show answer
> **Correct answers: A and B.**
> - **A** — this is the core recommendation: upstream agents should return structured data (key facts, citations, relevance scores) instead of verbose content and reasoning chains when a downstream agent has a limited context budget.
> - **B** — requiring metadata like dates and source locations in the structured output is explicitly called out as necessary to support accurate downstream synthesis.
> - **C is wrong** — leaving the verbose trace in the payload still consumes the synthesis agent's token budget; a prompt-level instruction to "ignore" content doesn't reclaim the tokens the model already had to process, unlike removing it at the source.
> - **D is wrong** — this pushes the summarisation burden onto the very agent that has the limited context budget, and risks reintroducing the progressive-summarisation trap (losing precision) instead of avoiding the problem by having the upstream agent emit structured output in the first place.

---

## Question 6

A team implements prompt caching for a support agent. The prompt is assembled as: `[current user message] + [cache_control breakpoint] + [system instructions] + [long product policy reference document]`, all passed inside the `messages` array with role `"user"`. Cache hit rates are near zero, and every request is billed at full input price. What are the two problems with this implementation?

A. The volatile content (current user message) is placed before the static content, so the prefix changes on every request and nothing matches; also, system-level static content belongs in the top-level `system` parameter, not inside a `"user"` message.

B. The `cache_control` breakpoint type should be `"persistent"` instead of `"ephemeral"` for support agents handling many customers.

C. Prompt caching does not support product policy reference documents; only tool definitions can be cached.

D. The request likely exceeds the maximum of four `cache_control` breakpoints allowed per request.

> [!question]- Show answer
> **Correct answer: A.**
> Two independent problems, both present here: (1) caching matches from the start of the prompt, prefix by prefix — placing the volatile user message *before* the static system instructions and reference doc means the prefix differs on every call, so there's never a match; (2) the Messages API has no `"system"` role for `messages` (which only takes `"user"` and `"assistant"` turns) — static system-level content belongs in the top-level `system` parameter, where the breakpoint should sit at the end of the static block.
> - **B is wrong** — `"persistent"` is not a valid `cache_control` type in the guide's model; the documented options are `ephemeral` (roughly 5-minute default) or `ephemeral` with an extended `ttl` of `"1h"`.
> - **C is wrong** — nothing restricts caching to tool definitions; long reference documents are exactly the kind of stable content the guide recommends placing in the cached static block.
> - **D is wrong** — this single-breakpoint setup is nowhere near the four-breakpoint maximum; the described symptom (zero cache hits) is fully explained by the ordering and placement problems in option A.
