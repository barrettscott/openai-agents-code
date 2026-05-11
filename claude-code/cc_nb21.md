# cc_nb21.md — 21_Vector_Memory.ipynb

**Notebook:** `/Users/scott/Library/CloudStorage/Dropbox/Notebooks/openai-agents/Week_4_Memory_Safety_Observability/21_Vector_Memory.ipynb`

Apply each change below to the notebook. Use NotebookEdit for cell edits.

---

## Change 1: Explain how to make the vector store persistent

3 of 5 reviewers (Anthropic, Gemini, Grok) flagged this as transfer-blocking. Lesson 20 just taught that memory should survive restarts; the canonical vector memory pattern then uses `EphemeralClient()` with no explanation of what to change for persistence.

Add the following sentence after the `chroma_client = chromadb.EphemeralClient()` setup cell in Part 3:

```
For a real project, swap `EphemeralClient()` for `chromadb.PersistentClient(path='./chroma_db')` — the API is identical and the collection survives restarts. Ephemeral is used here only to keep the demo self-contained.
```

---

## Change 2: Document the shape of the `results` dict

2 of 5 reviewers (Gemini, Grok) flagged this as transfer-blocking. Students can copy the `results['documents'][0][0]` line but don't know what `results` looks like, so they can't adapt it (e.g., to get top-3 results or similarity scores).

Add the following inline comment immediately after the `.query(...)` call in Part 3's "Search the Memory" section:

```python
# results is a dict with 'documents', 'ids', and 'distances'; each is a list-of-lists, one inner list per query
```

---

## Change 3: Acknowledge the missing capture (write) half of the lifecycle

1 reviewer (Anthropic) flagged this as transfer-blocking. Lessons 19–20 captured new facts automatically; here all memories are pre-loaded. Students can't picture how their agent would remember new things told to it during a conversation.

Add the following sentence to the Part 4 "Why This Works" cell:

```
In a real agent you'd typically pair this with an `add_memory` tool — or write to the collection from your application code as conversations happen. This notebook focuses on retrieval; capture follows the same `collection.add()` pattern.
```

---

## Change 4: Make the Part 4 demo query require the stored fact

1 reviewer (Grok) flagged this as transfer-blocking — and DeepSeek raised the same observation just below the threshold. The current query ("What tech stack should I use for a data project?") doesn't visibly require stored memory, so students can't tell whether the vector lookup changed anything.

**Find** (in Part 4, "Run the Agent" cell):
```python
result = await Runner.run(memory_agent, input="What tech stack should I use for a data project?")
```

**Replace with:**
```python
result = await Runner.run(memory_agent, input="Based on what you know about Alex, what tech stack should I recommend for his data science project?")
```

Also add a `print(f"[Memory] Querying: {query}")` line inside the `search_past_memories` function body so the retrieval is visible when the demo runs.

---

## Change 5: Add two "adapt to your own project" sentences

2 of 5 reviewers (OpenAI, specifically F3 and F5) flagged this gap from different angles. Students can copy the code exactly but don't know what to change for their own project.

Add the following sentence after the collection creation cell in Part 3 (near the existing "In this demo, Chroma handles embeddings…" prose):

```
In your own project, you usually change only the collection name and the texts you store — Chroma handles the embedding step for both storage and search.
```

Add the following sentence to the Part 4 "Why This Works" cell (can be appended after Change 3 above):

```
Keep the pattern the same in your own projects: query the vector store, return only the top few relevant snippets, and let the agent reason over those instead of the whole database.
```

---

## Change 6: Add inline comment explaining `ids` parameter purpose

1 reviewer (Gemini) flagged this. Coming right after Lesson 19's emphasis on session IDs, students wonder what these IDs are for and whether they need special handling.

**Find** (in Part 3, the `collection.add(...)` call):
```python
ids=["id1", "id2", "id3", "id4"]
```

**Replace with:**
```python
ids=["id1", "id2", "id3", "id4"]  # each document needs a unique ID; use it later to update or delete a specific memory
```

---

## Change 7: Clarify what is doing the embedding and whether it costs money

2 of 5 reviewers (Anthropic, Grok) flagged this from different angles. Students see text being converted to vectors and worry about API charges.

**Find** (in Part 3, the markdown cell after the first code cell):
```
In this demo, Chroma handles embeddings for the text we add and query, so we can work with plain text instead of vectors directly.
```

**Replace with:**
```
In this demo, Chroma uses a small local embedding model that ships with the library — no API calls, no cost. For production you can swap in OpenAI's embedding model for higher quality; the rest of the code stays the same. This means we can work with plain text instead of vectors directly.
```

---

## Change 8: Add decision rule for vector memory vs. session memory

1 reviewer (OpenAI) flagged this. Students leaving Part 4 don't have a heuristic for when to choose this over the session-based approach from Lessons 19–20.

> ⚠️ NOTE: This overlaps somewhat with the Part 5 table — apply only if the Part 4 intro doesn't already have a decision rule.

Add the following sentence to the Part 4 intro:

```
Reach for this pattern when the agent only occasionally needs old notes, docs, or preferences — sending the full memory store on every turn would waste tokens and clutter the prompt.
```
