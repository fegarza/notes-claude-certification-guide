---
tags:
  - claude-cert/dominio-2
  - task-statement/2.4
---

# 4 test — MCP Server Integration (CCAR-F Practice)

Practice exam covering **Task Statement 2.4: Integrate MCP servers into Claude Code and agent workflows** (Domain 2, `examguide.pdf`). Scenario-based questions in the style of the real exam. Answers are hidden — commit to an answer before revealing.

---

## Question 1

A developer wants to try out an experimental MCP server for a personal productivity tool they're evaluating — it's not something the rest of the team has agreed to use yet, and they don't want it appearing for anyone else who pulls the repository. Where should this server be configured?

A. In `.mcp.json` at the project root, with a comment noting it's experimental.

B. In `~/.claude.json` in the developer's home directory.

C. In a separate `.mcp.experimental.json` file at the project root, excluded via `.gitignore`.

D. In `.mcp.json`, but behind a feature flag that only enables it for that developer's username.

> [!question]- Show answer
> **Correct answer: B.**
> `~/.claude.json` is the user-level scope: personal, not version-controlled, and not shared with teammates — exactly right for an experimental server one developer is trying out before proposing it to the team.
> - **A is wrong** — `.mcp.json` is version-controlled and shared with everyone who clones or pulls the repository; a comment doesn't change that it would appear for the whole team.
> - **C is wrong** — this invents a non-existent configuration mechanism instead of using the scope Claude Code already provides for exactly this purpose.
> - **D is wrong** — there is no per-username feature-flag mechanism for `.mcp.json` entries; this describes infrastructure that doesn't exist.

---

## Question 2

A team's `.mcp.json` contains the following entry for their internal GitHub-hosted MCP server:

```json
{
  "mcpServers": {
    "github": {
      "url": "https://api.githubcopilot.com/mcp/"
    }
  }
}
```

Several developers report that the GitHub tools never appear in their agent's toolkit, even though the URL is reachable and correct. What is the most likely cause?

A. The `mcpServers` key should be renamed to `servers` for remote server entries.

B. The entry is missing `"type": "http"`, so Claude Code reads it as a stdio server, skips it, and expects a `command` instead.

C. Remote MCP servers require an explicit `"scope": "project"` field that is missing here.

D. GitHub's remote MCP endpoint requires a `command` and `args` wrapper even for HTTP servers.

> [!question]- Show answer
> **Correct answer: B.**
> A remote server entry needs both `"type": "http"` and a `url`. Without the explicit `type`, Claude Code interprets the entry as a stdio server (which needs `command`/`args`), finds none, and skips it — which matches the reported symptom exactly.
> - **A is wrong** — `mcpServers` is the correct key regardless of transport type; renaming it would break the configuration further.
> - **C is wrong** — scope is determined by which file the entry lives in (`.mcp.json` vs. `~/.claude.json`), not by a field inside the entry.
> - **D is wrong** — `command`/`args` is the local/stdio shape; a remote HTTP server like GitHub's uses `type` and `url` instead, not both shapes combined.

---

## Question 3

A team's `.mcp.json` currently has the GitHub authentication token committed directly in plain text: `"Authorization": "Bearer ghp_abc123..."`. A security review flags this. What is the correct fix, and why does it work even though `.mcp.json` stays shared and version-controlled?

A. Move the entire `github` server entry out of `.mcp.json` into each developer's `~/.claude.json`, duplicating it per developer.

B. Replace the literal token with `${GITHUB_TOKEN}` in `.mcp.json`, so each developer sets the actual value in their own local environment; the file only ever contains the variable name.

C. Base64-encode the token before committing it, so it's no longer readable as plain text in the repository.

D. Add `.mcp.json` to `.gitignore` going forward so future commits don't include it, and leave the historical commits as-is.

> [!question]- Show answer
> **Correct answer: B.**
> `${VARIABLE_NAME}` expansion is exactly the mechanism designed for this: the shared, version-controlled file references a variable name, not a value, so it stays safe to commit while each developer authenticates with their own locally-set token. Token rotation also then requires no config file changes.
> - **A is wrong** — this abandons the team-wide sharing that `.mcp.json` exists for, duplicating configuration that should stay centralized; it also doesn't solve the underlying problem of where the token itself is stored.
> - **C is wrong** — base64 is trivially reversible encoding, not encryption; it does not remove the secret from version control or protect it in any meaningful way.
> - **D is wrong** — `.gitignore` only affects future commits; it does nothing about the token already exposed in repository history, and it also stops the legitimate, safe parts of `.mcp.json` from being shared going forward.

---

## Question 4

An agent has access to both a `search_codebase` MCP tool (semantic, AST-aware code search) and the built-in Grep tool. Logs show the agent consistently uses Grep even for requests like "find the function that handles refund calculations," where semantic search would clearly perform better. The current `search_codebase` tool description reads only `"Searches code."` What is the most effective fix?

A. Remove the Grep tool from the agent's toolkit entirely so `search_codebase` is the only search option available.

B. Add a system prompt instruction stating "always prefer search_codebase over Grep for semantic queries."

C. Expand the `search_codebase` description to explain what it returns, how it differs from grep-style text matching, and when to use it instead of Grep.

D. Rename the tool from `search_codebase` to `grep` so the model recognizes it as the preferred search tool by name.

> [!question]- Show answer
> **Correct answer: C.**
> The root cause is that the model has richer context about Grep's capabilities than about the sparse `search_codebase` description. Enhancing the MCP tool's description — what it does, what it returns, and when to prefer it over Grep — gives the model the context it needs to make the better selection on its own.
> - **A is wrong** — removing Grep entirely is disproportionate and removes a tool the agent may legitimately need for genuine text-pattern searches, not just semantic ones.
> - **B is wrong** — a general system-prompt instruction competes against a built-in tool's already-detailed description; the guide's pattern for this exact failure is to fix the sparse description itself, not to patch around it with prompt instructions.
> - **D is wrong** — renaming the tool doesn't change what information the model has about its actual capabilities, and colliding the name with the built-in `grep` naming convention would create ambiguity rather than resolve it.

---

## Question 5

A team is integrating with Jira to let their coding agent read and update issue status during a workflow. No one on the team has evaluated what's already available; a developer immediately starts scaffolding a custom MCP server that wraps the Jira REST API. What's the correct first step per the guide's build-vs-use principle, and what would justify building custom instead?

A. Proceed with the custom build — building your own server always gives tighter control over exactly which Jira operations are exposed.

B. Evaluate existing community MCP servers for Jira first; build custom only if team-specific workflows or business logic exist that those servers can't handle.

C. Skip MCP entirely and call the Jira REST API directly from Bash commands, since that avoids server configuration altogether.

D. Build the custom server, but only expose read operations initially, expanding to writes once the team is confident it's stable.

> [!question]- Show answer
> **Correct answer: B.**
> Jira is a standard integration with maintained, tested, community MCP servers. The guide's build-vs-use principle is to evaluate those first; a custom build is only justified when the scenario describes team-specific workflows or proprietary logic the community server genuinely cannot handle — which nothing here indicates yet.
> - **A is wrong** — "tighter control" is not, by itself, a justification; the guide flags this exact reasoning as the common trap of building custom for a standard integration without checking community options first.
> - **C is wrong** — bypassing MCP entirely gives up the standardized tool/resource interface (descriptions, schemas, discovery) that MCP provides, and isn't what the build-vs-use decision is actually about.
> - **D is wrong** — this still commits to building custom without first evaluating whether a community server already covers the need; phasing the rollout doesn't address the missing evaluation step.

---

## Question 6

A team exposes their internal database to their coding agent through an MCP server. Currently, the only way for the agent to understand the database structure is to call `list_tables`, then call `describe_table` once per table — for a database with 40 tables, that's 41 tool calls before the agent can write a single useful query. What change would most directly address this, and how does it differ from adding more tools?

A. Add a `describe_all_tables()` tool that returns every table's schema in one call, keeping the information behind a tool call.

B. Expose the schema as an MCP **resource** (e.g. `db://schema/main`) that the agent has visibility into up front, rather than something it must call a tool to discover.

C. Cache the results of `list_tables` and `describe_table` after the first run so subsequent agent sessions skip the repeated calls.

D. Instruct the agent via the system prompt to always call `describe_table` for every table at the start of a session, so the cost is paid once per session instead of once per query.

> [!question]- Show answer
> **Correct answer: B.**
> This is precisely the distinction the guide draws between resources and tools: resources show agents what data is available without requiring an exploratory tool call at all, while tools remain for actions. A schema resource makes the structure available to the agent up front.
> - **A is wrong** — this still requires the agent to know to call a tool, and to spend a call doing it; it reduces the call count from 41 to 1 but doesn't adopt the resource pattern the guide identifies as the correct mechanism for content catalogs.
> - **C is wrong** — caching optimizes the existing tool-call pattern rather than replacing it with the mechanism designed for this exact problem (resources), and still requires the initial exploratory calls on a cold cache.
> - **D is wrong** — this still spends 41 tool calls per session; moving the cost to "once per session" via a prompt instruction doesn't eliminate the exploratory-call pattern the resource mechanism is designed to avoid entirely.

---

> [!tip] Repasa el tema completo en [[1 resumen]], [[2 example]] y [[3 cuestionario]]
