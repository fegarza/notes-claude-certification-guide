Claude Certified Architect (CCAR-F) practice exam for [[1 resumen]], targeting **Task Statement 1.7: Manage session state, resumption, and forking** from the official `examguide.pdf`. Each question presents a real-world case; pick the correct option before revealing the answer.

---

**Question 1**

A developer spends day 1 auditing a 50-file codebase in a named session (`auth-audit`) and identifies three authentication issues in `auth.ts`, `session.ts`, and `middleware.ts`. That night, all three are fixed. On day 2, the developer runs `claude --resume auth-audit` and asks the agent to re-read the three modified files and confirm whether the issues are resolved. The agent gives a contradictory answer — flagging the session-expiration issue as still present while confirming the password-comparison fix. What is the most likely cause?

A. The `--resume` flag is malformed and should include `--fork-session` to load current file contents.

B. The conversation history restored by `--resume` still contains the stale tool results from day 1's reads of the unfixed files, and the agent is reasoning from a mix of those and the fresh re-reads.

C. The agent's context window is too small to hold both the day-1 findings and the day-2 file contents at once.

D. Named sessions expire after 24 hours, so `auth-audit` partially reset overnight, corrupting the history.

> [!question]- Reveal answer
> **Correct: B.** `--resume` restores the entire conversation history, including prior tool results. Asking the agent to re-read the modified files adds fresh reads but does not remove the stale ones already in history, so the model can reason from either depending on what it attends to — producing exactly this kind of inconsistency. A misunderstands what `--fork-session` does (it branches history, it doesn't refresh file contents). C misattributes the problem to context capacity rather than attention to stale vs. fresh data. D describes behavior sessions don't have — session history doesn't silently expire or corrupt.

---

**Question 2**

Continuing the same scenario, what is the most reliable way to get consistent, accurate advice about the three fixed files?

A. Resume `auth-audit` again and explicitly instruct the agent to "ignore any earlier information and trust only the latest file reads."

B. Start a completely new session and re-explore the entire 50-file codebase from scratch to rebuild full context.

C. Start a fresh session, inject a structured summary of the three prior findings plus the fact that they were fixed, and specify the three files for targeted re-analysis.

D. Use `fork_session` from `auth-audit` to create a clean branch that discards the stale tool results.

> [!question]- Reveal answer
> **Correct: C.** A fresh session with an injected summary contains no stale tool results at all — only the curated findings — so the agent verifies the current file state without any contamination from day-1 reads. A is the "insufficient fix" the guide explicitly calls out: an instruction to "ignore earlier information" doesn't remove that information from the context the model attends to. B wastes effort re-analyzing 47 files that never changed. D is a common trap: `fork_session` copies the existing history as-is, including its stale tool results — forking does not clean anything.

---

**Question 3**

A team completed a shared codebase analysis in one session and now wants to compare two independent approaches — a unit-testing strategy and an integration-testing strategy — both building on that same analysis, without either approach's exploration affecting the other. Which session management option fits, and why?

A. `--resume`, because both strategies need the full conversation history from the shared analysis.

B. Fresh start with summary injection, because starting clean avoids any cross-contamination between the two strategies.

C. `fork_session`, because it creates independent branches from a shared baseline, and changes in one branch don't affect or become visible to the other.

D. Neither — both strategies should be explored in the same session sequentially to keep costs low.

> [!question]- Reveal answer
> **Correct: C.** This is precisely the divergent-exploration use case the guide defines for `fork_session`: a shared baseline, two paths that must not see or affect each other. A ignores that plain `--resume` would have both explorations writing into and reading from the *same* history, not independent ones. B throws away the shared analysis baseline entirely, which isn't needed here since nothing is stale — the prior context is fully valid, just needs to branch. D would let the two strategies leak into each other's context within one linear conversation, defeating the point of independent exploration.

---

**Question 4**

A developer wants to resume investigating an open question from yesterday's session in the same directory, with no file changes and no need to branch. Which is the most direct way to continue, per the guide's naming conventions?

A. `claude --resume` with no arguments, which always picks the most recently modified file in the project.

B. `claude -c` (or `--continue`), which resumes the most recent conversation in the current directory without needing a session name.

C. `claude --fork-session` alone, which resumes the last session by default when no `--resume` is given.

D. Re-run the exact same prompt from yesterday to reconstruct the context manually.

> [!question]- Reveal answer
> **Correct: B.** The guide describes `-c` / `--continue` specifically as resuming the most recent conversation in the current directory — no session name required. A misdescribes `--resume`'s behavior (it needs a session name unless paired with `-c`'s shorthand behavior, not a "most recently modified file" heuristic). C is invalid: `fork_session` is meant to pair with `--resume` to branch a session, not to function as a standalone resume mechanism. D discards all prior context and forces expensive re-work, ignoring that continuation is available and appropriate here.

---

**Question 5** *(Multiple-response — select the two correct statements)*

A team is deciding how to handle a session where 30 of 50 codebase files were modified overnight due to a dependency upgrade, and they also want to explore two different migration strategies afterward. Which two statements are correct?

A. Because more than half the files changed, a summary-injection approach may be less valuable than a genuinely fresh, full re-exploration of the codebase.

B. `fork_session` should be used first, directly from the original stale session, to create two clean branches for the two migration strategies.

C. Once the codebase state is re-established without stale tool results, `fork_session` is the appropriate way to explore the two migration strategies independently from that clean baseline.

D. Resuming with `--resume` and asking the agent to "be careful about changed files" is sufficient given how many files changed.

> [!question]- Reveal answer
> **Correct: A and C.** A reflects the practical limit of targeted re-analysis: when the majority of a codebase changed, the summary-injection approach (designed for a handful of changed files) loses its efficiency advantage over just re-exploring fully. C correctly sequences the two techniques: fork for divergence only makes sense once the baseline itself is trustworthy — forking should happen *after* stale context is resolved, not before. B is the classic trap of forking directly from a session that still holds stale tool results, which both branches would inherit. D repeats the "insufficient fix" pattern — an instruction to "be careful" doesn't remove stale results already in history, and it's even less adequate here given how extensive the changes are.

---
> [!tip] Sigue con este tema
> Repasa la teoría en [[1 resumen]], refuerza con [[3 cuestionario]], y aplica el código en [[2 example]].
