Claude Certified Architect (CCAR-F) practice exam for [[1 resumen]], targeting **Task Statement 1.6: Design task decomposition strategies for complex workflows** from the official `examguide.pdf`. Each question presents a real-world case; pick the correct option before revealing the answer.

---

**Question 1** *(Official sample question — `examguide.pdf`, Section 9, Question 12)*

A pull request modifies 14 files across the stock tracking module. Your single-pass review analyzing all files together produces inconsistent results: detailed feedback for some files but superficial comments for others, obvious bugs missed, and contradictory feedback — flagging a pattern as problematic in one file while approving identical code elsewhere in the same PR. How should you restructure the review?

A. Split into focused passes: analyze each file individually for local issues, then run a separate integration-focused pass examining cross-file data flow.

B. Require developers to split large PRs into smaller submissions of 3-4 files before the automated review runs.

C. Switch to a higher-tier model with a larger context window to give all 14 files adequate attention in one pass.

D. Run three independent review passes on the full PR and only flag issues that appear in at least two of the three runs.

> [!question]- Reveal answer
> **Correct: A.** Splitting reviews into focused passes directly addresses the root cause: attention dilution when processing many files at once. File-by-file analysis ensures consistent depth, while a separate integration pass catches cross-file issues. Option B shifts the burden to developers without improving the system itself. Option C misunderstands that a larger context window doesn't solve attention *quality* issues — the files already fit, the problem was never capacity. Option D would actually suppress detection of real bugs by requiring consensus on issues that may only be caught intermittently, and does nothing to fix the underlying single-pass design.

---

**Question 2**

An engineering team is building an agent to process expense reports. Every report has the same five fields (date, amount, category, vendor, employee ID) in a fixed layout, and the extraction steps — parse header, extract fields, validate against policy, generate summary — never change from one report to the next. Which decomposition pattern fits this task, and why?

A. Dynamic adaptive decomposition, because financial data always requires careful step-by-step investigation.

B. Fixed sequential pipeline, because the steps and input structure are known in advance and consistency matters more than adaptability here.

C. Dynamic adaptive decomposition, because the agent should adjust its plan based on what it finds in each report.

D. Fixed sequential pipeline, because dynamic decomposition is only for coordinator/subagent architectures, never for single-agent workflows.

> [!question]- Reveal answer
> **Correct: B.** The task characteristics match the fixed-pipeline column of the decision framework exactly: predetermined fields, predetermined format, steps known in advance. A and C both apply the "investigate and adapt" reasoning to a task that has nothing left to discover — the structure is already fully defined. D reaches the right pattern for the wrong reason: the choice between patterns is about whether the task's scope is known in advance, not about single-agent versus coordinator/subagent architecture.

---

**Question 3**

A team is building an agent to investigate a production incident in an unfamiliar service with no existing documentation. They design it as a fixed pipeline: Step 1 checks recent deploys, Step 2 checks the database connection pool, Step 3 checks the cache layer, Step 4 writes an incident report — in that exact order, regardless of findings. During a real incident, Step 1 reveals the real cause (a bad deploy) but the agent still runs Steps 2-4 before concluding. What is the most likely problem with this design?

A. The pipeline has too many steps and should be reduced to two steps.

B. Nothing is wrong — a fixed pipeline guarantees the incident report always covers all four areas.

C. The task is open-ended investigation with unknown root cause, so it should use dynamic adaptive decomposition instead, letting findings at each step change what happens next.

D. The agent needs a larger context window to hold all four steps' findings at once.

> [!question]- Reveal answer
> **Correct: C.** Debugging an unfamiliar system is explicitly the decision framework's example for dynamic adaptive decomposition: "the root cause is unknown; investigation must adapt." Forcing a fixed sequence means the agent keeps executing predetermined steps even after the root cause is already found, wasting effort and delaying the conclusion. B defends the fixed-pipeline choice on the wrong grounds — full coverage isn't the goal when the cause is already known partway through. A and D are unrelated to the actual mismatch: the problem isn't step count or context size, it's that the pattern itself doesn't fit an open-ended investigation.

---

**Question 4**

A document-review agent processes 20 contracts in one pass and its findings degrade the same way the stock-tracking PR review did: thorough on the first few contracts, shallow on the rest, with inconsistent verdicts on identical clauses. A senior engineer proposes two independent fixes in parallel: (1) upgrade to a model advertised with a much larger context window, and (2) add a system-prompt instruction telling the agent to "maintain the same level of scrutiny for every contract, including the last ones." Will this combination fix the root cause?

A. Yes — a larger context window solves the capacity problem and the prompt instruction solves the consistency problem, so together they cover both angles.

B. No — both are the same category of non-fix: attention dilution is an architectural problem, and neither a bigger model nor a more emphatic prompt changes that the design is still one pass over 20 items with a fixed attention budget.

C. Yes, but only the prompt instruction is necessary; the context window upgrade is redundant.

D. No — the real fix is to reduce the number of contracts reviewed per run to 3, with no further changes needed.

> [!question]- Reveal answer
> **Correct: B.** Both proposed fixes target the wrong layer. A larger context window addresses whether the contracts *fit*, not how attention is allocated across them once they do — the files were already fitting, that was never the problem. A more emphatic prompt raises average quality but doesn't change that the model is still dividing one attention budget across 20 items in a single pass. A and C both accept one or both non-fixes as sufficient. D gets partway there (reducing items per pass helps within each batch) but is incomplete on its own: without a separate cross-item integration pass, splitting into groups of 3 still misses issues that span across those groups.

---

**Question 5** *(Multiple-response — select the two correct statements)*

A code-review agent is redesigned to fix attention dilution across a 14-file PR. Which two statements correctly describe the resulting multi-pass architecture?

A. The per-file local-analysis passes should each see only their own file's content, so every file gets the same attention budget regardless of its position in the PR.

B. The cross-file integration pass should run before the per-file local-analysis passes, so it can tell each local pass what to look for.

C. The cross-file integration pass runs after all per-file local passes complete, and its job is specifically to catch cross-cutting concerns — like the same pattern being flagged in one file but approved in another — that no single-file pass could detect on its own.

D. Splitting the 14 files into two batches of 7 files each, reviewed in two separate calls, is by itself equivalent to the full multi-pass architecture.

> [!question]- Reveal answer
> **Correct: A and C.** A is the definition of the local-analysis layer: isolating each file into its own pass is exactly what prevents the position-based degradation (thorough on early files, shallow on later ones). C is the definition of the integration layer: it exists specifically to catch inconsistencies across files, which is structurally invisible to any pass that only ever sees one file at a time. B reverses the required order — the guide's architecture runs local passes first and integrates after, since the integration pass needs the local findings as its input, not the other way around. D repeats the batching trap from the resumen: batching alone reduces dilution within each group but is not equivalent to the full architecture, because it still lacks a dedicated cross-item integration pass.

---
> [!tip] Sigue con este tema
> Repasa la teoría en [[1 resumen]], refuerza con [[3 cuestionario]], y aplica el código en [[2 example]].
