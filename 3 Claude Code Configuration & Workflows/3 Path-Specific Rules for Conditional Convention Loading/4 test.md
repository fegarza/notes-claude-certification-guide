Practice exam for Task Statement 3.3 — *Apply path-specific rules for conditional convention loading*. Based on the concepts in [[1 resumen]].

## Question 1 (Official sample question — from `examguide.pdf`)

Your codebase has distinct areas with different coding conventions: React components use functional style with hooks, API handlers use async/await with specific error handling, and database models follow a repository pattern. Test files are spread throughout the codebase alongside the code they test (e.g., `Button.test.tsx` next to `Button.tsx`), and you want all tests to follow the same conventions regardless of location. What's the most maintainable way to ensure Claude automatically applies the correct conventions when generating code?

A. Create rule files in `.claude/rules/` with YAML frontmatter specifying glob patterns to conditionally apply conventions based on file paths

B. Consolidate all conventions in the root `CLAUDE.md` file under headers for each area, relying on Claude to infer which section applies

C. Create skills in `.claude/skills/` for each code type that include the relevant conventions in their `SKILL.md` files

D. Place a separate `CLAUDE.md` file in each subdirectory containing that area's specific conventions

> [!question]- Show answer
> **Correct answer: A.**
> `.claude/rules/` with glob patterns (e.g., `**/*.test.tsx`) allows conventions to be automatically applied based on file paths regardless of directory location — essential for test files spread throughout the codebase.
> - **B is wrong**: it relies on inference rather than explicit matching, making it unreliable.
> - **C is wrong**: skills require manual invocation or rely on Claude choosing to load them, contradicting the need for deterministic "automatic" application based on file paths.
> - **D is wrong**: it can't easily handle files spread across many directories, since `CLAUDE.md` files are directory-bound.

## Question 2

Your team's `terraform/` directory conventions (remote backend requirement, workspace usage, module versioning) currently live as a section inside the root `CLAUDE.md`. Developers report that Claude Code seems to "waste context" reviewing infrastructure rules even during pure frontend work sessions. What should you do?

A. Move the Terraform-specific conventions into a rule file at `.claude/rules/terraform.md` with `paths: ["terraform/**/*", "**/*.tf"]`

B. Shorten the Terraform section in `CLAUDE.md` so it consumes fewer tokens

C. Move the Terraform conventions into a skill so they only load when explicitly invoked

D. Split the root `CLAUDE.md` into one file per developer machine so each only sees relevant sections

> [!question]- Show answer
> **Correct answer: A.**
> Root `CLAUDE.md` loads for every session regardless of which files are being edited, so file-type-specific conventions (like Terraform's) waste tokens during unrelated work. A path-specific rule with a `paths` glob loads only when a matching file is actually being edited.
> - **B is wrong**: shortening the section reduces but doesn't eliminate the waste — it still loads on every session, including frontend-only ones.
> - **C is wrong**: a skill loads on-demand via invocation or intent-matching, not automatically whenever a `.tf` file is edited — it doesn't guarantee deterministic, always-on application by path.
> - **D is wrong**: per-machine files don't exist as a Claude Code mechanism, and doesn't solve the root problem (conventions loading regardless of file type).

## Question 3

A developer proposes creating a Skill called `apply-api-conventions` with `paths: ["src/api/**/*"]` in its frontmatter, arguing this will make Claude automatically follow API conventions whenever an API file is opened, exactly like a path-specific rule would. Is this a safe substitution?

A. No — skills load on-demand as task-style workflows triggered by intent-match or explicit invocation; they don't guarantee the same always-on, automatic loading that rules provide purely from a file path match

B. Yes — since both skills and rules support a `paths` field in frontmatter, they behave identically for automatic convention loading

C. No — skills cannot use glob patterns in their frontmatter at all, so the proposal is invalid syntax

D. Yes, but only if the skill is also referenced from the root `CLAUDE.md` to force it to load

> [!question]- Show answer
> **Correct answer: A.**
> Both skills and rules can declare a `paths` frontmatter field, which is exactly why this distractor is tempting — but a rule stays in context as background guidance the moment a matching file is read, shaping every edit deterministically. A skill remains a task-style workflow: even with `paths` present, its activation still depends on the model's intent-match or explicit invocation, not a guaranteed automatic load.
> - **B is wrong**: this is the classic Skills-vs-Rules exam pitfall — matching frontmatter fields don't imply matching activation guarantees.
> - **C is wrong**: skills can use `paths`-style frontmatter; the issue is activation semantics, not syntax validity.
> - **D is wrong**: referencing a skill from `CLAUDE.md` doesn't change how skills activate, and reintroduces the token-waste problem the rule was meant to avoid.

## Question 4 (Multiple-response — select the two correct statements)

Which **two** of the following are accurate consequences of using directory-level `CLAUDE.md` files (instead of a single path-specific rule) to enforce testing conventions across 50+ directories that each contain a co-located test file?

A. Any change to the testing convention requires manually updating 50+ files, risking drift where some copies fall behind

B. Token usage during a session editing a non-test file (e.g., a `.tf` file) increases, because all 50+ `CLAUDE.md` files load simultaneously

C. Every new directory that introduces a test file needs its own new copy of the conventions file

D. Claude Code will refuse to start a session if more than 10 duplicate `CLAUDE.md` files exist in the repository

> [!question]- Show answer
> **Correct answers: A and C.**
> Directory-level `CLAUDE.md` duplication means every copy must be kept in sync manually (A), and every new test-containing directory needs a fresh copy created (C) — this is exactly the maintenance failure mode path-specific rules eliminate.
> - **B is wrong**: directory-level `CLAUDE.md` files only load when Claude is working within that specific directory's scope, not all 50+ simultaneously regardless of location — that "always loads everywhere" problem describes the *root* `CLAUDE.md` pitfall, not the directory-level one.
> - **D is wrong**: no such hard limit or refusal behavior exists in Claude Code; duplication is a maintainability problem, not a technical error condition.
