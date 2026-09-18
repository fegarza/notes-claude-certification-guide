Claude Certified Architect (CCAR-F) practice exam for [[1 resumen]], targeting **Task Statement 1.2: Orchestrate multi-agent systems with coordinator-subagent patterns** from the official `examguide.pdf`. Each question presents a real-world case; pick the correct option before revealing the answer.

---

**Question 1 — Official sample question (examguide.pdf, Question 7, Scenario: Multi-Agent Research System)**

After running the system on the topic "impact of AI on creative industries," you observe that each subagent completes successfully: the web search agent finds relevant articles, the document analysis agent summarizes papers correctly, and the synthesis agent produces coherent output. However, the final reports cover only visual arts, completely missing music, writing, and film production. When you examine the coordinator's logs, you see it decomposed the topic into three subtasks: "AI in digital art creation," "AI in graphic design," and "AI in photography." What is the most likely root cause?

A. The synthesis agent lacks instructions for identifying coverage gaps in the findings it receives from other agents.

B. The coordinator agent's task decomposition is too narrow, resulting in subagent assignments that don't cover all relevant domains of the topic.

C. The web search agent's queries are not comprehensive enough and need to be expanded to cover more creative industry sectors.

D. The document analysis agent is filtering out sources related to non-visual creative industries due to overly restrictive relevance criteria.

> [!question]- Reveal answer
> **Correct: B.** The coordinator's logs reveal the root cause directly: it decomposed "creative industries" into only visual-arts subtasks, completely omitting music, writing, and film. The subagents executed their assigned tasks correctly — the problem is what they were assigned. A, C, and D incorrectly blame downstream agents that are working correctly within their assigned scope.

---

**Question 2**

A legal-research system uses a coordinator that assigns subtopics to search subagents. During a review of a report on "employment law changes in 2025," you find it thoroughly covers federal regulations but says nothing about state-level changes, even though every subagent's output was accurate and well-sourced for what it was asked to research. What is the most effective fix?

A. Add a post-processing step where the synthesis agent flags any topic area it suspects is underrepresented.

B. Expand the coordinator's decomposition logic so it explicitly partitions the topic to include state-level regulation alongside federal regulation before any subagent is invoked.

C. Instruct the web search subagent to broaden its queries to include state-level sources on its next invocation.

D. Add a fourth search subagent so more parallel research capacity is available.

> [!question]- Reveal answer
> **Correct: B.** This is the narrow-decomposition failure: the coordinator never assigned state-level regulation to any subagent, so no amount of subagent effort could have covered it. The fix belongs at the decomposition step, before any subagent runs. A treats a coordinator-level problem as a synthesis-level one. C blames a subagent that was never asked about state law. D adds capacity to a decomposition that is still incomplete — more subagents executing the same narrow assignment list changes nothing.

---

**Question 3**

A coordinator invokes a "market analysis" subagent, then later invokes a "recommendation" subagent, expecting the recommendation subagent to already know what the market analysis subagent found, because "they're part of the same research pipeline." The recommendation subagent's output ends up generic and disconnected from the actual market data gathered earlier. What is the root cause?

A. The market analysis subagent should have written its findings to a shared memory store that all subagents in the pipeline can read from.

B. Subagents do not automatically inherit context from other subagents or the coordinator's conversation history; the coordinator must explicitly pass the market analysis findings into the recommendation subagent's prompt.

C. The recommendation subagent's system prompt needs to be more detailed about what a "recommendation" is.

D. The two subagents should communicate directly so the recommendation subagent can query the market analysis subagent for its findings.

> [!question]- Reveal answer
> **Correct: B.** Subagents operate under complete context isolation — nothing is inherited automatically, and no shared memory or collective state exists between invocations. The coordinator is responsible for explicitly forwarding whatever context a subagent needs. A invents a shared-memory mechanism that does not exist in this pattern. C addresses a different, unrelated problem. D violates the hub-and-spoke rule that all communication must route through the coordinator.

---

**Question 4**

Your team is designing a coordinator for a competitive-intelligence system with four specialized subagents: pricing search, review search, patent search, and synthesis. Someone on the team proposes letting the pricing search subagent call the review search subagent directly whenever it finds a product it wants more detail on, to "cut out an extra round trip through the coordinator and reduce latency." What is the strongest reason to reject this proposal?

A. Direct subagent-to-subagent calls are technically impossible with the Claude API.

B. It would break the coordinator's observability, consistent error handling, and controlled information flow — the three benefits the centralized hub-and-spoke design exists to provide.

C. It would exceed the pricing search subagent's context window.

D. It is fine as a latency optimization as long as the coordinator is notified afterward.

> [!question]- Reveal answer
> **Correct: B.** Routing everything through the coordinator is what enables unified observability, centralized and consistent error handling, and coordinator-controlled information flow. Bypassing it for a latency win — even with good intentions — undermines all three, which is exactly the exam trap around proposing direct inter-subagent communication for efficiency. A is factually false as a blanket claim. C is unrelated to the proposal. D still describes a direct call that already happened outside the coordinator's control, so it doesn't fix the underlying issue.

---

**Question 5 — Multiple response (Select the two)**

A coordinator produces an incomplete competitive-analysis report: it covers only two of five known competitors. Investigation shows every subagent that ran returned accurate, well-sourced results for the competitors it was assigned. Select the **two** statements that correctly describe how to diagnose and fix this.

A. The root cause is most likely that the coordinator's decomposition step only assigned two competitors to subagents in the first place.

B. The fix is to have the synthesis subagent request additional competitor research directly from the search subagents when it notices thin coverage.

C. The fix is to update the coordinator's decomposition logic to identify and assign all five known competitors before invoking any search subagent.

D. The fix is to add two more search subagents so there is more parallel capacity for competitor research.

> [!question]- Reveal answer
> **Correct: A and C.** The pattern matches the narrow-decomposition failure: subagents performed correctly within their assigned scope, so the root cause sits in the coordinator's decomposition (A), and the fix is to correct that decomposition so it covers the full, known set of competitors before any subagent runs (C). B is wrong because it routes a coordinator-level responsibility (deciding what to research) through the synthesis subagent, and also implies subagent-to-subagent-style escalation outside the coordinator's control. D adds capacity to an assignment list that is still incomplete — it doesn't address the actual cause.
