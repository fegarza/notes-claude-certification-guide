Practice exam for Task Statement 3.4 — *Determine when to use plan mode vs direct execution*. Based on the concepts in [[1 resumen]].

## Question 1 (Official sample question — from `examguide.pdf`, Question 5)

You've been assigned to restructure the team's monolithic application into microservices. This will involve changes across dozens of files and requires decisions about service boundaries and module dependencies. Which approach should you take?

A. Enter plan mode to explore the codebase, understand dependencies, and design an implementation approach before making changes.

B. Start with direct execution and make changes incrementally, letting the implementation reveal the natural service boundaries.

C. Use direct execution with comprehensive upfront instructions detailing exactly how each service should be structured.

D. Begin in direct execution mode and only switch to plan mode if you encounter unexpected complexity during implementation.

> [!question]- Show answer
> **Correct answer: A.**
> Plan mode is designed for complex tasks involving large-scale changes, multiple valid approaches, and architectural decisions — exactly what monolith-to-microservices restructuring requires. It enables safe codebase exploration and design before committing to changes.
> - **B is wrong**: it risks costly rework when dependencies are discovered late.
> - **C is wrong**: it assumes you already know the right structure without exploring the code.
> - **D is wrong**: it ignores that the complexity is already stated in the requirements, not something that might emerge later.

## Question 2

A production alert fires with the following stack trace:

```text
Traceback (most recent call last):
  File "src/orders/checkout.py", line 88, in apply_discount
    pct = coupon.percentage / 100
AttributeError: 'NoneType' object has no attribute 'percentage'
```

The on-call engineer confirms the cause: `apply_discount` is called with `coupon=None` when the customer has no coupon, and the intended behavior is to return the original price. The engineer asks Claude Code to fix it. Which approach is most appropriate?

A. Start the session with `claude --permission-mode plan` so Claude can map every caller of `apply_discount` before proposing a fix

B. Use direct execution: provide the stack trace and expected behavior and let Claude apply the guard in `apply_discount`

C. Use the Explore subagent to summarize the `orders` module first, then switch to plan mode to design the fix

D. Use plan mode because it is a production incident, and production changes should always be planned first

> [!question]- Show answer
> **Correct answer: B.**
> A single-function fix with a clear stack trace and a known cause is the textbook case for direct execution: the problem, location, and solution are all clear, so planning adds nothing.
> - **A is wrong**: over-planning a well-scoped change adds unnecessary overhead.
> - **C is wrong**: there is no verbose discovery phase to isolate — the location is already known.
> - **D is wrong**: the mode is chosen by ambiguity, not by urgency or perceived risk. A production incident with a known fix is still unambiguous.

## Question 3

Your team must migrate from `requests` to `httpx` across a service. A quick grep shows 47 files importing `requests`, some using sessions, some using streaming responses, and a few with custom retry adapters. Which workflow best fits this task?

A. Use direct execution file by file, since each individual file change is small and well understood

B. Use plan mode for the entire task, including applying the changes, so that no file is modified without a plan

C. Use plan mode to identify all affected files, map API differences, and design a single migration pattern including edge cases; then switch to direct execution to apply that pattern file by file

D. Use direct execution on a few representative files first, and enter plan mode only if inconsistencies appear across files

> [!question]- Show answer
> **Correct answer: C.**
> This is the plan-then-execute hybrid pattern: plan mode investigates and designs a consistent strategy for a multi-file migration; direct execution then implements the decided strategy. It is "plan THEN direct", not "plan OR direct".
> - **A is wrong**: without an upfront strategy, the migration is applied inconsistently across 47 files, and edge cases (sessions, streaming, retry adapters) surface late.
> - **B is wrong**: plan mode doesn't write to disk; once the strategy is decided, implementation is a well-defined change that belongs in direct execution.
> - **D is wrong**: the multi-file scope and the edge cases are already known — waiting for problems to emerge is the reactive anti-pattern.

## Question 4

A product manager asks for "a way to let partners receive order updates." After a short discussion, the team identifies three viable options: partners poll a REST endpoint, the platform pushes webhooks, or partners subscribe to a managed message queue. Each option has different infrastructure requirements (rate limiting, retry/delivery guarantees, new cloud resources). A developer argues: "It's just one feature and the code is simple, so direct execution is fine." What's the best response?

A. Agree — the feature is not technically difficult, so direct execution is appropriate

B. Use direct execution but include a detailed prompt describing the webhook approach, since it is the most common solution

C. Use plan mode to explore how orders and integrations are currently built, evaluate the three approaches with their infrastructure trade-offs, and propose one before any code is written

D. Ask Claude to implement all three approaches in direct execution and keep the one that works best

> [!question]- Show answer
> **Correct answer: C.**
> The mode is decided by ambiguity, not difficulty. Multiple valid approaches with different infrastructure requirements is a defining trigger for plan mode, even when the eventual code is simple.
> - **A is wrong**: it confuses "not difficult" with "not ambiguous".
> - **B is wrong**: it commits to an approach without exploring the codebase or evaluating alternatives.
> - **D is wrong**: implementing every option is the costly rework that planning is meant to prevent.

## Question 5

During a multi-phase refactoring task, a developer asks Claude in the main session to "list every file under `src/`, show all functions that touch the payment gateway, and print the full dependency graph." After this, Claude's answers during implementation become noticeably less precise, and it starts forgetting constraints stated early in the session. What is the most effective fix for future tasks like this?

A. Run the discovery phase with the Explore subagent, so the verbose output stays isolated and only a summary returns to the main conversation

B. Restate the constraints at the end of every prompt during implementation

C. Switch to direct execution for the discovery phase, since it is faster than plan mode

D. Split the discovery output into several smaller prompts in the same main session

> [!question]- Show answer
> **Correct answer: A.**
> The Explore subagent isolates verbose discovery output (file listings, dependency graphs, code excerpts) and returns summaries, preserving the main context window for implementation and preventing context window exhaustion.
> - **B is wrong**: it treats the symptom; the context window is still filled with discovery noise.
> - **C is wrong**: execution mode doesn't address where the verbose output lands.
> - **D is wrong**: smaller prompts in the same session still accumulate the same volume of output in the main context.

## Question 6

A developer wants Claude to propose a design for reorganizing the project's module system without modifying anything yet. In an ongoing session, they type:

```text
> Please work in plan mode and don't edit anything: reorganize src/ into feature-based modules.
```

Claude immediately starts moving files. What should the developer have done?

A. Add "IMPORTANT:" before the instruction so Claude treats it as a hard constraint

B. Actually switch into plan mode — press `Shift+Tab` until the status bar shows plan mode, or start the prompt with `/plan` — because writing "plan mode" in the prompt body doesn't activate it

C. Add a rule to `CLAUDE.md` saying "always plan before editing"

D. Nothing — plan mode can only be enabled when starting a new session with `--permission-mode plan`

> [!question]- Show answer
> **Correct answer: B.**
> Plan mode is something you switch into, not something you ask for. The three routes are `claude --permission-mode plan` at session start, `Shift+Tab` during a session, or a `/plan` prefix on a single prompt. Writing "in plan mode" in the body of a prompt does nothing.
> - **A is wrong**: emphasis in prompt text still doesn't change the permission mode, so files can still be written.
> - **C is wrong**: a `CLAUDE.md` instruction is guidance, not the mode that guarantees nothing is written to disk until the plan is approved.
> - **D is wrong**: `Shift+Tab` and `/plan` both work within an ongoing session.

## Question 7 (Multiple-response — select the two tasks that should use direct execution)

A team lead is triaging the sprint backlog for Claude Code. Which **two** tasks are best handled with direct execution? *(Select two.)*

A. Add a check in `create_booking()` that rejects `end_date` earlier than `start_date`

B. Decide whether the reporting service should read from a read replica or from a new analytics warehouse, each requiring different infrastructure

C. Change `MAX_UPLOAD_MB` from `10` to `25` in `config/settings.py`

D. Replace the ORM across the persistence layer (~60 files) with a different library

E. Split the `users` module into separate authentication and profile services with new API contracts

> [!question]- Show answer
> **Correct answers: A and C.**
> Both are well-scoped changes with a known location and a known approach: adding a validation conditional to one function and updating a configuration value.
> - **B is wrong**: choosing between integration approaches with different infrastructure requirements is an architectural decision → plan mode.
> - **D is wrong**: a library migration across many files needs a consistent strategy → plan mode, then direct execution.
> - **E is wrong**: defining service boundaries and API contracts has downstream consequences → plan mode.

---

> [!tip] Repasa en [[1 resumen]] y [[3 cuestionario]], o revisa el código en [[2 example]]
