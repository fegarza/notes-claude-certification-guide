---
tags:
  - claude-cert/dominio-5
  - task-statement/5.6
---

# 4 test — Information Provenance & Multi-Source Synthesis (CCAR-F Practice)

Practice exam covering **Task Statement 5.6: Preserve information provenance and handle uncertainty in multi-source synthesis** (Domain 5, `examguide.pdf`). Scenario-based questions in the style of the real exam, set in the **Multi-Agent Research System** scenario. Answers are hidden — commit to an answer before revealing.

> [!info] About official questions
> None of the "9. Sample Questions" in `examguide.pdf` maps 1:1 to Task Statement 5.6 (Questions 7–9 share the research-system scenario but assess task decomposition, error propagation and tool scoping). Question 1 below is adapted from the practice scenario in the study guide (claudecertificationguide.com, lesson 5.6); the rest are original questions built on the TS 5.6 "Knowledge of" and "Skills in" items and on Exercise 4 of the exam guide.

---

## Question 1 (Adapted — study guide practice scenario, lesson 5.6)

A multi-agent research system produces a synthesis report on market trends. Two credible sources report different growth rates: Source A reports 12% growth (2023 data) and Source B reports 8% growth (2024 data). The synthesis agent currently selects the more recent value. What is the correct approach?

A. Flag the conflict and escalate it to a human researcher for resolution before including either of the two figures in the final report.

B. Average the two values and report 10% growth, with a footnote recording the variance between the two sources.

C. Always use the most recent source, since the later publication date makes it the more reliable reflection of current market conditions.

D. Annotate both values with source attribution and publication dates, letting the consumer decide how to interpret the difference.

> [!question]- Show answer
> **Correct answer: D.**
> When two credible sources disagree, the correct handling is to preserve both values with full attribution (source, publication date, data period) so the consumer sees the complete picture. The dates are also what let the reader interpret the difference correctly — it may reflect different reporting periods or methodologies rather than an error.
> - **A is wrong** — escalation is one option the *coordinator* may choose for specific conflicts, but making it the default blocks the report and withholds two credible, attributable figures that can simply be presented side by side.
> - **B is wrong** — averaging produces a number no source reported and hides the disagreement; a footnote does not undo the false certainty of the headline figure.
> - **C is wrong** — "most recent wins" is still an arbitrary selection; it destroys information and presents false certainty, which is exactly the current (buggy) behavior.

---

## Question 2

Your research system has two subagents (web search, document analysis) that each return well-cited findings. Reviewers complain that the final reports contain sentences like *"Investment in renewable energy has grown significantly in recent years"* with no figures, no sources and no dates. Inspecting intermediate outputs, you confirm that each subagent's findings include source URLs and excerpts. The synthesis agent's prompt is: *"Combine the findings below into a clear, concise report."* What is the most effective fix?

A. Require the subagents' findings to be passed as structured claim-source mappings and instruct the synthesis agent to preserve and merge those mappings so every claim in its output is traceable to a specific source, via inline citations or a structured reference section.

B. Add a post-processing agent that reads the final report and runs web searches to find a supporting source for each sentence, appending the citations it finds.

C. Increase `max_tokens` for the synthesis call so the agent has room to include more detail from each subagent's findings.

D. Ask the web search subagent to return fewer, higher-quality findings so the synthesis agent has less material to compress.

> [!question]- Show answer
> **Correct answer: A.**
> The logs show attribution is present upstream and lost at synthesis — the most common failure point. The synthesis agent naturally compresses and paraphrases; its prompt must explicitly require that claim-source mappings are preserved and merged so each claim remains traceable.
> - **B is wrong** — retrofitting citations after the fact doesn't restore the original provenance; it may attach a *different* source that happens to agree, and it cannot recover figures and dates already paraphrased away.
> - **C is wrong** — output length isn't the problem; with the same prompt, the agent will still paraphrase without carrying mappings forward.
> - **D is wrong** — it blames an upstream agent that is working correctly and doesn't change the synthesis behavior that drops attribution.

---

## Question 3

A document analysis subagent extracts company financials for a due-diligence report. For `annualRevenue` it finds **$4.2M** in the "Annual Report 2023" (audited, fiscal year ending December 2023) and **$3.8M** in an "SEC Filing Q4 2023" (preliminary, unaudited, calendar year 2023). How should the analysis subagent handle this?

A. Report $4.2M, because audited figures are more reliable than preliminary ones, and mention the SEC filing in a footnote.

B. Stop processing the document and immediately escalate to a human analyst, since conflicting financial data means the extraction cannot be trusted.

C. Complete the analysis with both values included and explicitly annotated (source and context for each, plus a possible explanation such as audited vs preliminary and fiscal vs calendar periods), leaving the coordinator to decide how to reconcile before synthesis.

D. Omit the `annualRevenue` field from the output, since including an unresolved value would propagate uncertainty to downstream agents.

> [!question]- Show answer
> **Correct answer: C.**
> The analysis agent should finish its work with the conflict intact and annotated; reconciling belongs to the coordinator, which can then present both values, investigate further, or escalate to a human analyst.
> - **A is wrong** — even with a sensible-sounding heuristic, the analysis agent is arbitrarily resolving a conflict it doesn't own, hiding a credible second value from the coordinator.
> - **B is wrong** — halting throws away the rest of the analysis; escalation is a coordinator decision that should be made with the annotated conflict in hand, not a reason to abandon the document.
> - **D is wrong** — silently dropping the field destroys information and prevents any downstream reconciliation; the coordinator never learns there was a conflict.

---

## Question 4

Your synthesis agent includes a "data consistency check." In testing, it flagged a contradiction between a 2023 industry report (adoption rate 8%) and a 2024 report (adoption rate 12%) and **suppressed both figures** from the final output as "unreliable." Subagent outputs currently contain `claim`, `sourceUrl` and `relevantExcerpt`. What change best addresses the root cause?

A. Instruct the synthesis agent to be less strict and only flag contradictions when values differ by more than 50%.

B. Require subagents to include publication or data collection dates in their structured outputs, preserve them through synthesis, and present them alongside the figures so temporal differences are interpreted correctly.

C. Configure the synthesis agent to keep only one source per metric, choosing the one with the highest relevance score, to avoid contradictions altogether.

D. Move the consistency check into the research subagents so contradictions are removed before they ever reach synthesis.

> [!question]- Show answer
> **Correct answer: B.**
> Without dates, 8% vs 12% looks like a contradiction; with dates, it's a trend (adoption accelerated). Temporal data must be required in structured outputs so the synthesis agent can distinguish trends from true conflicts instead of flagging or suppressing consistent findings.
> - **A is wrong** — a numeric threshold is an arbitrary heuristic that doesn't use the information that actually explains the difference (when the data was collected).
> - **C is wrong** — keeping one source per metric is arbitrary selection; it destroys information and would still lose the trend.
> - **D is wrong** — it relocates the same mistake earlier in the pipeline and discards data before anyone can interpret it.

---

## Question 5

A research report on the electric-vehicle market is generated by a synthesis agent that renders **every section as bulleted lists**: the five-year investment figures, a narrative of recent regulatory news, and the battery-architecture findings. Executives say the investment section is hard to compare and the news section "reads like fragments." What should you change?

A. Render every section as tables so the report has a consistent, scannable format.

B. Render every section as prose paragraphs with clearer headings, since executives prefer narrative reports.

C. Keep bullets but add bold labels to each bullet to improve scannability.

D. Have the synthesis agent choose the rendering by content type: financial data as tables, news as prose, and technical findings as structured lists.

> [!question]- Show answer
> **Correct answer: D.**
> Synthesis should not flatten all content into one format. Financial comparisons read best as tables, news with cause-and-effect and chronology reads best as prose, and technical findings are clearest as hierarchical lists.
> - **A is wrong** — tables fix the financial section but break the narrative news section; it's still a uniform format.
> - **B is wrong** — prose makes numerical comparisons harder and loses the hierarchy of technical findings; again a uniform format.
> - **C is wrong** — cosmetic changes to bullets don't fix the mismatch between content type and format.

---

## Question 6

A stakeholder notices that your system's report states, with equal confidence, that "Model X reduces inference cost by 40%" and "Model X is the most widely adopted open model." You trace the first claim to three independent benchmarks and the second to a single vendor press release that described it as "likely the most widely adopted." How should the report be structured to address this?

A. Include explicit sections distinguishing well-established findings from contested or single-source ones, preserving the original source characterizations (e.g., "likely") and methodological context for each claim.

B. Remove the single-source claim entirely, since only findings with multiple independent sources should appear in a research report.

C. Add a numeric confidence score next to every claim, computed by the synthesis agent, and keep the current structure.

D. Rewrite both claims with hedged language ("may," "possibly") so the reader treats all findings with caution.

> [!question]- Show answer
> **Correct answer: A.**
> A finding supported by three independent sources differs from one based on a single report, even if the text presents both identically. Separate sections — plus the source's own characterization ("likely") and its methodological context — let the reader calibrate trust.
> - **B is wrong** — dropping attributable information is unnecessary; the fix is to present it with its actual level of support, not to delete it.
> - **C is wrong** — a synthesis-computed score replaces the sources' own characterization with the agent's opinion and still mixes established and contested findings in one undifferentiated list.
> - **D is wrong** — hedging everything uniformly erases the real difference in support and misrepresents the well-established finding.

---

## Question 7 (Multiple response — select TWO)

Your coordinator delegates to two research subagents whose findings go to a synthesis agent. Two problems recur: (1) reviewers can't verify which passage of which document supports a given claim, and (2) the synthesis agent treats figures from different years as contradictions. The subagents currently return `{ "claim": "...", "confidence": 0.0–1.0 }`. Which **two** changes to the subagents' structured output schema address these problems?

A. Add required `sourceUrl`, `documentName` and `relevantExcerpt` fields so each claim carries its source and the specific supporting passage.

B. Add a required `preferredValue` field in which each subagent selects the single most reliable figure when its sources disagree.

C. Add a required `publicationDate` field (publication or data collection date) so downstream agents can interpret temporal differences correctly.

D. Replace `confidence` with a free-text `notes` field where subagents can mention sources informally when they consider it relevant.

E. Have each subagent return a prose summary in addition to the JSON so the synthesis agent has more context to work with.

> [!question]- Show answer
> **Correct answers: A and C.**
> Together they make up the structured claim-source mapping: claim + source URL + document name + relevant excerpt + publication date.
> - **A** — fixes traceability: every claim is tied to where it was found and the exact passage supporting it.
> - **C** — fixes the temporal misreading: with dates, 8% (2023) vs 12% (2024) reads as a trend rather than a contradiction.
> - **B is wrong** — it pushes arbitrary conflict resolution into the subagents; conflicting values should be preserved and annotated, not collapsed.
> - **D is wrong** — optional, informal notes are exactly how provenance gets lost; the fields must be structured and required.
> - **E is wrong** — prose adds unstructured material that the synthesis agent will paraphrase, without adding traceable mappings.

---

## Question 8

Your pipeline is research subagent → analysis subagent → synthesis subagent → report generator. The research subagent outputs complete claim-source mappings. The report generator correctly renders whatever citations it receives. Final reports cite sources for only ~30% of claims. Logs show the analysis subagent returns objects like `{ "claim": "...", "assessment": "strong evidence" }`, and the synthesis subagent receives only these. Where is the defect, and what is the fix?

A. In the report generator — it should look up the original research outputs and re-attach citations to each claim before rendering.

B. In the analysis subagent — it must add its assessment while preserving the original claim-source mappings (source URL, document name, excerpt, date), so they can be merged downstream through synthesis.

C. In the research subagent — it should repeat the source information inside the text of each claim so that it survives later steps.

D. In the synthesis subagent — its prompt should instruct it to infer the most likely source for each claim from its own knowledge.

> [!question]- Show answer
> **Correct answer: B.**
> Attribution must survive every step. The analysis step is supposed to *add* assessment while preserving original mappings; here it replaces them, so synthesis never receives provenance to merge.
> - **A is wrong** — re-attaching at the end is a fragile retrofit: after analysis and synthesis have reworded and merged claims, matching them back to original findings is unreliable. Provenance must be carried forward, not reconstructed.
> - **C is wrong** — the research subagent is already correct; stuffing sources into claim text makes them part of what gets paraphrased away.
> - **D is wrong** — inferring sources from model knowledge fabricates provenance; the output would look cited but wouldn't be traceable to the documents actually used.
