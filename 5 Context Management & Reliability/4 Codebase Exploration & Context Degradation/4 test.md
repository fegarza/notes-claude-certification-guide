Claude Certified Architect (CCAR-F) practice exam for [[1 resumen]], targeting **Task Statement 5.4: Manage context effectively in large codebase exploration** from the official `examguide.pdf`. Each question presents a real-world case; pick the correct option before revealing the answer.

> [!info] About official sample questions
> `examguide.pdf` has no sample question that maps 1:1 to Task Statement 5.4. All questions below are original, built on the Task Statement's "Knowledge of" and "Skills in" items, in the style of the official questions (scenario: *Developer Productivity with Claude*).

---

**Question 1**

A developer productivity agent is exploring an unfamiliar 400-file monorepo. Early in the session it correctly identified that `InvoiceRepository` (`src/billing/repos/invoice.ts`) overrides `save()` to emit an `InvoiceSaved` domain event. Two hours and ~60 file reads later, a developer asks how invoices trigger email notifications. The agent answers: *"Invoices likely follow the typical repository pattern and notifications are probably handled by an observer."* The session is at 55% of its context window. What is the most effective mitigation?

A. Switch to a model with a larger context window so all discovery output stays available throughout the session.

B. Have the agent maintain a scratchpad file recording key findings (class names, paths, dependency chains) from the start of exploration, and read it before answering subsequent questions.

C. Restart the session with a fresh context and instruct the agent to explore more efficiently this time.

D. Pre-load the full directory tree and all interface files into the system prompt so the agent has less to rediscover.

> [!question]- Reveal answer
> **Correct: B.** The agent is showing the classic symptom of context degradation — referencing "typical patterns" instead of specific classes it already found — while only at 55% of its window, which confirms this is not a token-limit problem. A scratchpad persists findings outside the conversation context, so they don't get buried by verbose output. A treats it as a capacity problem; a bigger window still fills with verbose output. C fixes the symptom but discards all accumulated knowledge. D front-loads even more volume into context, accelerating the very problem it aims to solve.

---

**Question 2**

Your team builds a coordinator agent for codebase onboarding. Today the coordinator has `Read`, `Grep` and `Glob` tools and performs all exploration itself. Logs show its answers become vague after investigating 4–5 modules. An engineer proposes spawning subagents for questions like "find all test files for the order service" and "trace the refund flow dependencies." In the design review, what is the strongest justification for this change?

A. Subagents can run in parallel, which reduces total wall-clock time for the exploration.

B. Each subagent explores in its own isolated context and returns only a structured summary, keeping the coordinator's context clean for high-level coordination.

C. Subagents automatically share the coordinator's full conversation history, so they never need to re-read files.

D. Subagents can use a cheaper model, reducing the cost of each file read.

> [!question]- Reveal answer
> **Correct: B.** The primary value of subagent delegation for codebase exploration is context isolation: verbose file contents and search results stay in the subagent's context, and the coordinator receives only key findings — directly addressing the degradation seen after 4–5 modules. A is a real but secondary benefit; it doesn't explain why the coordinator's answers would stop degrading. C is false — subagents do not inherit the coordinator's context; anything they need must be passed explicitly in their prompt. D may be a cost optimization but does nothing for context quality.

---

**Question 3**

A two-phase exploration pipeline first runs subagents to map the architecture (Phase 1), then spawns new subagents to investigate error handling in specific components (Phase 2). Phase 2 traces show each subagent spending its first 15–20 tool calls listing directories and reading controllers — the same files Phase 1 already analyzed — and one subagent investigated `LegacyRefundService`, which Phase 1 had identified as dead code. What is the best fix?

A. Give Phase 2 subagents a higher `max_tokens` so they have room to finish after re-exploring.

B. Merge both phases into a single long-running subagent so it remembers everything it explored.

C. Summarize Phase 1's key findings (architecture, relevant flows and classes, known concerns, Phase 2 objective) and inject that summary into the initial prompt of every Phase 2 subagent.

D. Pass the coordinator's complete Phase 1 conversation transcript, including all tool results, to every Phase 2 subagent.

> [!question]- Reveal answer
> **Correct: C.** This is the "cold start" problem: Phase 2 subagents don't inherit prior findings, so they duplicate exploration and lack the architectural understanding to ask the right questions (e.g., ignoring dead code). Injecting a concise Phase 1 summary into their initial context solves both. A doesn't prevent the duplication. B recreates a single long session that accumulates verbose output — the setup that causes context degradation. D passes the information but also all the verbose output, polluting each subagent's context from the start instead of giving it a focused summary.

---

**Question 4**

A multi-agent exploration of a large legacy codebase runs for about six hours. Twice this week, a network interruption killed the coordinator process, and each time the team had to restart from scratch, re-exploring files that had already been analyzed. How should you design the system to handle this?

A. Have each agent export structured state — explored paths, key findings, current phase, next steps, open questions — to a known file location; on resume, the coordinator loads this manifest and injects it into agent prompts.

B. Increase the retry count on all API calls so network interruptions are absorbed and the session never dies.

C. Log every tool call and tool result to a database; on resume, replay the full log into the coordinator's context to rebuild the conversation.

D. Run the exploration in smaller sessions and accept that some re-exploration is unavoidable after failures.

> [!question]- Reveal answer
> **Correct: A.** Structured state persistence with a manifest at a known location lets the coordinator resume from the last checkpoint and inject the state into agent prompts, so agents continue where they left off. B reduces some failures but can't prevent process crashes or context exhaustion, and doesn't preserve progress when they happen. C preserves data but replays all the verbose output into context, recreating context degradation on resume — the manifest should hold *structured state*, not raw history. D accepts the loss the question asks you to prevent.

---

**Question 5**

During an interactive Claude Code session, a developer has spent 90 minutes tracing a payments module. The conversation now contains dozens of full file reads, large grep outputs and directory listings. Claude still has room in its context window, but its answers have started to drift from the specific function names discovered earlier. The developer plans to keep exploring for another hour. What should they do?

A. Keep going and run `/compact` only once Claude Code warns that the context is nearly full.

B. Run `/compact` now — proactively — to summarize the verbose discovery output while preserving key findings, and continue recording findings in a scratchpad file.

C. Exit and start a brand-new session so the next hour begins with an empty context.

D. Ask Claude to re-read all previously explored files so the details are fresh again.

> [!question]- Reveal answer
> **Correct: B.** `/compact` should be used proactively during extended exploration when the context fills with verbose discovery output; its purpose is to protect context *quality*, not just quantity. Combined with a scratchpad, the key findings persist. A treats `/compact` as a last resort, by which time quality has already degraded (as it has here). C loses all accumulated knowledge unless state was persisted first. D adds even more verbose output to the context, making the degradation worse.

---

**Question 6**

An agent investigating a monolith has degraded badly: it contradicts its own earlier conclusions about the `AuthMiddleware` chain. The team lead decides a fresh session is necessary. What is the correct way to restart?

A. Start the new session immediately; the agent will rediscover the important parts faster the second time.

B. Before restarting, persist key findings to a scratchpad file and the exploration state to a manifest, then inject them into the new session's initial context.

C. Copy the last 20 messages of the degraded conversation into the new session as context.

D. Restart with a larger context window model so the degradation doesn't recur.

> [!question]- Reveal answer
> **Correct: B.** Restarting clears degradation but also discards all accumulated knowledge; findings must be persisted first (scratchpad + manifest) and then injected into the new session. A throws away hours of exploration. C carries over the most recent — and most verbose — part of the conversation, not necessarily the key findings, and may include the degraded, contradictory answers. D misdiagnoses degradation as a token-limit problem.

---

**Question 7** *(Select TWO)*

An architect is writing the system prompt for a long-running codebase exploration coordinator. Which **two** instructions most directly counteract context degradation?

A. "From the start of the session, maintain `notes/scratchpad.md` with key class names, file paths and dependency chains, and read it before answering any question about earlier findings."

B. "Read every file in the repository at the beginning so you have complete knowledge before answering."

C. "Delegate specific investigation questions (e.g., 'find all test files', 'trace refund flow dependencies') to subagents and keep only their structured summaries."

D. "When you are unsure about a detail from earlier, describe the most common pattern used in similar codebases."

E. "Only start writing notes once you notice your answers becoming inconsistent."

> [!question]- Reveal answer
> **Correct: A and C.** A establishes a scratchpad from the outset — persisting specific findings outside the conversation context. C uses subagent delegation for context isolation, keeping verbose exploration out of the coordinator's context. B floods the context with volume, causing degradation. D literally institutionalizes the degradation symptom ("typical patterns" instead of specific findings). E treats the scratchpad as a rescue move rather than a deliberate strategy from the start — by the time inconsistency is noticed, earlier details are already lost.

---

**Question 8**

Your coordinator spawns subagents to explore a codebase, and each subagent writes its findings to a manifest. After a crash, the coordinator restarts, but the newly spawned subagents still re-read files that were already explored. Inspecting the code, you find:

```python
manifest = json.load(open(".exploration/manifest.json"))
log.info("Resumed session %s at phase %d", manifest["sessionId"], manifest["phase"])

for task in PHASE_TASKS[manifest["phase"]]:
    spawn_subagent(prompt=TASK_PROMPTS[task])
```

What is the defect?

A. The manifest should be stored in the coordinator's conversation history rather than in a file.

B. The coordinator loads the manifest but never injects its contents (explored paths, key findings, next steps) into the subagent prompts, and it doesn't skip tasks already completed.

C. The subagents should be spawned sequentially instead of in a loop so they can share context.

D. The manifest is missing a timestamp, so subagents can't tell whether it's stale.

> [!question]- Reveal answer
> **Correct: B.** Crash recovery requires two steps: the coordinator loads the manifest **and injects it into agent prompts** so they continue where they left off. Here the manifest is only logged; subagents receive the original task prompts with no knowledge of prior progress, so they re-explore. A would lose the state on the very crash it's meant to survive. C misunderstands subagents — they don't share context; state must be passed explicitly. D may be a nice-to-have but doesn't explain the re-exploration.

---
> [!tip] Sigue con este tema
> Vuelve a la teoría en [[1 resumen]], repasa con [[3 cuestionario]] y revisa el código en [[2 example]].
