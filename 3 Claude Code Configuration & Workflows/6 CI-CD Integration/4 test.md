Claude Certified Architect (CCAR-F) practice exam for [[1 resumen]], targeting **Task Statement 3.6: Integrate Claude Code into CI/CD pipelines** from the official `examguide.pdf`. Includes official sample questions plus original questions in the same style. Each question presents a real-world case; pick the correct option before revealing the answer.

---

**Question 1** — *Official sample question from `examguide.pdf`, Scenario: Claude Code for Continuous Integration*

Your pipeline script runs `claude "Analyze this pull request for security issues"` but the job hangs indefinitely. Logs indicate Claude Code is waiting for interactive input. What's the correct approach to run Claude Code in an automated pipeline?

A. Add the `-p` flag: `claude -p "Analyze this pull request for security issues"`

B. Set the environment variable `CLAUDE_HEADLESS=true` before running the command

C. Redirect stdin from `/dev/null`: `claude "Analyze this pull request for security issues" < /dev/null`

D. Add the `--batch` flag: `claude --batch "Analyze this pull request for security issues"`

> [!question]- Reveal answer
> **Correct: A.** The `-p` (or `--print`) flag is the documented way to run Claude Code in non-interactive mode. It processes the prompt, outputs the result to stdout, and exits without waiting for user input — exactly what CI/CD pipelines require. The other options reference non-existent features (`CLAUDE_HEADLESS` environment variable, `--batch` flag) or use Unix workarounds that don't properly address Claude Code's command syntax.

---

**Question 2** — *Official sample question from `examguide.pdf`, Scenario: Claude Code for Continuous Integration*

Your team wants to reduce API costs for automated analysis. Currently, real-time Claude calls power two workflows: (1) a blocking pre-merge check that must complete before developers can merge, and (2) a technical debt report generated overnight for review the next morning. Your manager proposes switching both to the Message Batches API for its 50% cost savings. How should you evaluate this proposal?

A. Use batch processing for the technical debt reports only; keep real-time calls for pre-merge checks.

B. Switch both workflows to batch processing with status polling to check for completion.

C. Keep real-time calls for both workflows to avoid batch result ordering issues.

D. Switch both to batch processing with a timeout fallback to real-time if batches take too long.

> [!question]- Reveal answer
> **Correct: A.** The Message Batches API offers 50% cost savings but has processing times up to 24 hours with no guaranteed latency SLA. This makes it unsuitable for blocking pre-merge checks where developers wait for results, but ideal for overnight batch jobs like technical debt reports. B is wrong because relying on "often faster" completion isn't acceptable for blocking workflows. C reflects a misconception — batch results can be correlated using `custom_id` fields. D adds unnecessary complexity when the simpler solution is matching each API to its appropriate use case.

---

**Question 3**

A developer notices that their team's code review bot almost never flags meaningful design issues in code that Claude Code generated earlier in the same pipeline run. Reviewing the pipeline config, they find that both the "generate" and "review" steps use `claude -p -c` (resuming the same session) instead of two separate invocations. Why does this most likely explain the problem?

A. `-c` silently switches to a lower-quality model for the resumed turn.

B. The resumed session retains the reasoning Claude used to justify its own decisions while generating the code, making it less likely to question those decisions during review.

C. `-c` disables file-reading tools for any turn after the first.

D. The problem is unrelated to `-c`; it is caused by a missing `--max-turns` flag.

> [!question]- Reveal answer
> **Correct: B.** Reviewing code in the same session where it was generated is less effective than an independent review, because the session retains the reasoning and justifications from generation — a continuity bias. The fix is two fully separate `claude -p` invocations with no `-c`/`-r` between them. A, C, and D misdescribe what `-c` actually does.

---

**Question 4**

An automated PR review pipeline runs on every push to a branch. Developers report that every new push re-posts comments for issues that were already flagged in earlier pushes on the same PR — including some issues the team explicitly decided not to fix — and have started ignoring the bot entirely. What change would most effectively fix this?

A. Reduce how often the pipeline runs, e.g. once per day instead of every push.

B. Store the previous run's findings and include them in the next run's context, instructing Claude to report only new issues or previously-flagged issues that are still present in the code.

C. Turn off the review bot until the PR is ready to merge.

D. Increase `--max-turns` so Claude performs a more thorough review each time.

> [!question]- Reveal answer
> **Correct: B.** The root cause is that each run has no memory of prior runs. Passing prior findings into context, with explicit instructions not to repeat resolved or deliberately-unaddressed issues, is the documented fix for duplicate comments eroding developer trust. A, C, and D don't address the lack of context carried between runs.

---

**Question 5**

A team configures `anthropics/claude-code-action@v1` in a GitHub Actions workflow to review pull requests. The job runs successfully but Claude reports that it cannot find any project files, as if the repository were empty on the runner. Reviewing the workflow YAML, an early step appears to be missing. Which step is it?

A. `actions/setup-node@v4`

B. `actions/checkout@v6`

C. `actions/cache@v4`

D. A separate step to set `anthropic_api_key`

> [!question]- Reveal answer
> **Correct: B.** `actions/checkout` is the step that places the repository's contents on disk in the runner; without it, there is nothing on disk for Claude Code to read, regardless of any other configuration. A and C solve unrelated problems (language tooling, caching). D concerns authentication, not file availability on disk.

---

**Question 6**

A team documents testing standards, fixture locations, and severity criteria for security findings in their project's `CLAUDE.md`, expecting this to improve the quality of Claude Code's automated PR reviews in CI. A skeptical teammate argues this only helps in interactive/local use, since CI runs are non-interactive. Are they correct?

A. Yes — `CLAUDE.md` is only loaded in interactive sessions, so CI-invoked runs need the same context duplicated via `--append-system-prompt` instead.

B. No — `CLAUDE.md` is read identically in CI and interactive modes, so documenting project-specific context there directly improves CI-invoked reviews as well.

C. Yes — `CLAUDE.md` requires a `--load-context` flag that most CI configurations omit.

D. No, but only if the CI job is triggered manually rather than on a scheduled workflow.

> [!question]- Reveal answer
> **Correct: B.** `CLAUDE.md` is the documented mechanism for providing project context (testing standards, fixture conventions, review criteria) to CI-invoked Claude Code, and it is read the same way regardless of interactive vs. headless (`-p`) mode. A, C, and D invent constraints that don't exist.
