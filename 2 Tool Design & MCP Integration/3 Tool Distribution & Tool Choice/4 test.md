---
tags:
  - claude-cert/dominio-2
  - task-statement/2.3
---

# 4 test — Tool Distribution & Tool Choice (CCAR-F Practice)

Practice exam covering **Task Statement 2.3: Distribute tools appropriately across agents and configure tool choice** (Domain 2, `examguide.pdf`). Scenario-based questions in the style of the real exam. Answers are hidden — commit to an answer before revealing.

---

## Question 1 (Official — examguide.pdf Sample Question 9)

During testing, you observe that the synthesis agent frequently needs to verify specific claims while combining findings. Currently, when verification is needed, the synthesis agent returns control to the coordinator, which invokes the web search agent, then re-invokes synthesis with results. This adds 2-3 round trips per task and increases latency by 40%. Your evaluation shows that 85% of these verifications are simple fact-checks (dates, names, statistics) while 15% require deeper investigation. What's the most effective approach to reduce overhead while maintaining system reliability?

A. Give the synthesis agent a scoped `verify_fact` tool for simple lookups, while complex verifications continue delegating to the web search agent through the coordinator.

B. Have the synthesis agent accumulate all verification needs and return them as a batch to the coordinator at the end of its pass, which then sends them all to the web search agent at once.

C. Give the synthesis agent access to all web search tools so it can handle any verification need directly without round-trips through the coordinator.

D. Have the web search agent proactively cache extra context around each source during initial research, anticipating what the synthesis agent might need to verify.

> [!question]- Show answer
> **Correct answer: A.**
> This applies the principle of least privilege by giving the synthesis agent only what it needs for the 85% common case (simple fact verification) while preserving the existing coordination pattern for complex cases.
> - **B is wrong** — the batching approach creates blocking dependencies, since synthesis steps may depend on facts that haven't been verified yet.
> - **C is wrong** — this over-provisions the synthesis agent, violating separation of concerns.
> - **D is wrong** — this relies on speculative caching that cannot reliably predict what the synthesis agent will need to verify.

---

## Question 2

A `data_ops` agent is given 18 separate tools, one for each supported transformation (`pivot_table`, `calculate_percentile`, `normalise_currency`, `deduplicate_rows`, and 14 others), all sharing the same input/output shape (a dataset in, a transformed dataset out). Production logs show the agent frequently invokes the wrong transformation tool when two operations have similar names. What is the most effective architectural fix?

A. Rewrite each tool's description with more distinguishing detail so the model can better tell them apart.

B. Consolidate the 18 tools into a single parameterized tool (e.g., `transform_data`) with an enum parameter selecting the operation.

C. Split the 18 tools across two new specialized agents, roughly 9 tools each, so no single agent carries the full load.

D. Add a routing layer that inspects the user's request and pre-selects the correct transformation tool before the agent runs.

> [!question]- Show answer
> **Correct answer: B.**
> When tools all share the same input-operation-output pattern, splitting by role only moves the same near-duplicate problem elsewhere — the fix is collapsing them into one parameterized tool, where every operation is still reachable as an enum value picked inside a single call, instead of a tool the model has to find among many near-identical descriptions.
> - **A is wrong** — better descriptions might marginally help, but with 18 tools sharing the same shape, the root cause is tool-count complexity, not description quality.
> - **C is wrong** — splitting 18 near-duplicate tools into two agents of 9 each still leaves each agent well above the 4-5 tool optimum, and doesn't address that the tools are fundamentally the same operation type.
> - **D is wrong** — a routing layer is over-engineered infrastructure for a problem that a schema-level fix (consolidation) solves directly.

---

## Question 3

A document-extraction pipeline must always output structured data — invoice, receipt, or contract fields — regardless of which document type is uploaded, and must never fall back to a plain-text conversational reply. The team currently configures the API call with `tool_choice: {"type": "auto"}` and three extraction tools (`extract_invoice`, `extract_receipt`, `extract_contract`). Occasionally, the model responds with a text description of the document instead of calling any tool. What is the correct fix?

A. Switch to `tool_choice: {"type": "any"}`, so the model must call one of the three tools but can still choose which one based on the document type.

B. Switch to `tool_choice: {"type": "tool", "name": "extract_invoice"}` to force a specific tool on every call.

C. Keep `tool_choice: {"type": "auto"}` but add a stronger instruction in the system prompt demanding a tool call every time.

D. Remove the three separate extraction tools and rely on the model's default text output, then parse the structure out of the response afterward.

> [!question]- Show answer
> **Correct answer: A.**
> `any` forces the model to invoke some tool — guaranteeing structured output — while still letting it pick the correct schema (invoice, receipt, or contract) based on the document type, which is exactly the scenario `any` is designed for.
> - **B is wrong** — forcing a single named tool (`extract_invoice`) would incorrectly apply the invoice schema even to receipts or contracts; forced selection is for mandatory workflow steps, not for choosing among mutually exclusive schema options.
> - **C is wrong** — `auto` never guarantees a tool call regardless of prompt wording; relying on instructions for a hard guarantee reintroduces the exact reliability gap that caused the bug.
> - **D is wrong** — this abandons structured tool output entirely and reintroduces the parsing fragility that structured tool use exists to avoid.

---

## Question 4

A multi-agent content pipeline has a `web_research_agent` that owns `search_web`, `fetch_page`, `extract_links`, and `save_snippet`. To "save engineering time," a teammate proposes also giving the separate `summarizer_agent` access to `fetch_page`, reasoning that "it might occasionally need to grab a page directly instead of waiting for research results." Production logs later show the summarizer agent sometimes fetches pages on its own instead of using the research agent's already-gathered snippets, producing summaries based on different source material than the rest of the pipeline used. What principle explains this outcome, and what's the fix?

A. This is expected and acceptable — cross-agent tool sharing improves resilience when one agent is unavailable; no fix is needed.

B. Agents with tools outside their specialization tend to misuse them; the fix is to remove `fetch_page` from the summarizer agent and, if a genuine high-frequency cross-role need exists, add a narrowly scoped tool instead.

C. The summarizer agent needs a better system prompt explaining that it should prefer the research agent's snippets over fetching pages itself.

D. The fix is to merge the summarizer agent and the web research agent into a single agent so there's no ambiguity about who fetches pages.

> [!question]- Show answer
> **Correct answer: B.**
> This is a direct instance of the guide's "agents with tools outside their specialization tend to misuse them" principle — giving `fetch_page` to the summarizer just in case led to it being used, causing inconsistent source material. If a real high-frequency cross-role need exists, the correct pattern is a deliberately scoped tool (like `verify_fact` for the synthesis agent), not handing over the other agent's general-purpose tool.
> - **A is wrong** — this is precisely the failure mode the guide warns against, not an acceptable tradeoff; it caused a real correctness problem (inconsistent sources).
> - **C is wrong** — relying on a prompt instruction to suppress use of an available tool is the probabilistic, unreliable fix the guide's tool-scoping principle exists to avoid; removing access is the deterministic fix.
> - **D is wrong** — merging agents is a disproportionate architectural change that abandons role separation entirely, when the actual fix is narrow tool scoping.

---

## Question 5 (multiple-response — select the two correct options)

You are designing tool access for a subagent that summarizes uploaded PDF and DOCX reports. A generic `fetch_url` tool is available in the shared tool registry and would technically work for this. Which **two** of the following reflect the guide's least-privilege approach to this situation?

A. Give the subagent `fetch_url` since it already exists and works, avoiding the effort of building a new tool.

B. Build a constrained `load_document` tool that validates the URL points to a supported document file type before loading it.

C. A constrained tool like `load_document` prevents the agent from retrieving arbitrary, unrelated URLs and makes the tool's intended purpose explicit through its description.

D. Give the subagent both `fetch_url` and `load_document` so it has a fallback if the constrained tool's validation is too strict.

> [!question]- Show answer
> **Correct answers: B and C.**
> - **B** — replacing a generic tool with a constrained, purpose-built one (like `load_document` for document URLs only) is the guide's core example of least privilege in tool design.
> - **C** — this states the two concrete benefits the guide attributes to constrained tools: preventing misuse of broad access and clarifying intent through a specific description.
> - **A is wrong** — reusing `fetch_url` "because it already works" is exactly the generic-tool anti-pattern the guide warns against; convenience isn't a substitute for scoping access to what the role actually needs.
> - **D is wrong** — giving both tools defeats the purpose of the constrained tool entirely, since the agent could simply fall back to the unrestricted `fetch_url` whenever `load_document` rejects a URL.

---

> [!tip] Repasa el tema completo en [[1 resumen]], [[2 example]] y [[3 cuestionario]]
