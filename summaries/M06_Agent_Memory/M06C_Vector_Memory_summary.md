

---

# M06C: Vector Memory with ChromaDB — Recording Prep

## What this notebook teaches

This notebook teaches students how to give an agent the ability to retrieve memories by *meaning* rather than exact keywords, using ChromaDB as a lightweight vector database. It solves the problem where a user asks about "coding tools" but the stored memory says "Python and JavaScript" — exact-match lookup finds nothing, but semantic retrieval finds the right answer. Students build a working vector memory store, wire it to an agent as a tool, and learn when to choose vector memory over the SQLite approach from M06B.

## Concepts to explain on camera

### Vector Database
- **Term:** A database that stores text as lists of numbers (vectors) so it can find entries that are *similar in meaning* to a query, rather than requiring an exact word match.
- **Analogy:** A librarian who understands what you *mean* when you ask a question, rather than one who can only look up books by their exact title.
- **What students need to understand:**
  - It stores meaning, not just text — "programming tools" can match "Python and JavaScript"
  - It uses math (distance between vectors) to rank how relevant each stored item is
  - ChromaDB is one of many vector databases — it's lightweight and runs locally in your notebook
  - Unlike SQLite, there's no table schema or SQL — you add documents and query with natural language

### Embedding
- **Term:** A list of numbers (a vector) that represents the meaning of a piece of text. Two texts with similar meanings will have similar numbers, even if they use completely different words.
- **Analogy:** Like GPS coordinates for meaning — "happy" and "joyful" would be located near each other on the map, while "happy" and "server infrastructure" would be far apart.
- **What students need to understand:**
  - An embedding model (a separate AI model, not the chat model) converts text → numbers
  - ChromaDB handles this automatically — you pass in plain strings, it does the embedding behind the scenes
  - Both stored documents AND queries get embedded, then the database compares the vectors
  - You never see or manipulate the vectors directly in this notebook

### Semantic Retrieval
- **Term:** Finding stored information based on what it *means*, not which words it contains. A query for "food preferences" can retrieve a memory that says "Alex enjoys spicy cuisine" even though no words overlap.
- **Analogy:** Asking a friend "what does Alex like to eat?" — they don't search their memory for the exact phrase "like to eat," they recall anything *about* Alex's food preferences regardless of how they originally heard it.
- **What students need to understand:**
  - This is the core capability that vector memory adds over SQLite
  - It's probabilistic, not deterministic — results are ranked by similarity, not exact
  - You control how many results come back with `n_results`
  - Quality depends on how descriptive the stored memories are

### EphemeralClient
- **Term:** ChromaDB's in-memory mode — everything lives in RAM and disappears when the kernel restarts. No files are saved to disk.
- **Analogy:** A whiteboard that gets erased when you leave the room, vs. a filing cabinet (PersistentClient) that keeps everything.
- **What students need to understand:**
  - Perfect for demos and experimentation — no cleanup needed
  - For production, you'd switch to `chromadb.PersistentClient(path="./path")`
  - Re-running the notebook after a kernel restart means the collection is empty again
  - The troubleshooting section mentions `PersistentClient` as the upgrade path

### Collection
- **Term:** ChromaDB's equivalent of a database table — a named container where you store and query documents.
- **Analogy:** A labeled folder in a filing cabinet — "agent_memories" is one folder, "travel_prefs" could be another.
- **What students need to understand:**
  - Created with `chroma_client.get_or_create_collection(name="agent_memories")`
  - You add documents to it with `.add()` and search it with `.query()`
  - Each document needs a unique string ID (like `"id1"`, `"id2"`)
  - `get_or_create_collection` is safe to re-run; `create_collection` would throw a `UniqueConstraintError` on the second run

## Key SDK pattern

This notebook introduces two connected patterns: the ChromaDB store-and-query pattern, and the tool-wrapping pattern that connects it to an agent.

**Pattern 1 — ChromaDB Store and Query:**

```python
chroma_client = chromadb.EphemeralClient()

collection = chroma_client.get_or_create_collection(name="agent_memories")

collection.add(
    documents=[
        "Alex prefers Python for data science and JavaScript for web apps.",
        "The company uses Slack for internal chat and Zoom for meetings.",
        "The production servers are located in the US-East region.",
        "Standard shipping takes 3-5 business days."
    ],
    ids=["id1", "id2", "id3", "id4"]
)

results = collection.query(
    query_texts=[query],
    n_results=1
)
```

- `EphemeralClient()` — in-memory only, gone on kernel restart. No arguments needed.
- `get_or_create_collection(name=...)` — creates a new collection or reuses an existing one with that name. The `name` is just a string label.
- `documents=` — a list of plain text strings. ChromaDB embeds these automatically.
- `ids=` — a list of unique string identifiers, one per document. Must be the same length as `documents`. These are required — ChromaDB won't auto-generate them.
- `query_texts=` — a list of query strings (even for a single query, it must be a list: `[query]`). ChromaDB embeds this the same way it embedded the stored documents.
- `n_results=` — how many closest matches to return. Set to `1` for the initial demo, `2` for the agent tool.

**Pattern 2 — Wrapping ChromaDB as an Agent Tool:**

```python
@function_tool
def search_past_memories(query: str) -> str:
    """Search long-term memory for relevant facts or preferences."""
    results = collection.query(query_texts=[query], n_results=2)
    memories = results['documents'][0]
    return "Relevant memories: " + "; ".join(memories)
```

- The `collection` variable is captured from the outer scope — the tool function closes over it
- `results['documents'][0]` — the `[0]` is needed because `query_texts` accepts a list, so results are nested: one result set per query. Since we only pass one query, we always want index `[0]`
- The tool returns a plain string that gets injected into the agent's context

## The demos and what each shows

**Part 2 — Exact Match vs Semantic Retrieval:** This is a pure-Python demo with no ChromaDB yet. It takes four stored memory strings and runs a keyword search against the query `"What coding tools does the developer prefer?"`. The keyword search looks for words longer than 4 characters from the query in each memory. It finds nothing because "coding," "tools," and "developer" don't appear in the memory about Python and JavaScript. Emphasize on camera: this is the exact problem vector memory solves. The words are different but the meaning is the same. This motivates everything that follows.

**Part 3 — Building a Memory Store with ChromaDB:** This is the first hands-on ChromaDB code. Students create an `EphemeralClient`, build a collection called `"agent_memories"`, add the same four memories from Part 2, then query with `"What are the preferred programming tools?"`. The query returns `"Alex prefers Python for data science and JavaScript for web apps."` — the exact memory that keyword search missed. Emphasize: you pass in plain text both times (storing and querying) — ChromaDB handles the embedding math silently. Point out that `query_texts` takes a list (even for one query) and that `results['documents'][0][0]` is needed because of the nested list structure.

**Part 4 — Connecting Memory to an Agent:** This wires ChromaDB to an actual agent via `@function_tool`. The `search_past_memories` tool queries the same collection with `n_results=2`. The agent `MemoryAssistant` is given the instruction to use this tool when needed, then asked `"What tech stack should I use for a data project?"`. The agent should call the tool, retrieve the Python/JavaScript memory, and give a recommendation grounded in that memory. Emphasize: the agent decides when to call the tool — the memory isn't crammed into the prompt upfront. This is the scalable pattern: the memory store could have thousands of entries and the agent only pulls in the 2 most relevant.

**Part 5 — When to Use Vector Memory vs SQLite:** This is the comparison table, not a code demo. Walk through the table on camera. Key message: SQLite (from M06B) is great for exact facts like "user's timezone is EST" — you look up by session ID. Vector memory is for when you have a large, freeform knowledge base and the user's query phrasing won't match the stored text exactly. Many real agents will use both.

## Gotchas worth knowing before recording

- **`results['documents'][0][0]` double indexing:** This looks wrong but isn't. `query_texts` accepts a list of queries, so results come back as a list of lists — one inner list per query. Since we always pass one query, we always need `[0]` to unwrap the outer list, then `[0]` again for the first result. If students see `[0][0]` and get confused, explain the nesting.

- **`query_texts` must be a list:** Writing `query_texts=query` instead of `query_texts=[query]` will cause an error. Easy to do on camera — make sure the brackets are there.

- **`ids` are required and must be unique strings:** ChromaDB won't auto-generate IDs. If you re-run the `.add()` cell with the same IDs on the same collection, you'll get a `UniqueConstraintError` unless you used `get_or_create_collection`. The troubleshooting section covers this — worth mentioning proactively.

- **`EphemeralClient` resets on kernel restart:** If you restart the kernel mid-recording and try to query, the collection will be empty. You need to re-run the setup and `.add()` cells. This is by design but could look like a bug on camera.

- **Semantic search is probabilistic:** The "right" memory is usually returned, but ranking can shift slightly between runs or if you rephrase the query. Don't promise deterministic results — say "most relevant" not "the exact match."

- **ChromaDB uses its own default embedding model:** ChromaDB ships with a built-in lightweight embedding model (all-MiniLM-L6-v2 by default). You don't configure it and it's not the OpenAI embedding model. Students don't need to know the details, but if someone asks — it's not using GPT to create the embeddings.

- **The `collection` variable in the tool function is a closure:** The `search_past_memories` function references `collection` from the outer scope. If a student redefines `collection` to point to a different ChromaDB collection later (e.g., in exercises), the tool will silently query the new one. Not a problem in this notebook but worth knowing.

- **The security warning about retrieved documents:** The notebook flags that vector-retrieved content is unverified context and could contain adversarial text. Don't skip this — it's a forward reference to M07B and reinforces a pattern from M04A and M04B.

## How it connects to adjacent notebooks

**Builds on M06B (Persistent Memory):** Students already built SQLite-based persistent memory in M06B. Callback suggestion: *"In the last notebook, we stored facts in SQLite and retrieved them by session ID — exact-match lookup. That works great for structured data. But what happens when the user's question doesn't match the exact words you stored? That's the problem we solve today."* The comparison table in Part 5 explicitly contrasts SQLite sessions with vector memory, so students should have M06B fresh in their minds.

**Builds on M02B (Building Tools for Agents):** The `@function_tool` pattern from M02B is reused here to wrap ChromaDB as a tool. Students should be comfortable with this pattern by now, but worth a quick reminder: *"Same `@function_tool` decorator we've been using since Module 2 — the agent sees the docstring and decides when to call it."*

**Connects forward to M04B (File Search):** The notebook's intro references that OpenAI's File Search (M04B) uses vector search under the hood. If students already completed M04, callback: *"Remember File Search in Module 4? That was vector search managed by OpenAI. Now you're building your own."* If they haven't done M04 yet, this still works as standalone.

**Enables M07A (Guardrails):** The "Next Step" cell explicitly points to M07A. Transition suggestion: *"Now your agent has memory — both SQLite for exact facts and vector memory for semantic retrieval. But with power comes risk. In the next module, we add guardrails to control what the agent does with all this context."* The security note in Part 4 about retrieved documents being unverified context is a deliberate bridge to M07B (Prompt Injection).

---