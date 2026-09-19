---
tags:
  - claude-cert/dominio-2
  - task-statement/2.5
---

# 4 test — Built-in Tools (CCAR-F Practice)

Practice exam covering **Task Statement 2.5: Select and apply built-in tools (Read, Write, Edit, Bash, Grep, Glob) effectively** (Domain 2, `examguide.pdf`). Scenario-based questions in the style of the real exam. Answers are hidden — commit to an answer before revealing.

---

## Question 1

A developer needs to find every file that calls a deprecated function, `processLegacyOrder()`, and also every test file that exercises those callers. Which tool sequence is correct?

A. Bash with `find` and `xargs grep` for both steps, since a single shell pipeline can locate the callers and their test files in one pass without switching between built-in tools.

B. Glob for `**/*processLegacyOrder*` to find caller files, then Grep inside that result set for test files.

C. Grep for `processLegacyOrder` to find callers (this also surfaces tests that import the function directly), then Glob for the sibling test file of each caller (e.g. `**/OrderProcessor.test.*`) to catch tests that exercise the function through the source module without naming it.

D. Read all the source files to search for the function manually, then Read all the test files to pair them with their callers.

> [!question]- Show answer
> **Correct answer: C.**
> This is the Grep → Glob (→ Grep again if a wrapper is involved) pattern: content search finds direct references, then path matching finds the sibling test files by naming convention — including tests that exercise the function indirectly through the source module.
> - **A is wrong** — no filename contains `processLegacyOrder`, so a Glob-style pattern here would miss the function entirely; a shell pipeline doesn't change which tool is conceptually correct for content vs. path search.
> - **B is wrong** — Glob matches file paths, not contents; searching for `processLegacyOrder` in filenames returns nothing useful, since the function's name doesn't appear in any file path.
> - **D is wrong** — reading every source and test file upfront is the context-budget killer the guide explicitly flags; nothing here justifies reading files before Grep/Glob have identified which ones matter.

---

## Question 2

A developer wants to find every file where a function named `processLegacyOrder` is called. They run `Glob: "**/*processLegacyOrder*"` and get zero results, even though Grep confirms the function is called in three files. What's the most likely explanation?

A. The Glob pattern needs a leading `./` to search from the project root.

B. Glob matches file paths by naming pattern; since none of the three files are named `processLegacyOrder.ts` or similar, there's nothing for the pattern to match — the function name only appears inside file contents, which Glob doesn't search.

C. Glob has a hidden depth limit and the calling files are nested too deeply in the directory tree.

D. Glob only matches exact filenames, not wildcard patterns, so `**/*processLegacyOrder*` is invalid syntax.

> [!question]- Show answer
> **Correct answer: B.**
> Glob is a path-matching tool. It has no visibility into file contents, so a function call buried inside a file's body is invisible to it unless the filename itself happens to contain the search string. Grep is the correct tool for content search.
> - **A is wrong** — this misdiagnoses a syntax issue when the real problem is a fundamental mismatch between what Glob searches (paths) and what the developer is looking for (content).
> - **C is wrong** — invents a nonexistent limitation instead of recognizing the content-vs-path distinction.
> - **D is wrong** — Glob wildcard syntax like `**/*pattern*` is valid; the pattern is correctly formed, it's just searching the wrong dimension (names, not contents).

---

## Question 3

Edit is used to rename a variable inside `orders.ts`, but it fails with `old_string matches 3 locations` — the variable name appears in the function declaration, a comment, and one internal call. Per the exam guide, what is the documented fallback, and what should the developer avoid doing?

A. Retry Edit with `old_string` widened to include more surrounding context, since that's the officially documented recovery step.

B. Read the full file, then Write the complete modified version back with all three occurrences renamed; avoid defaulting to Read + Write for every future modification, since Edit should still be tried first.

C. Delete the file and recreate it from scratch with the corrected content, since that guarantees no stale occurrences remain.

D. Run three separate Bash `sed` commands, one per occurrence, since shell text substitution bypasses Edit's uniqueness requirement entirely.

> [!question]- Show answer
> **Correct answer: B.**
> The exam guide's documented fallback for a non-unique Edit match is Read + Write: read the full file, then write back the complete corrected version. This is a fallback, not the default — Edit should still be attempted first for every modification, since it's cheaper in context tokens.
> - **A is wrong** — widening the anchor (or using `replace_all: true`) is what current Claude Code documentation recommends as a cheaper move, but it is not the answer the exam guide keys to; the exam's documented fallback is Read + Write.
> - **C is wrong** — recreating the file from scratch is unnecessary and risky; it isn't a built-in tool behavior the guide describes at all.
> - **D is wrong** — this bypasses the built-in Edit/Read/Write tools entirely in favor of raw shell text substitution, which is not the tool-selection guidance the exam evaluates.

---

## Question 4

A developer is about to start investigating a bug in an unfamiliar 200-file codebase. They plan to `Read` every file in the repository first "to get full context" before making any changes. What is the most accurate assessment of this plan, and what should they do instead?

A. This is correct — full context up front prevents missing any relevant detail during the investigation.

B. This is the most costly exploration mistake the guide identifies — it consumes the context budget on files unrelated to the task. Instead, start with Grep to find entry points relevant to the bug, then Read only the files that search surfaces, expanding incrementally.

C. This is acceptable only if the codebase is under 500 files total, per the guide's stated threshold.

D. This is correct as a first pass, but only if followed immediately by a second pass using Glob to filter out irrelevant files.

> [!question]- Show answer
> **Correct answer: B.**
> Reading every file upfront is explicitly called out as the costliest exploration mistake: it swallows the context window on files that have nothing to do with the task. The correct approach is incremental — Grep for entry points, Read to follow the trail those entry points reveal, expanding only as justified.
> - **A is wrong** — "full context" achieved by reading everything is exactly the anti-pattern the guide warns against; it trades a mostly-irrelevant context window for the ability to skip targeted search.
> - **C is wrong** — invents a specific file-count threshold that isn't part of the guide's guidance; the principle is incremental discovery regardless of codebase size.
> - **D is wrong** — the damage of reading everything is already done by the time a second Glob pass could filter anything; Glob also isn't a tool for filtering already-loaded content.

---

## Question 5

While tracing callers of `processOrder()` (defined in `orders.ts`), a developer runs `Grep: "processOrder"` and finds 2 of 5 known consumers. Reading `orders.ts`'s barrel file, `utils/index.ts`, reveals `export { processOrder as submitOrder }`. What should the developer do next, and why did the first Grep miss the other 3 consumers?

A. Run `Glob: "**/submitOrder*"` to find files named after the re-exported name — Glob is appropriate here since the search is now for a specific known string.

B. Run `Grep: "submitOrder"` across the codebase — the first Grep missed those consumers because they import the function under its re-exported name, and the string `processOrder` never appears in their files.

C. Conclude the search is complete, since Grep already found the definition and the barrel re-export line, which confirms the function's full usage surface.

D. Run `Bash` with a `find` and `xargs grep -l submitOrder` pipeline, since Grep alone cannot search for a second pattern after already being used once.

> [!question]- Show answer
> **Correct answer: B.**
> The three missing consumers import the function via its barrel-re-exported name, `submitOrder` — that string, not `processOrder`, is what appears in their source. A second Grep for `submitOrder` is the correct next step to surface them, per the wrapper/barrel-tracing conclusion.
> - **A is wrong** — the goal is finding where `submitOrder` is *used* (content), not files *named* `submitOrder`; Glob would search paths, which is the wrong dimension for finding call sites.
> - **C is wrong** — finding the definition and the re-export line is not the same as finding every consumer; the whole point of this scenario is that indirect consumers are still missing.
> - **D is wrong** — nothing prevents running Grep a second time with a different pattern; reaching for Bash pipelines here abandons the built-in Grep tool without any reason to.

---

## Question 6

**(Official sample question style, per the exam guide's Task Statement 2.5 knowledge/skills list.)** A team wants to select all TypeScript test files under a `src/` directory that follow the naming convention `*.test.tsx`, regardless of which subdirectory they live in. Which built-in tool is correct, and why?

A. Grep, because it can search recursively through all subdirectories of `src/`.

B. Glob, with a pattern like `src/**/*.test.tsx`, because the task is matching file paths by naming pattern, not searching file contents.

C. Bash with `ls -R src/ | grep test.tsx`, because shell recursion handles nested directories more reliably than either built-in search tool.

D. Read applied to the `src/` directory itself, since Read can enumerate directory contents before filtering by extension.

> [!question]- Show answer
> **Correct answer: B.**
> This is a pure path/naming-pattern task — finding files by extension and naming convention, regardless of what they contain. That is precisely what Glob is built for, and `src/**/*.test.tsx` expresses "any depth of subdirectory" correctly.
> - **A is wrong** — Grep's recursion capability is irrelevant here; Grep searches file contents, not file names, and the task has nothing to do with what's inside the files.
> - **C is wrong** — this reimplements Glob's job with a fragile shell pipeline instead of using the built-in tool designed for exactly this kind of path matching.
> - **D is wrong** — Read loads file contents; it is not a directory-listing or pattern-matching tool, and using it this way misunderstands its purpose entirely.

---

> [!tip] Repasa el tema completo en [[1 resumen]], [[2 example]] y [[3 cuestionario]]
