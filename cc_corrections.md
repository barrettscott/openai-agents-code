# CC Corrections — oa Prose Review

WTW cells missed during the first pass. All need cutting to ≤2 sentences. Apply after the full course pass is complete.

---

## NB05 — Writing Agent Instructions

**WTW cell 1 (durable instructions)**

Current:
```
The durable instructions specify three things that the vague version left open:
- **Format** — `produce exactly one sentence`
- **Length** — `use plain language`
- **Stop condition** — `Do not add commentary or follow-up questions`
These are *completion criteria* — they tell the agent when it's done.
```

Replace with:
```
The durable instructions specify format, length, and a stop condition — completion criteria that tell the agent when it's done.
```

---

**WTW cell 2 (clarifying questions)**

Current:
```
Same instructions, two behaviors — depending on what the user provides:
- Missing information → triggers a question
- Complete information → agent proceeds without asking
The mechanism: list the specific required fields explicitly.
Without that list, "ask if unclear" produces inconsistent behavior.
```

Replace with:
```
Same instructions, two behaviors — list the specific required fields explicitly, and the agent asks only when they're missing.
```

---

## NB07 — Structured Outputs

**WTW cell (output_format)**

Current (3 sentences):
```
The SDK uses the Pydantic model to generate a JSON schema and instructs the model to conform to it.
`result.final_output` is now an instance of `SentimentResult`, not a string — descriptive field names like `reasoning` produce better results than short names like `rsn`.
Note: structured outputs guarantee *shape*, not truth — the model still generates the values, so validate facts separately if correctness matters.
```

Cut sentence 2 (naming tip). Keep sentences 1 and 3.

---

## NB08 — Error Handling & Recovery

**WTW cell (fallback strategy)**

Current (3 sentences):
```
The convention from Part 2 holds here: a failing tool returns a clear error string instead of raising.
The agent reads that string and follows the instructions to call the fallback — there is no automatic SDK mechanism.
Putting recovery logic in the instructions (rather than a Python try/except around the tool calls) means you can change the strategy by editing the prompt as you add or swap tools.
```

Cut sentence 3 (design guidance). Keep sentences 1 and 2.

---

## NB10 — Web Search

**WTW cell 1 (WebSearchTool)**

Current (3 sentences):
```
`WebSearchTool()` is a built-in tool — OpenAI runs the search, no API key needed.
This **grounds** the agent's answer in real retrieved content instead of training data.
The result: better accuracy on time-sensitive questions and less hallucination.
```

Cut sentence 3 — it restates what "grounds" already implies. Keep sentences 1 and 2.

---

**WTW cell 2 (citations)**

Current (3 sentences):
```
Citations may appear automatically, but they aren't guaranteed.
If your application requires citations, instruct the agent explicitly to cite its sources.
Look for inline URL references in the text — those are the citations the agent surfaced from search results.
```

Cut sentence 3 — the demo shows this. Keep sentences 1 and 2.

---

## NB11 — File Search

**WTW cell (file search retrieval)**

Cut sentence 2 ("Notice the third test — the agent refused to invent an answer.") — the demo shows this.

Keep: sentences 1 and 3.

---

## NB12 — Code Interpreter

**WTW cell (file upload)**

Cut sentence 3 ("The agent's code can then read it directly with pandas or the standard library.") — the demo shows this.

Keep: sentences 1 and 2.

---

## NB14 — Handoffs

**WTW cell (triage handoff)**

4 sentences — cut sentences 2 and 4. Keep:

```
The SDK automatically creates `transfer_to_billing_agent`, `transfer_to_tech_support_agent`, and `transfer_to_refunds_agent` tools for the triage agent.

Routing is prompt-driven, not enforced — clear, narrow triage instructions are what keep the agent from answering directly.
```

---

## NB16 — Parallel Execution

**WTW cell (asyncio.wait_for)**

Cut sentence 3 ("Always decide explicitly: skip the failed task, use a default, or abort the workflow.") — design guidance that doesn't belong in WTW.

Keep: sentences 1 and 2.

---

## NB17 — Debate & Critique

**WTW cell (debate pattern)**

Cut sentence 3 ("Parallelism here is an optimization carried over from Lesson 16…") — conceptual clarification the demo doesn't need.

Keep: sentences 1 and 2.

---

## NB19 — Sessions & Conversation State

**WTW cell 1 (session storage)**

Cut sentence 3 ("The agent always sees the complete context.") — restates sentence 1.

Keep: sentences 1 and 2.

---

**WTW cell 2 (session identity)**

Cut sentence 3 ("Use in-memory sessions for demos and short tasks…") — design guidance, not mechanism.

Keep: sentences 1 and 2.

---

**WTW cell 3 (shared session ID)**

Cut sentence 3 ("Tradeoff: all prior turns are visible to every agent…") — trade-off discussion belongs in a decision-guide cell, not WTW.

Keep: sentences 1 and 2.

---

## NB20 — Persistent Memory

**WTW cell 1 (SQLiteSession reload)**

Cut sentence 3 ("Change either value and you get a fresh session.") — implied by sentence 2.

Keep: sentences 1 and 2.

---

**WTW cell 2 (summarize → clear → store)**

Cut sentence 3 ("A separate summarizer agent keeps concerns clean…") — design guidance.

Keep: sentences 1 and 2.

---

**WTW cell 3 (curation mental model)**

Cut sentence 3 ("The correction works because of the instruction line…") — the code shows this.

Keep: sentences 1 and 2.

---

## NB21 — Vector Memory

**WTW cell (search_past_memories as tool)**

Cut sentence 3 ("In a real agent you'd typically pair this with an `add_memory` tool…") — forward-looking design guidance.

Keep: sentences 1 and 2.

---

## NB23 — Prompt Injection & Tool Safety

**WTW cell 1 (over-privileged vs read-only)**

4 sentences — cut sentence 2 ("The difference is the tool surface.") as filler. Still 3 sentences. Merge sentences 3 and 4:

```
Both agents receive the same injection — the over-privileged one can delete on success, the read-only one can't delete at all.

The bad decision is impossible, not just unlikely.
```

---

**WTW cell 2 (confirmation policy)**

4 sentences — cut sentences 3 and 4. Keep:

```
The confirmation policy makes dangerous actions visible — the user sees what the agent is about to do before it happens.

This reduces risk significantly, but note: enforcement here is instruction-level only.
```

---

## NB24 — Human in the Loop

**WTW cell 1 (needs_approval)**

Cut sentence 3 ("This is the reusable pattern: run until interrupted…") — the code shows this pattern.

Keep: sentences 1 and 2.

---

**WTW cell 2 (auto-approve vs escalation)**

Cut sentence 3 ("Your code inspects the parameters and decides which path to take.") — restates sentences 1 and 2.

Keep: sentences 1 and 2.

---

## NB26 — Capstone 3: Customer Service

**WTW cell (read vs write safety)**

Cut sentence 3 ("This is the read vs write safety principle from Lesson 23 applied in a production pipeline.") — back-reference that adds no mechanism.

Keep: sentences 1 and 2.

---

## NB30 — Project Structure & CLI

**WTW cell 1 (streaming)**

5 sentences — cut sentences 3, 4, and 5. Keep:

```
The agent definition is unchanged — same SDK, same tools.

What changes is how you consume the result: `Runner.run_streamed()` returns a stream you iterate over with `async for`, filtering for text-delta events.
```

---

**WTW cell 2 (FastAPI integration)**

Cut sentence 3 ("You're just changing the entry point.") — restates sentence 2.

Keep: sentences 1 and 2.

---

## NB32 — Deploying With Gradio

**WTW cell 1 (gr.ChatInterface)**

Cut sentence 2 ("Gradio manages the conversation loop, the message display, and the UI.") — obvious from context.

Keep sentences 1 and 3:

```
`gr.ChatInterface` expects a function that takes `message` and `history` and returns a string.

Your agent code doesn't change at all — you're just changing what calls it.
```

---

**WTW cell 2 (streaming)**

Cut sentence 3 ("The bridge is simple: accumulate the text, yield the growing string on each delta.") — the code shows this.

Keep: sentences 1 and 2.

---

**WTW cell 3 (error handling)**

Cut sentence 3 ("Expose the error type but not the full stack trace.") — design guidance that doesn't belong in WTW.

Keep: sentences 1 and 2.

---
