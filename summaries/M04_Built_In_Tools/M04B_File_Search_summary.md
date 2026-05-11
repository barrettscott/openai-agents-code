

---

# M04B: File Search — Recording Prep

## What this notebook teaches

This notebook teaches students how to give an agent access to their own private documents using OpenAI's built-in File Search tool, which uses vector search under the hood (a pattern called RAG — Retrieval-Augmented Generation). It matters because web search (M04A) only finds public information; File Search lets an agent answer questions grounded in private files like product FAQs, policies, or internal docs. The key outcome is that the agent answers from retrieved document chunks — not from its training data — and says "I don't know" when the answer isn't in the documents.

## Concepts to explain on camera

- **Vector Store**
  - **Term:** A specialized database that stores your documents as mathematical representations (vectors) so they can be searched by meaning, not just exact keyword matches. OpenAI hosts and manages it for you.
  - **Analogy:** Think of a library where the librarian doesn't just search the card catalog by title — they actually understand what each book is about and can find the one that answers your question, even if you phrase it differently than the book does.
  - **What students need to understand:**
    - You create a vector store once, upload files to it, and then any agent can query it
    - Vector stores persist on OpenAI's servers and cost $0.10/GB/day (first 1 GB free)
    - You must explicitly delete them when done or they keep charging
    - Vector stores are managed with the `openai` client, not the agents SDK

- **RAG (Retrieval-Augmented Generation)**
  - **Term:** A pattern where the agent first retrieves relevant chunks from your documents, then uses those chunks as context to generate an answer. The agent doesn't see the whole document — just the parts that match the query.
  - **Analogy:** Instead of memorizing an entire textbook, you flip to the relevant pages first, read those, and then answer the question based on what you just read.
  - **What students need to understand:**
    - File Search handles the entire RAG pipeline automatically — chunking, embedding, retrieval, injection into context
    - The agent only sees the most relevant chunks, not the full document — this is why it scales to large knowledge bases
    - "Grounded" means the answer comes from the retrieved content, not the model's training data
    - We build our own vector search from scratch in M06C — this is the managed/easy version

- **Chunking**
  - **Term:** The process of splitting a document into smaller pieces (chunks) so the vector store can index and retrieve just the relevant parts instead of the entire file.
  - **Analogy:** Instead of photocopying an entire 200-page manual for someone who asked one question, you photocopy just the two paragraphs that answer it.
  - **What students need to understand:**
    - OpenAI handles chunking automatically when you upload a file
    - The `max_num_results` parameter controls how many chunks come back per query
    - Smaller, well-structured documents with clear headings tend to chunk better

- **Grounding**
  - **Term:** Constraining the agent to answer based on specific retrieved content rather than its general training knowledge. When the answer isn't in the documents, a properly grounded agent says so instead of guessing.
  - **Analogy:** Like an open-book exam where you can only cite what's on the page in front of you — no making things up from memory.
  - **What students need to understand:**
    - Grounding is enforced through instructions ("Answer using only information from the provided documents")
    - The "color options" demo proves this works — the agent refuses to guess
    - This is the core value proposition of File Search over a plain LLM call
    - Grounding isn't perfect — prompt injection in documents can break it (covered in M07B)

- **`upload_and_poll`**
  - **Term:** A convenience method that uploads a file to a vector store, triggers processing (chunking + embedding), and waits until processing is complete before returning — all in one call.
  - **Analogy:** Like a one-click "upload and wait for confirmation" button, rather than uploading, then manually checking every few seconds if it's done.
  - **What students need to understand:**
    - This is a blocking call — it won't return until the file is ready to query
    - The agent can't search files that are still processing, so polling is essential
    - If the status comes back as `failed`, it's usually a file format issue
    - Plain text and PDF work reliably

## Key SDK pattern

The notebook introduces a two-step pattern: (1) create/populate a vector store with the `openai` client, then (2) wire it to an agent with `FileSearchTool`.

**Step 1 — Vector store creation and file upload:**

```python
vector_store = client.vector_stores.create(name="TechGadget FAQ")

with open(doc_path, "rb") as file_handle:
    file_batch = client.vector_stores.file_batches.upload_and_poll(
        vector_store_id=vector_store.id,
        files=[file_handle]
    )
```

- `client.vector_stores.create(name=...)` — creates an empty vector store on OpenAI's servers. The `name` is just a label for the dashboard; the important thing is the returned `vector_store.id`.
- `upload_and_poll(vector_store_id=..., files=[...])` — uploads one or more file handles, processes them (chunking + embedding), and blocks until done. The `files` parameter takes a list, so you can upload multiple files in one batch.
- Note this uses `client` (the raw `OpenAI` client), not the agents SDK. Vector store management is a platform operation.

**Step 2 — Agent with FileSearchTool:**

```python
agent = Agent(
    name="SupportAgent",
    instructions=instructions,
    model=MODEL,
    tools=[FileSearchTool(
        vector_store_ids=[vector_store.id],
        max_num_results=3
    )]
)
```

- `FileSearchTool` is imported from `agents` (the agents SDK), not from `openai`.
- `vector_store_ids` takes a **list** — you could point an agent at multiple vector stores if needed.
- `max_num_results=3` limits retrieval to the 3 most relevant chunks per query. Higher values give the agent more context but cost more tokens.
- The notebook also mentions `include_search_results=True` as an optional parameter to access raw retrieved chunks and source metadata, but doesn't demo it directly.

## The demos and what each shows

**Part 2 — Create a Sample Document:** This cell creates a fake "TechGadget Pro — Product FAQ" text file with sections on warranty, reset, firmware, returns, compatibility, and device description. It writes to `techgadget_faq.txt` locally. Emphasize on camera: this is just a plain text file. In a real project it could be PDFs, internal docs, anything. The fake product is intentional — the model has never seen this in training data, so any correct answer *must* come from the document.

**Part 3 — Upload and Create a Vector Store:** This is the infrastructure setup. The cell creates a vector store named `"TechGadget FAQ"`, uploads the file, and polls until processing completes. Point out the two-step flow on camera: create the store, then upload to it. Call attention to `file_batch.status` and `file_batch.file_counts.completed` in the output — these confirm everything worked. Mention that this is a one-time setup; once the store exists, you can query it as many times as you want.

**Part 4 — Query with FileSearchTool (three test queries):** This is the payoff. The first query ("What is the warranty policy?") proves the agent can retrieve and summarize specific information from the document. The second query ("How do I factory reset the device?") shows it handles a different section — the agent isn't just returning the first chunk every time. The **critical** third query ("What color options are available?") is the grounding test: the document says nothing about colors, so the agent should say it doesn't have that information. Emphasize this on camera — this is the whole point. A plain LLM would make something up. A grounded agent admits what it doesn't know. This is the key difference between File Search and just asking GPT a question.

**Cleanup cells:** Two cleanup steps: one deletes the local `techgadget_faq.txt` file before the exercises, and the final cell at the bottom deletes the vector store from OpenAI's servers. Stress the vector store deletion on camera — it costs money if you forget.

## Gotchas worth knowing before recording

- **Two different clients in one notebook:** Vector store operations use `client = OpenAI(...)` (the raw OpenAI client), but the agent itself uses `Agent` and `Runner` from the `agents` SDK. Students may be confused about why you need both. Name this explicitly on camera: "Vector store management is a platform operation; the agent SDK just connects to it."
- **`vector_store_ids` is a list, not a string:** Easy to accidentally write `vector_store_ids=vector_store.id` instead of `vector_store_ids=[vector_store.id]`. If you make this typo live, the error will be confusing.
- **`await Runner.run(...)` — this is async:** The notebook uses `await` for all `Runner.run` calls. Jupyter handles this natively, but don't accidentally drop the `await` or you'll get a coroutine object instead of a result. This was also the pattern in M04A, but worth being conscious of.
- **The "color options" test is non-deterministic:** The agent *should* say it can't find that info, but occasionally the model might still guess. If it does on camera, that's actually a great teaching moment about why grounding isn't a guarantee and why guardrails (M07A) exist.
- **Cleanup cell is at the very bottom, after exercises:** If you run the exercises, the vector store still exists and they'll work. But if a student runs the cleanup cell before the exercises, their exercise code will fail because the vector store is gone. Worth mentioning: "Don't run the cleanup cell until you're done with the exercises."
- **Exercise 1 reuses the existing `vector_store` variable:** Students add a second document to the *same* vector store. Exercise 2 creates a *new* vector store. Make sure the distinction is clear — same store vs. separate store is an important architectural choice.
- **The local file cleanup cell runs before exercises but after demos:** The `techgadget_faq.txt` file gets deleted, but the vector store (which is the one that matters) is still on OpenAI's servers. Students might be confused about why deleting the local file doesn't affect anything — the data lives in the cloud now.
- **Cost callout matters:** $0.10/GB/day plus $2.50/1000 calls. For this notebook it's trivially small, but mention it so students don't leave vector stores running across dozens of experiments.
- **`file_batch.file_counts.completed` — not `file_batch.files`:** The status check uses `file_counts.completed`, which is a count, not a list of files. Don't try to iterate over it.

## How it connects to adjacent notebooks

**What came before:** M04A (Web Search) introduced the first built-in tool — web search finds public information with no API key needed. On camera, callback: *"In the last notebook, web search gave our agent access to the live internet. But what if the answers live in your own private documents? That's what File Search solves."* Both M04A and M04B are "give the agent access to external knowledge" tools, but targeting different sources (public web vs. private files).

**What comes next:** M04C (Code Interpreter) completes the built-in tools trio by letting the agent write and execute Python in a sandbox — for data analysis, CSV processing, and generating outputs. On camera, forward reference: *"Now our agent can search the web and search our documents. Next, we'll give it the ability to actually run code."* Then M04D (Capstone #1) combines all three built-in tools into a single research agent.

**Longer-range connections:** The notebook explicitly notes that File Search uses vector search under the hood and that students will build their own vector search in M06C (Vector Memory with ChromaDB). Mention this on camera — it helps students understand that File Search is the "managed easy mode" and M06C is the "build it yourself" version. The security note about retrieved documents being untrusted input points forward to M07B (Prompt Injection & Tool Safety).

---