# cc_nb11.md — 11_File_Search.ipynb

**Notebook:** `/Users/scott/Library/CloudStorage/Dropbox/Notebooks/openai-agents/Week_2_Reliability_And_Built_In_Tools/11_File_Search.ipynb`

Apply each change below to the notebook. Use NotebookEdit for cell edits.

---

## Change 1: Add plain-English explanation of "vector" and RAG in Part 1

3 of 5 reviewers (Anthropic, OpenAI, Grok) flagged that Part 1 drops four new terms (vector, chunk, embed, semantic search, RAG) in one paragraph with no plain-language anchors. This is the student's first exposure to RAG concepts.

**Find** (in the Part 1 "What Is a Vector Store?" cell — the RAG introduction sentence):
```
This pattern is called **RAG** — Retrieval-Augmented Generation. File Search handles all of it automatically.
```

**Replace with:**
```
This pattern is called **RAG** — Retrieval-Augmented Generation. In practice, that means the agent first **retrieves** relevant document chunks, then **generates** an answer using those chunks as evidence. A *vector* is a numerical representation of meaning — similar content ends up with similar numbers, letting the system find relevant passages even when the exact words don't match. File Search uses semantic search (matching by meaning) plus keyword search as a fallback, and handles all of it automatically.
```

---

## Change 2: Name the grounding payoff in Part 4 — third test

2 of 5 reviewers (Anthropic, Grok). This is the highest-leverage teaching moment in the notebook: the agent refuses to invent a color list because the retrieved chunks don't contain that information. The outline explicitly mandates this beat and the demo demonstrates it without naming it.

Add the following sentence to the Part 4 "Why This Works" cell, after the existing explanation of chunk retrieval:

```
Notice the third test — the agent refused to invent an answer. Without grounding, the model would likely guess plausible-sounding colors. With grounding, it can only answer from what was actually retrieved — which matters most when the agent is answering about private or high-stakes content.
```

---

## Change 3: Explain `max_num_results` and `vector_store_ids` in Part 4

3 of 5 reviewers (Anthropic, Gemini, Grok) flagged `max_num_results=3` as unexplained. 2 of 5 (Gemini, Grok) also flagged the plural `vector_store_ids`. Both belong in the same Why This Works cell.

Add the following to the Part 4 "Why This Works" cell (can be appended or integrated):

```
`max_num_results=3` caps how many document chunks the agent retrieves per query — keep it small to control context size and cost. `vector_store_ids` accepts a list, so a single agent can search across multiple knowledge bases at once — for example, a product FAQ and a user manual in the same turn.
```

---

## Change 4: Name the setup-vs-runtime distinction in Part 3

2 of 5 reviewers (Gemini, and flagged as a grouping by the synthesis). Students see setup and querying run together in the notebook and don't know whether to re-run setup every time their app starts.

> ⚠️ **Placement note:** The target paragraph in cell 14 contains "This is a one-time setup step." followed by additional sentences. Insert the new sentence **immediately after** the "This is a one-time setup step." sentence, not at the end of the paragraph — do not append after the "Vector stores act as..." sentence.

**Find** (in the Part 3 "Upload and Create a Vector Store" prose — the one-time setup sentence):
```
This is a one-time setup step.
```

**Replace with:**
```
This is a one-time setup step. In a real application, you'd run this setup code once, save the resulting `vector_store.id`, and reuse that ID in your agent code without recreating the store on every run.
```

---

## Change 5: Add the transfer recipe to Part 4 intro

1 of 5 reviewers (OpenAI), but it directly answers the student question "what do I change for my own project?" — the key transfer question for any demo.

Add the following sentence to the Part 4 intro cell, below "Now connect the vector store to an agent using `FileSearchTool`":

```
In your own project, the pattern is the same: upload your documents to a vector store, pass that store's ID into `FileSearchTool`, and rewrite the agent instructions for your domain.
```

---

## Change 6: Clarify demo vs. production lifecycle in the Cleanup cell

1 of 5 reviewers (OpenAI), but the gap is real: students don't know whether to always delete or to keep vector stores around. The fix adds one sentence.

**Find** (in the Cleanup cell prose):
```
Vector stores persist on OpenAI's servers until deleted. Run this cell when you're done to avoid ongoing storage charges.
```

**Replace with:**
```
Vector stores persist on OpenAI's servers until deleted. Run this cell when you're done to avoid ongoing storage charges. For demos, delete the store when you're done; in a real app, you'd usually keep the store and reuse its ID until the underlying documents change.
```

---

## Change 7: Add a Part 4 bridge sentence after the long setup sequence

2 of 5 reviewers (OpenAI, Grok). After three screens of vector store setup, students can lose sight of what the lesson is teaching. A one-sentence bridge reorients them before the payoff cell.

Add the following sentence as a markdown cell (or inline at the start of the Part 4 intro) before the agent construction code:

```
Now that the document is indexed, here's the key moment: we'll attach that store to an agent and watch it answer from the document instead of guessing from model knowledge.
```

---

## Change 8: Fix vocabulary inconsistency in Key Takeaways

1 of 5 reviewers (Anthropic). Part 1 says "converted into vectors"; Key Takeaways switches to "embedded" — the same concept, different word.

**Find** (in the Key Takeaways cell):
```
Files are chunked, embedded, and stored for semantic and keyword search
```

**Replace with:**
```
Files are chunked, converted into vectors, and stored for semantic and keyword search
```

---

## Change 9: Reframe the security note so it lands for demo users

1 of 5 reviewers (DeepSeek). The current security note says documents are "untrusted input," which confuses students who just wrote the document themselves. Reframing it around the production use case makes the principle apply.

**Find** (the security note cell after the three test queries):
```
⚠️ **Security note:** Retrieved document chunks are untrusted input and can enable prompt injection (see Lesson 23: Prompt Injection & Tool Safety). Treat them the same way you'd treat user input and don't pass them directly into system instructions without validation.
```

**Replace with:**
```
⚠️ **Security note:** In a real application, documents often come from user uploads or external pipelines — that's why retrieved chunks are treated as untrusted input and can enable prompt injection (see Lesson 23). Treat retrieved content the same way you'd treat user input; don't pass it directly into system instructions without validation.
```
