Practice exam for **Task Statement 3.2: Create and configure custom slash commands and skills** (CCAR-F). Answers and explanations are hidden in collapsible callouts — try to answer before revealing.

## Question 1 (Official sample question from the Exam Guide)

You want to create a custom `/review` slash command that runs your team's standard code review checklist. This command should be available to every developer when they clone or pull the repository. Where should you create this command file?

A. In the `.claude/commands/` directory in the project repository

B. In `~/.claude/commands/` in each developer's home directory

C. In the `CLAUDE.md` file at the project root

D. In a `.claude/config.json` file with a commands array

> [!question]- Show answer
> **Correct answer: A.** Project-scoped custom slash commands should be stored in the `.claude/commands/` directory within the repository. These commands are version-controlled and automatically available to all developers when they clone or pull the repo.
>
> Option B (`~/.claude/commands/`) is for personal commands that aren't shared via version control. Option C (`CLAUDE.md`) is for project instructions and context, not command definitions. Option D describes a configuration mechanism that doesn't exist in Claude Code.
>
> *(Source: official sample question, Exam Guide §9, "Scenario: Code Generation with Claude Code.")*

## Question 2

A developer is building a skill at `.claude/skills/audit-deps/SKILL.md` that scans every dependency file in a large monorepo and prints a detailed report of outdated packages, transitive dependency trees, and license findings for each one. Testing shows the skill's output alone consumes a significant fraction of the available context window, crowding out the rest of the conversation. Which frontmatter option should be added to `SKILL.md` to fix this, and why?

A. `argument-hint`, so the developer is prompted to scope the scan to fewer packages

B. `allowed-tools`, restricted to `Read` only, so the skill produces less output

C. `context: fork`, so the verbose scan output runs in an isolated sub-agent context instead of the main conversation

D. Splitting the skill into `.claude/commands/audit-deps.md` instead, since flat files produce less output

> [!question]- Show answer
> **Correct answer: C.** `context: fork` is designed exactly for this case: skills that produce extensive output (like this dependency scan across a whole monorepo) run in an isolated sub-agent context, so the main conversation's token budget is preserved.
>
> Option A doesn't address the mechanism at all — a hint might reduce scope if the developer types a narrower argument, but it does nothing to isolate output if a broad scan is still requested. Option B is a plausible-sounding distractor: restricting tools controls *what the skill can do*, not *where its output lands* — a `Read`-only skill can still print an enormous report into the main context. Option D is a fabricated distinction: switching between the flat and directory-based file structures does not change output volume or context isolation behavior; both create equivalent commands.

## Question 3

A skill's `SKILL.md` includes this frontmatter:

```yaml
---
description: "Refactor a module and write the changes to disk"
allowed-tools:
  - Read
  - Edit
  - Write
---
```

A teammate asks: "If I don't list `Bash` here, does that just mean Bash isn't pre-approved, so I'll get a permission prompt the first time the skill tries to use it?" What is the accurate way to describe what `allowed-tools` does here?

A. Yes — any tool not listed simply triggers a normal permission prompt when the skill first attempts to use it

B. `allowed-tools` restricts the skill's tool access to exactly the listed tools; tools outside the list are not available for that skill's execution, not merely gated behind an extra prompt

C. `allowed-tools` only affects autocomplete suggestions shown to the developer, not actual tool availability

D. `allowed-tools` is ignored unless `context: fork` is also set in the same frontmatter block

> [!question]- Show answer
> **Correct answer: B.** The Exam Guide frames `allowed-tools` as a mechanism that **restricts** tool access during skill execution — for example, limiting a skill to file-write operations to prevent destructive actions. Listing `Read`, `Edit`, and `Write` scopes the skill to exactly those tools; it is not simply a shortcut that avoids a prompt while still leaving other tools reachable.
>
> Option A understates the effect — this describes ordinary permission-prompt behavior, not what `allowed-tools` specifically does. Option C confuses `allowed-tools` with `argument-hint`, which is the option that affects autocomplete guidance. Option D is a fabricated dependency; `allowed-tools` and `context: fork` are independent frontmatter options that can be set separately.

## Question 4

Two developers on the same team both want quick access to a `/scaffold` skill that generates boilerplate for a new microservice. Developer A wants the team's standard version, shared via the repo, so every teammate gets identical scaffolding. Developer B additionally wants a personal variant that also generates a Dockerfile, but without changing what the rest of the team receives. What is the correct way to set this up?

A. Both developers edit the same `.claude/skills/scaffold/SKILL.md` file; Developer B adds the Dockerfile step behind a comment explaining it's optional

B. Developer A creates `.claude/skills/scaffold/SKILL.md` (project scope); Developer B creates a separate skill with a different name, such as `~/.claude/skills/scaffold-docker/SKILL.md` (user scope)

C. Developer B forks the entire repository so their modified `SKILL.md` doesn't affect Developer A

D. Developer B renames the shared file to `.claude/skills/scaffold-docker/SKILL.md`, replacing the team's original `/scaffold` command

> [!question]- Show answer
> **Correct answer: B.** The team-wide command belongs in project scope (`.claude/`) so it's version-controlled and identical for everyone. A personal variant belongs in user scope (`~/.claude/`) under a **different name**, so it doesn't collide with or silently alter the shared skill teammates rely on.
>
> Option A is the classic trap: editing the shared project-scoped file directly propagates Developer B's personal preference to the whole team on the next pull, which is not what either developer wants. Option C is wildly disproportionate — forking the repository to isolate one skill variant ignores that project vs. user scoping already solves this natively. Option D actively breaks the team's shared command by renaming and replacing it, which is the opposite of the stated goal of keeping Developer A's standard version intact.

## Question 5 (Select the two best answers)

Which two statements correctly distinguish when to use a skill versus when to use `CLAUDE.md`? (Select two.)

A. A procedure that should run only when a developer explicitly requests it, such as generating a release changelog, belongs in a skill rather than `CLAUDE.md`

B. Any convention followed by the entire team, regardless of how often it's needed, should go in `CLAUDE.md` so nothing is missed

C. A convention that must apply automatically in every single session, such as a commit-message format, belongs in `CLAUDE.md` rather than a skill

D. Skills and `CLAUDE.md` are interchangeable, since both can contain any kind of instruction and either one is loaded the same way

> [!question]- Show answer
> **Correct answer: A and C.** The determining factor is *when* the instruction should apply. On-demand, task-specific workflows (like changelog generation, triggered only for a release) belong in a skill. Always-on conventions that must shape every interaction (like a mandatory commit-message format) belong in `CLAUDE.md`.
>
> Option B is a common trap: "used by the whole team" is not the deciding factor — frequency of need is. A team-wide procedure that's only needed occasionally still wastes context if placed in `CLAUDE.md`, since `CLAUDE.md` loads in every session whether or not that procedure is relevant. Option D is false on the central distinction of this topic: skills load on demand, `CLAUDE.md` loads automatically and persistently — they are not interchangeable.

## Question 6

A skill file is created at `.claude/skills/lint-fix.md` (a flat Markdown file placed directly inside the `skills/` directory, with no subfolder). A developer expects `/lint-fix` to work exactly as if it were defined the canonical way. What is the most accurate assessment?

A. This works identically to `.claude/skills/lint-fix/SKILL.md` — both are valid canonical skill definitions

B. This is not a valid skill; the skills path requires a directory structure (`.claude/skills/<name>/SKILL.md`). A flat file belongs under `.claude/commands/<name>.md` instead

C. This works, but only if `context: fork` is added to the frontmatter

D. This works only in user scope (`~/.claude/`), not in project scope (`.claude/`)

> [!question]- Show answer
> **Correct answer: B.** The canonical skills structure requires a directory per skill containing a `SKILL.md` file. A bare `.md` file dropped directly into `.claude/skills/` does not follow that structure. If a flat file (no subfolder) is what's wanted, the correct location for that is `.claude/commands/<name>.md`, which is explicitly supported for backward compatibility.
>
> Option A is the core misconception this question tests — the two structures are not interchangeable file-for-file; the directory-based form specifically requires the nested `SKILL.md`. Option C invents a dependency between frontmatter options and basic path validity that doesn't exist. Option D invents a scope restriction that has nothing to do with the actual problem, which is structural, not about project vs. user scope.

---

> [!tip] Review the concepts in [[1 resumen]], see them applied in [[2 example]], and drill recall in [[3 cuestionario]]
