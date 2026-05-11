# cc_nb31.md — 31_Architecture_Decisions.ipynb

**Notebook:** `/Users/scott/Library/CloudStorage/Dropbox/Notebooks/openai-agents/Week_5_MCP_And_Production/31_Architecture_Decisions.ipynb`

Apply each change below to the notebook. Use NotebookEdit for cell edits.

---

## Change 1: Add Part 8 — Course Wrap-Up & Suggested Next Projects

5/5 reviewers flagged this as the top finding. The notebook ends at Part 7 with no wrap-up section and no project list. The Topics list promises "Course wrap-up and suggested next projects" — Part 8 is where that promise lands. The Practice Exercise references "a real or suggested project" but currently has no project list to draw from.

> ⚠️ NOTE: The exercise may not explicitly say "pick one of the six suggested projects from Part 8" — apply the change to deliver the missing wrap-up content regardless of exact exercise wording.

Add a new markdown cell after the Part 7 summary table and before the Practice Exercises header:

```markdown
## 🎁 Part 8: Course Wrap-Up & Suggested Next Projects

You've now covered the full arc: from single-agent tool use through multi-agent systems, safety, MCP, and production structure. The hard part was never learning what's possible — it was learning when to reach for each pattern and when to leave it on the shelf.

Pick one of these as your next build, or use a real project you care about:

- **Research paper reviewer** — agent reads papers from a folder, applies a judge-agent rubric, and outputs a ranked summary
- **Personal finance assistant** — MCP filesystem + a simple ledger tool; approval-gated for any write action
- **Meeting notes summarizer** — takes a raw transcript, extracts action items, and saves a structured summary to disk
- **Code review agent** — reads a diff or PR, runs tests, and posts a structured review; guardrail on off-topic requests
- **Customer feedback analyzer** — batch-processes feedback files, clusters themes, and outputs a prioritized report
- **Web research agent** — MCP web fetch + filesystem; approval-gated saves; streaming output so progress is visible

Start with the simplest version. Add complexity only when you have data showing you need it.
```

---

## Change 2: Add cost and quality hook after `MODEL_EVAL_TEMPLATE`

3 reviewers (Anthropic, OpenAI, Grok) flagged that the template tells students to compare quality, latency, and cost — but only shows latency, stubs quality with a comment, and has no cost hook at all. Students who try to apply this template to their own project will be stuck on two of the three metrics.

Add the following sentence immediately after the `print(MODEL_EVAL_TEMPLATE)` cell:

```
Replace the scoring comment with the judge-agent rubric pattern from Lesson 09 to produce a quality score per test case. Cost can be read from `result.usage` on each run — sum input and output tokens across the test set, then multiply by the model's per-token rate.
```

---

## Change 3: Fix the Guardrails row in Part 7 summary table

2 reviewers (Anthropic, Gemini) flagged that the current "Use when / Skip when" wording implies guardrails are reactive only — wait until you see bad inputs in production. This contradicts how guardrails were framed in the safety week (protective by design). The synthesis recommends reframing both cells.

**Find:**
```
| Guardrails | Bad inputs observed in real usage | Preemptive for every edge case before any real usage |
```

**Replace with:**
```
| Guardrails | Protecting against clear, high-impact risks (e.g., off-topic requests, PII leakage) | Trying to preemptively cover every edge case before you've seen any real usage |
```

---

## Change 4: Add `uvx` context in Part 5

2 reviewers (OpenAI, Gemini) flagged `uvx` appearing without context in the MCP tradeoffs section. A two-word parenthetical is all it takes.

> ⚠️ NOTE: The Find text below may have drifted. Apply semantically — locate the MCP tradeoffs section in Part 5 and find the sentence mentioning `uvx`, then apply the replacement.

**Find:**
```
requires Node.js or uvx
```

**Replace with:**
```
requires Node.js or uvx (a utility for launching Python-based MCP servers)
```

---

## Change 5: Add parallel-specialists dependency rule in Part 1

1 reviewer (OpenAI), but this is a genuine transfer gap: the bullet lists "independent subtasks" as the signal for parallel specialists but never says what "independent" means in practice. Students may mistakenly parallelize sequential pipelines. The synthesis recommends including this one.

Add the following sentence after the parallel specialists bullet list in Part 1:

```
If one subtask needs the output of another, it's not a good parallel-specialist candidate — run those sequentially instead.
```
