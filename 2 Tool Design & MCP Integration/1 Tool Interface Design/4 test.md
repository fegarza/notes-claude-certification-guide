---
tags:
  - claude-cert/dominio-2
  - task-statement/2.1
---

# 4 test — Tool Interface Design (CCAR-F Practice)

Practice exam covering **Task Statement 2.1: Design effective tool interfaces with clear descriptions and boundaries** (Domain 2, `examguide.pdf`). Scenario-based questions in the style of the real exam. Answers are hidden — commit to an answer before revealing.

---

## Question 1 (Official — examguide.pdf Sample Question 2)

Production logs show the agent frequently calls `get_customer` when users ask about orders (e.g., "check my order #12345"), instead of calling `lookup_order`. Both tools have minimal descriptions ("Retrieves customer information" / "Retrieves order details") and accept similar identifier formats. What's the most effective first step to improve tool selection reliability?

A. Add few-shot examples to the system prompt demonstrating correct tool selection patterns, with 5-8 examples showing order-related queries routing to `lookup_order`.

B. Expand each tool's description to include input formats it handles, example queries, edge cases, and boundaries explaining when to use it versus similar tools.

C. Implement a routing layer that parses user input before each turn and pre-selects the appropriate tool based on detected keywords and identifier patterns.

D. Consolidate both tools into a single `lookup_entity` tool that accepts any identifier and internally determines which backend to query.

> [!question]- Show answer
> **Correct answer: B.**
> Tool descriptions are the primary mechanism LLMs use for tool selection. When descriptions are minimal, models lack the context to differentiate between similar tools. Option B directly addresses this root cause with a low-effort, high-leverage fix.
> - **A is wrong** — few-shot examples add token overhead without fixing the underlying issue.
> - **C is wrong** — a routing layer is over-engineered and bypasses the LLM's natural language understanding.
> - **D is wrong** — consolidating tools is a valid architectural choice but requires more effort than a "first step" warrants when the immediate problem is inadequate descriptions.

---

## Question 2

Your team maintains a `search_docs` tool with the description: "Analyses content and returns results." A second tool, `search_web`, has the description "Searches and returns results." Both operate on entirely different backends (internal docs vs. live web search), but users report the agent frequently calls the wrong one. Renaming and rewriting both descriptions is scheduled for next sprint. What is the underlying reason a full rewrite is warranted here, rather than just adding a boundary sentence to each?

A. Because both names and both descriptions are near-identical, so the ambiguity exists at multiple levels — fixing only the boundary sentence would still leave two generic, hard-to-tell-apart names in place.

B. Because tool descriptions should never be edited more than once per quarter, so it's more efficient to batch the full rewrite.

C. Because renaming a tool is a breaking change for the model and requires the description to be rewritten as a side effect.

D. Because a boundary sentence only works when there are more than 4-5 tools in the toolkit.

> [!question]- Show answer
> **Correct answer: A.**
> When both the names *and* the descriptions are generic and near-identical, the ambiguity isn't isolated to one dimension. A single boundary sentence added to otherwise vague descriptions wouldn't give the model enough to differentiate the tools reliably — the names themselves still don't signal distinct purpose. Both the rename and the description rewrite are warranted together.
> - **B is wrong** — there's no such cadence rule; the guide favors fixing ambiguity as soon as it's diagnosed.
> - **C is wrong** — renaming a tool doesn't mechanically require a description rewrite; they're independent fixes that happen to both be needed here.
> - **D is wrong** — boundary sentences aren't gated by toolkit size; the 4-5 tool threshold is about whether description fixes are the right *category* of remedy at all, not about when a single sentence "works."

---

## Question 3

An agent has 26 tools registered, covering CRM, billing, shipping, and internal analytics functions. The team notices declining tool-selection accuracy and responds by rewriting all 26 tool descriptions to the five-element production-grade format (purpose, inputs, example queries, edge cases, boundaries). Accuracy does not meaningfully improve. What is the most likely explanation?

A. The five-element format was applied incorrectly and needs a sixth element covering authentication requirements.

B. Selection is degrading from decision complexity at this toolkit size, not from ambiguous descriptions — the fix belongs to reducing or restructuring the toolkit, not to further description tuning.

C. The descriptions should have been converted to few-shot examples instead, since 26 tools exceeds the description-based selection ceiling.

D. A routing classifier should be layered in front of the now well-described tools to resolve the remaining ambiguity.

> [!question]- Show answer
> **Correct answer: B.**
> Past roughly 4-5 tools per agent, tool-selection accuracy degrades on decision complexity alone. Rewriting 26 descriptions treats a problem that doesn't exist here — the descriptions weren't necessarily the bottleneck. The correct remedy is toolkit-level: splitting by role or consolidating variants into parameterized tools (Task Statement 2.3), not further description polish.
> - **A is wrong** — there's no sixth element in the framework, and adding one wouldn't address a decision-complexity problem.
> - **C is wrong** — few-shot examples are never a sanctioned remedy for misselection, regardless of toolkit size.
> - **D is wrong** — routing classifiers are called out as mis-keyed for large tool sets, not a validated fix for this scenario.

---

## Question 4

A tool `process_document` accepts a `mode` parameter (`"extract"`, `"summarize"`, or `"verify"`) and has the description: "Processes a document according to the specified mode: extract, summarize, or verify." The team is debating whether this already satisfies the tool-splitting principle from the certification guide. Which statement correctly evaluates this design?

A. It satisfies the principle, because the description already lists all three operations explicitly.

B. It does not satisfy the principle — a single tool that still branches internally on a mode parameter keeps the ambiguity of "which operation is wanted" inside one call, instead of giving the model three narrow, separately described tools to choose between.

C. It satisfies the principle, because `mode` is a required, well-typed input, which is one of the five elements of a production-grade description.

D. It does not satisfy the principle, because the tool needs a boundary statement explaining when to use `extract` versus `summarize`.

> [!question]- Show answer
> **Correct answer: B.**
> Tool splitting means giving the model separate, purpose-specific tools with their own input/output contracts (`extract_data_points`, `summarize_content`, `verify_claim_against_source`) — not one generic tool whose internal `mode` parameter recreates the same "which operation?" ambiguity one level down. Listing the modes in the description doesn't remove the ambiguity; the model still has to infer which mode fits a given request.
> - **A is wrong** — listing the operations in prose doesn't split the tool; the ambiguity about which operation applies still exists per call.
> - **C is wrong** — a well-typed `mode` parameter is a schema detail, not a substitute for narrow, single-purpose tools.
> - **D is wrong** — a boundary statement differentiates between *separate* tools; it doesn't apply to branches within a single tool's `mode` parameter.

---

## Question 5 (multiple-response — select the two correct options)

A `get_customer` tool's description has just been rewritten to production-grade standard: it states purpose, accepted identifier formats, example queries, edge cases, and an explicit boundary against `lookup_order`. Despite this, production logs still show occasional misrouting of order-related queries to `get_customer`. Select the **two** most plausible root causes worth investigating next, consistent with the certification guide.

A. The system prompt contains keyword-sensitive wording (e.g., "always check customer details before proceeding") that creates a competing tool association independent of the tool descriptions.

B. The model has cached the old, minimal description from earlier in the conversation and needs a context reset before the new description takes effect.

C. `lookup_order`'s description was not updated to the same production-grade standard, so the asymmetry still leaves room for ambiguity between the two tools.

D. The five-element format inherently caps selection accuracy below 100%, so some residual misrouting is expected and not worth investigating further.

> [!question]- Show answer
> **Correct answers: A and C.**
> - **A** — keyword-sensitive system prompt instructions can silently override well-written tool descriptions by creating an unintended association on every turn; this is a documented, exam-tested failure mode.
> - **C** — the fix in the example scenario always rewrites *both* descriptions together; if only one side was updated, the asymmetry can still leave enough ambiguity for occasional misrouting.
> - **B is wrong** — tool descriptions are not cached between API calls; every request is evaluated fresh against the tool definitions sent with it.
> - **D is wrong** — the guide treats production-grade descriptions as capable of driving reliable selection (9/10 or 10/10 in the worked exercise); it does not present the five-element format as having a built-in accuracy ceiling.

---

> [!tip] Repasa el tema completo en [[1 resumen]], [[2 example]] y [[3 cuestionario]]
