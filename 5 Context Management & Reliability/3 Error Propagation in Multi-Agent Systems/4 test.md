Claude Certified Architect (CCAR-F) practice exam for [[1 resumen]], targeting **Task Statement 5.3: Implement error propagation strategies across multi-agent systems** from the official `examguide.pdf`. Each question presents a real-world case; pick the correct option before revealing the answer.

---

**Question 1** *(Official sample question — `examguide.pdf`, Question 8, scenario: Multi-Agent Research System)*

The web search subagent times out while researching a complex topic. You need to design how this failure information flows back to the coordinator agent. Which error propagation approach best enables intelligent recovery?

A. Return structured error context to the coordinator including the failure type, the attempted query, any partial results, and potential alternative approaches.

B. Implement automatic retry logic with exponential backoff within the subagent, returning a generic "search unavailable" status only after all retries are exhausted.

C. Catch the timeout within the subagent and return an empty result set marked as successful.

D. Propagate the timeout exception directly to a top-level handler that terminates the entire research workflow.

> [!question]- Reveal answer
> **Correct: A.** Structured error context gives the coordinator the information it needs to make intelligent recovery decisions — whether to retry with a modified query, try an alternative approach, or proceed with partial results. B's generic status hides valuable context from the coordinator, preventing informed decisions. C suppresses the error by marking failure as success, which prevents any recovery and risks incomplete research outputs. D terminates the entire workflow unnecessarily when recovery strategies could succeed. *(Official explanation.)*

---

**Question 2**

A customer support coordinator delegates order lookups to an `order_subagent`. During a 20-minute outage of the order database, several customers were told "We couldn't find any orders on your account." Logs show the subagent's handler:

```python
try:
    orders = orders_db.find(customer_id)
except TimeoutError:
    orders = []
return {"status": "success", "orders": orders}
```

What is the core problem and the correct fix?

A. The timeout is too short; increase it so the database has more time to respond during outages.

B. The subagent silently suppresses an access failure as a valid empty result. It should return `status: "error"` with `failureType: "transient"`, the attempted lookup, and alternatives, so the coordinator can retry or tell the customer the system is temporarily unavailable.

C. The coordinator should always re-run any lookup that returns zero orders, since an empty result may indicate a failure.

D. The subagent should raise the `TimeoutError` to the application's top-level handler so the whole conversation ends with a generic error page.

> [!question]- Reveal answer
> **Correct: B.** This is silent suppression: an access failure (the query never executed) is disguised as a valid empty result, so the coordinator confidently reports "no orders." The fix is to report it as an access failure with structured context. A treats a symptom and still suppresses future failures. C over-corrects by treating every valid empty result as suspect, wasting retries on customers who genuinely have no orders. D is workflow termination — disproportionate and loses the conversation.

---

**Question 3**

A research pipeline runs five subagents in parallel (market size, competitors, regulation, pricing, customer sentiment). The regulation subagent hits a permission error on a licensed legal database. The current implementation raises the exception, which aborts the pipeline; the other four subagents had already finished and their outputs are discarded. The team wants to fix this with minimal waste. What should they do?

A. Wrap the whole pipeline in a retry loop that re-runs all five subagents until all succeed.

B. Have the regulation subagent return an empty list with `status: "success"` so the pipeline continues.

C. Have the regulation subagent return structured error context (`failureType: "permission"`, attempted source and query, any partial results, alternatives such as public regulatory portals) so the coordinator keeps the other four results and decides on targeted recovery for the regulation topic only.

D. Increase the number of local retries on the regulation subagent to 10 before raising the exception.

> [!question]- Reveal answer
> **Correct: C.** The current design is workflow termination: a single failure throws away completed work. Structured error context lets the coordinator preserve the four successful results and decide a targeted recovery (alternative source, or proceed with a coverage annotation). A re-does completed work and won't fix a permission error. B swaps one anti-pattern for the other (silent suppression). D is pointless: permission failures don't resolve by retrying the same call, and the exception still kills the pipeline.

---

**Question 4**

A literature subagent queries a patents database for "solid-state sodium battery separators, 2015-2017". The database responds successfully in 300 ms with zero matches. The coordinator's recovery logic treats any empty result as a failure and retries three times with exponential backoff, then marks the topic as "source unavailable." What is wrong?

A. Nothing — retrying empty results is a safe default that guarantees completeness.

B. The retries should use linear rather than exponential backoff for empty results.

C. The subagent should have suppressed the empty result and omitted the topic from the report entirely.

D. The coordinator is conflating a valid empty result with an access failure. The query executed successfully and found nothing — that is the answer — so it should not be retried, and the report should state "no findings" rather than "source unavailable."

> [!question]- Reveal answer
> **Correct: D.** A valid empty result means the query ran and there were no matches; retrying wastes resources on a query that will always return nothing, and labelling it "source unavailable" misinforms the reader. A is the exact misconception the exam tests. B tunes the retry mechanics instead of removing an unnecessary retry. C hides a legitimate finding ("nothing exists in this range") instead of reporting it.

---

**Question 5** *(Select the TWO correct answers)*

You are designing the response schema that subagents return to the coordinator. Which two design choices best let the coordinator distinguish access failures from valid empty results and decide on recovery? **Select two.**

A. Return `status: "success"` with an empty `results` array for both timeouts and no-match queries, and log the difference server-side.

B. Return `status: "error"`, a `failureType` such as `transient` or `permission`, and `shouldRetry: true/false` when the data source could not be reached.

C. Return `status: "success"`, `results: []`, and `shouldRetry: false` with a message like "Query executed successfully. No matching records found." when the query ran with no matches.

D. Return a single string field `message` in all cases, letting the coordinator parse natural language to infer what happened.

E. Return only HTTP-like numeric codes (e.g., 200, 500) without the attempted query or partial results.

> [!question]- Reveal answer
> **Correct: B and C.** Together they make the distinction explicit in the response: access failures carry `status: "error"`, a failure type, and a retry signal; valid empty results carry `status: "success"` with empty results and `shouldRetry: false`. A is silent suppression — the coordinator can't see the server-side log. D forces the coordinator to guess from prose instead of structured fields. E is a generic status that hides the attempted query, partial results, and alternatives.

---

**Question 6**

A team's coordinator agent is becoming hard to maintain: it contains retry-with-backoff logic for every transient failure mode of every subagent (rate limits on the web search API, intermittent timeouts on the PDF parser, flaky connections to the citation service). What architectural change best addresses this, consistent with recommended error propagation strategy?

A. Remove all retry logic; any transient failure should be propagated immediately so the coordinator has full visibility.

B. Move local recovery into each subagent (retries with backoff, fallback sources) and have subagents propagate only errors they cannot resolve, including what was attempted and partial results.

C. Replace subagent errors with a single coordinator-level rule: if any subagent fails, restart the whole workflow from scratch.

D. Have subagents retry locally and, if still failing, return `{"status": "error", "message": "service unavailable"}` to keep the coordinator's interface simple.

> [!question]- Reveal answer
> **Correct: B.** Subagents should handle their own transient failures locally, which reduces coordinator complexity; only unresolved errors propagate, with structured context. A pushes every transient blip to the coordinator, increasing the complexity the team is trying to remove. C is workflow termination at scale. D gets local recovery right but ends in a generic status that hides the attempted action, partial results, and alternatives.

---

**Question 7**

A coordinator receives this from the web search subagent after its local retries were exhausted:

```json
{
  "status": "partial_failure",
  "failureType": "transient",
  "attemptedAction": {"tool": "web_search", "query": "AI in film production", "dateRange": "2020-2025"},
  "partialResults": [{"title": "Generative VFX in 2024 releases", "source": "trade journal"}],
  "alternativeApproaches": ["Retry with narrower date range (2023-2025)", "Search industry_reports index"]
}
```

The other subagents covered music, writing, and visual arts successfully. Which coordinator behavior is most appropriate?

A. Discard the film results and generate the report without a film section, since the data is incomplete.

B. Abort the report and ask the user to re-run the whole research task later.

C. Try one of the suggested alternatives for the film topic; if it still fails, proceed with the partial result and add a coverage annotation stating the film section is limited due to unavailable sources.

D. Treat the partial result as complete and write the film section with the same confidence as the other sections.

> [!question]- Reveal answer
> **Correct: C.** The structured context lets the coordinator pick a targeted recovery (the subagent's own alternatives), then proceed with partial results while transparently annotating the gap. A silently omits the topic, making the gap look like irrelevance. B is workflow termination, wasting three completed subtopics. D overstates support for a topic built on a single partial result — coverage annotations exist to distinguish well-supported findings from limited ones.

---

**Question 8**

Stakeholders reviewing an automated market research report complain that it "doesn't mention the APAC region at all," and assume the system judged APAC irrelevant. Logs show the APAC data provider returned permission errors throughout the run, which the coordinator correctly handled by continuing with the other regions. What change best prevents this misunderstanding?

A. Configure the coordinator to terminate the run whenever any regional data source is unavailable, so incomplete reports are never produced.

B. Structure the synthesis output with coverage annotations that mark each region as well-supported or limited/unavailable with the reason (e.g., "APAC: unavailable — data provider access denied during research").

C. Have the APAC subagent return an empty successful result so the report includes an "APAC: no data" line.

D. Ask the synthesis agent to fill the APAC section using general knowledge so the report looks complete.

> [!question]- Reveal answer
> **Correct: B.** Coverage annotations make the gap a known limitation rather than an apparent judgment of irrelevance — transparency is better than silent omission. A is workflow termination and discards valid work for other regions. C is silent suppression and would mislabel an access failure as a valid empty result. D fabricates content that isn't backed by sources and hides the real limitation.

---

**Question 9**

A subagent querying an internal analytics API receives `400 Bad Request: unsupported field 'regionCode'`. Its current local recovery logic retries every failure three times with backoff before propagating. Which response best reflects correct error propagation?

A. Keep retrying the identical request with longer backoff, since persistent errors often clear after enough time.

B. Return `status: "success"` with empty results, since the API is reachable and simply didn't return data.

C. Skip local retries for this error and immediately return structured error context with `failureType: "validation"`, the attempted request, any partial results, and an alternative such as using the supported field `region`.

D. Terminate the workflow, because a validation error indicates a bug that invalidates all other subagents' results.

> [!question]- Reveal answer
> **Correct: C.** The failure type determines the recovery path: a validation error won't succeed by retrying the identical request, so the subagent should propagate structured context — including the attempted request and a concrete alternative — so the coordinator can retry with a corrected query. A wastes retries on a non-transient failure. B is silent suppression: the query never actually executed successfully. D is a disproportionate workflow termination; other subagents' results are unaffected.

---
> [!tip] Sigue con este tema
> Vuelve a la teoría en [[1 resumen]], repasa con [[3 cuestionario]] y revisa el código en [[2 example]].
