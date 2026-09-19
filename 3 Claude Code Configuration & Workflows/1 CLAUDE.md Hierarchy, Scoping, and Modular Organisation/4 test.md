Practice exam for **Task Statement 3.1: Configure CLAUDE.md files with appropriate hierarchy, scoping, and modular organization** (CCAR-F). Answers and explanations are hidden in collapsible callouts — try to answer before revealing.

## Question 1

Your team has been using Claude Code for two months. Developer A's sessions consistently produce API endpoints in kebab-case and follow a specific error-handling pattern. Developer B just joined the team, cloned the same repository on the same branch, and Claude Code generates endpoints in camelCase with inconsistent error handling. Both developers ran `claude` from the repository root. What is the most likely root cause?

A. The `.claude/CLAUDE.md` file at the project root is corrupted and needs to be regenerated

B. Developer B needs to run `/memory` to load the project's configuration files

C. Developer A's naming and error-handling conventions live in Developer A's user-level `~/.claude/CLAUDE.md` rather than in the project-level configuration

D. Directory-level `CLAUDE.md` files override project-level files, and Developer B's directory has no `CLAUDE.md`

> [!question]- Show answer
> **Correct answer: C.** User-level configuration (`~/.claude/CLAUDE.md`) is personal to each developer's machine and is never shared via version control. If Developer A's conventions were only defined there, Developer B — cloning the exact same repo and branch — would never receive them. The fix is to move those conventions into `.claude/CLAUDE.md` (project-level) so they are versioned and delivered to every teammate on clone.
>
> Option A is wrong because nothing in the scenario suggests corruption — the described symptom exactly matches a scoping issue, not a broken file. Option B is wrong because `/memory` is a diagnostic command; it does not load or activate configuration files — files load automatically based on their location regardless of whether `/memory` is ever run. Option D is wrong because directory-level `CLAUDE.md` files do not override project-level files; all discovered files are concatenated into context, and conflicts (if any) resolve arbitrarily rather than through a strict override hierarchy.

## Question 2

A team lead wants to split a growing `.claude/CLAUDE.md` file into smaller pieces without duplicating content across every package. They add this to the project-level `CLAUDE.md`:

```
Coding standards:
@./standards/naming-conventions.md
@./standards/error-handling.md
```

What best describes how these referenced files are loaded into Claude's context?

A. They are loaded lazily, only when Claude determines the content is relevant to the current task

B. They are loaded eagerly and inlined at read time, becoming part of the context as soon as the referencing `CLAUDE.md` is read

C. They are loaded only when the developer explicitly runs `/memory` to activate imports

D. They function identically to `.claude/rules/` files, applying conditionally based on glob patterns in YAML frontmatter

> [!question]- Show answer
> **Correct answer: B.** The `@` syntax (note: there is no `@import` keyword — just `@` followed by the path) references files that are loaded eagerly and inlined at read time. As soon as the `CLAUDE.md` containing the `@` reference is read, the referenced file's content is inserted inline, unconditionally.
>
> Option A is wrong because there is no relevance-based lazy loading for `@` references — it is unconditional. Option C is wrong for the same reason established in Question 1: `/memory` is diagnostic only and never activates loading. Option D is wrong because conditional, glob-pattern-based loading is the defining feature of `.claude/rules/` (covered under path-specific rules), not of `@` references, which always load in full regardless of the current file being touched.

## Question 3 (Select the two best answers)

Which two statements accurately describe how conflicting instructions across `CLAUDE.md` files at different levels are resolved? (Select two.)

A. Files are concatenated into the same context rather than one overriding another

B. The directory-level file always wins because it is loaded last

C. If two rules contradict each other, Claude may resolve the conflict arbitrarily

D. `settings.json` is consulted automatically to break any tie between conflicting `CLAUDE.md` rules

> [!question]- Show answer
> **Correct answers: A and C.** All discovered `CLAUDE.md` files load into the same context by concatenation — there is no override mechanism between levels. When their content contradicts, Claude may pick either instruction arbitrarily; there is no deterministic tiebreaker baked into the `CLAUDE.md` system itself.
>
> Option B describes a CSS-like specificity model that does not apply here: being read last (directory-level loads after project-level) does not mean it wins a conflict. Option D is a plausible-sounding distractor, but `settings.json` is a separate, client-enforced configuration layer for hard rules (like permissions or hooks) — it is not an automatic conflict-resolution mechanism for contradictory `CLAUDE.md` text.

## Question 4

A repository has test files scattered throughout the codebase next to the code they test (e.g., `Button.test.tsx` beside `Button.tsx`, in many different directories). The team wants every test file, regardless of its directory, to automatically receive the same testing conventions. Given only the tools discussed under CLAUDE.md hierarchy and modular organization, what is the key limitation that makes directory-level `CLAUDE.md` files a poor fit here?

A. Directory-level `CLAUDE.md` files can only be written by the project lead, not by individual contributors

B. Directory-level `CLAUDE.md` files apply only to their own directory and below, so they cannot uniformly reach files scattered across many unrelated directories

C. Directory-level `CLAUDE.md` files are not version-controlled, unlike project-level files

D. Directory-level `CLAUDE.md` files cannot use the `@` syntax to import shared standards

> [!question]- Show answer
> **Correct answer: B.** A `CLAUDE.md` placed in a directory scopes its content to that directory and its subdirectories — it has no reach into sibling or unrelated directories. Since the test files in this scenario are spread across many directories, you would need to duplicate (or maintain) a `CLAUDE.md` per directory, which does not scale. This is precisely the gap that path-based, glob-pattern rules (organized separately from directory-scoped files) are designed to close.
>
> Option A is false — any contributor with repo write access can add a directory-level `CLAUDE.md`; there's no such restriction described in the material. Option C is false — directory-level files, like project-level files, live in the repo and are version-controlled; only user-level files are excluded from version control. Option D is false — directory-level `CLAUDE.md` files can use `@` imports exactly like project-level files; nothing about being directory-scoped disables that syntax.

## Question 5

A team wants a rule that Claude Code must never run `git push --force` under any circumstances, with no exceptions, regardless of how the conversation is phrased. Where should this rule be enforced, and why?

A. In the project-level `CLAUDE.md`, written in bold with the word "NEVER," since project-level files reach the whole team

B. In `settings.json` or via a hook, because `CLAUDE.md` shapes behavior but is not a hard enforcement layer — the client enforces `settings.json` and hooks regardless of what Claude decides

C. In the user-level `~/.claude/CLAUDE.md` of every team member, so each person's personal preferences reinforce the rule

D. In `.claude/rules/` with a glob pattern matching all files, since rules take precedence over `CLAUDE.md`

> [!question]- Show answer
> **Correct answer: B.** `CLAUDE.md` (at any level) and `.claude/rules/` are all guidance that shapes Claude's behavior — none of them are a hard enforcement layer, and Claude can in principle deviate from any of them. A rule with zero tolerance for exceptions belongs in `settings.json` (permissions, enforced by the client with a strict precedence of managed policy > local > project > user) or a hook (bound to a specific lifecycle event, such as blocking a `git push --force` command before it executes).
>
> Option A is the classic trap: writing something forcefully in `CLAUDE.md` does not make it deterministic — it is still text that guides rather than enforces. Option C makes the problem worse, not better, since user-level configuration isn't even shared across the team via version control. Option D is wrong on a factual point: `.claude/rules/` files do not take precedence over `CLAUDE.md`, and like `CLAUDE.md`, they are guidance rather than a hard enforcement mechanism.

---

> [!tip] Review the concepts in [[1 resumen]], see them applied in [[2 example]], and drill recall in [[3 cuestionario]]
