# cc_nb10.md — 10_Web_Search.ipynb

**Notebook:** `/Users/scott/Library/CloudStorage/Dropbox/Notebooks/openai-agents/Week_2_Reliability_And_Built_In_Tools/10_Web_Search.ipynb`

Apply each change below to the notebook. Use NotebookEdit for cell edits.

---

## Change 1: Bridge "hosted tool" and "grounding" to prior lessons in Part 1

2 of 5 reviewers (OpenAI, Grok) flagged that "hosted tool" arrives with no bridge from the `@function_tool` pattern students just learned, and "training data" does too much work without explanation. Both gaps land in the same sentence.

**Find** (in the Part 1 "Why This Works" cell):
```
`WebSearchTool()` is a built-in hosted tool — OpenAI runs the search on their side, no third-party API required. This is called **grounding**: the agent's answer is anchored to retrieved content rather than generated from training data alone.
```

**Replace with:**
```
Unlike the `@function_tool` decorators you wrote earlier, `WebSearchTool()` is a pre-built tool supplied by OpenAI that the agent can call like any other tool — OpenAI runs the search on their side, no third-party API required. The result is **grounding**: the answer is anchored to retrieved content rather than generated from training data alone (the information the model learned before this run, not live web results).
```

---

## Change 2: Clarify when search actually fires in Part 1

1 of 5 reviewers (Anthropic), but this is a genuine transfer blocker. The notebook sends three conflicting signals about what triggers a search (always-search instruction, query phrasing hint, "agent decides" takeaway). Students won't know how to control this in their own project.

Add the following sentence to the Part 1 "Why This Works" cell:

```
The agent decides whether to search based on the query and your instructions — telling it to "always search" or "search before answering" makes the behavior reliable; without that, it may answer from training data when it thinks it can.
```

---

## Change 3: Replace the citation guidance with a copyable pattern and explain inline URL references in Part 2

3 of 5 reviewers (Anthropic, OpenAI, Grok) flagged the citation guidance as insufficient. 1 reviewer (Gemini) also flagged the missing explanation of what inline URL references look like. Both changes target the same Part 2 "Why This Works" cell (cell 17) — apply them together in a single edit in the order shown below.

**Step 1 — Replace the citation guidance:**

**Find** (in the Part 2 "Why This Works" cell):
```
Citations may appear automatically, but they aren't guaranteed. If your application requires citations, instruct the agent explicitly to cite its sources.
```

**Replace with:**
```
Citations may appear automatically, but they aren't guaranteed. For more reliable citations, ask explicitly for a Sources section — for example: `"End every response with a Sources: list containing the full URL of each source you used."` Make this a requirement any time the answer will be used for research, reporting, fact-checking, or customer support — not casual chat.
```

**Step 2 — Immediately after the text inserted in Step 1, add:**

```
Citations often appear as bracketed numbers (e.g., `[1]`) referencing a sources list at the end, or as direct URLs within the text — those are the citations the agent surfaced from search results.
```

> ⚠️ Apply Step 1 and Step 2 together as a single edit to cell 17 to avoid one stomping the other.

---

## Change 4: Clarify what "bias" means and when to use location targeting in Part 3

2 of 5 reviewers (Anthropic, OpenAI) flagged "bias results" as vague, and 2 (OpenAI, Grok) flagged the missing guidance on when to reach for this in a real app. One sentence covers both.

Add the following sentence to the Part 3 "Why This Works" cell:

```
It prefers local results when relevant rather than strictly filtering — useful any time the right answer depends on where the user is (store hours, local services, regional pricing). In your own code, fill the location values from user input, profile data, or device/IP geolocation rather than hardcoding — and only add `user_location` when geography clearly changes the answer.
```
