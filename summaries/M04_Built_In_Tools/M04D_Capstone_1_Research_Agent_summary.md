

---

# M04D: Capstone #1 — Research Agent — Recording Prep

## What this notebook teaches

This notebook teaches students how to combine all three built-in tools — web search, file search, and code interpreter — into a single agent that researches a topic, queries private documents, computes statistics, and produces a structured Pydantic report. It matters because real-world agents rarely use just one tool; this is the first time students see multi-tool orchestration where the agent itself decides which tool to call at each step. It also reinforces structured outputs from M03A-2 and error handling from M03B in a realistic end-to-end pipeline.

## Concepts to explain on camera

- **Vector Store (as used in this notebook)**
  - **Term:** A cloud-hosted collection on OpenAI's servers that stores your uploaded documents in a searchable format. When the agent uses file search, it queries this vector store to find relevant chunks.
  - **Analogy:** Think of it like a private search engine that only indexes the documents you've uploaded — the agent can search your internal files the same way it searches the web.
  - **What students need to understand:**
    - You create it with `client.vector_stores.create()`, upload files with `file_batches.upload_and_poll()`, and pass its ID to `FileSearchTool`
    - It persists on OpenAI's servers and costs money until you delete it — that's why the cleanup cell at the bottom matters
    - It's the same concept from M04B, now used inside a multi-tool agent
    - The `upload_and_poll()` method blocks until processing finishes — you must wait for this before running the agent

- **Multi-Tool Orchestration**
  - **Term:** Giving one agent access to multiple tools and letting it decide which tool to use at each step of a task, rather than you hard-coding the sequence.
  - **Analogy:** Like giving a research assistant access to Google, the company filing cabinet, and a calculator, then saying "write me a report" — they figure out what to pull from where.
  - **What students need to understand:**
    - The `tools=[]` list on the Agent can hold any combination of `WebSearchTool()`, `FileSearchTool()`, and `CodeInterpreterTool()`
    - The agent's instructions are what guide it to use the right tool at the right time — vague instructions mean it may skip tools
    - More tools = slower runs (20–40 seconds is normal for this notebook)
    - The agent may not use every tool on every run — that's expected behavior, not a bug

- **Pipeline (as used in the System Architecture)**
  - **Term:** A sequence of steps the agent follows: receive a topic → research it (web + docs) → analyze the data → produce a report. Not hard-coded in the SDK — it's driven by the instructions.
  - **Analogy:** Like an assembly line where the agent moves through stages, but it's the agent reading its instructions and deciding the order, not your code forcing it.
  - **What students need to understand:**
    - This is a single-agent pipeline — one agent, multiple tools, one `Runner.run()` call
    - The pipeline order is suggested by instructions, not enforced by code
    - In M05E, students will build a multi-agent pipeline where code explicitly controls each phase

- **Untrusted Input (security context)**
  - **Term:** Any data coming from web search results or retrieved document chunks that could contain malicious instructions designed to manipulate the agent's behavior.
  - **Analogy:** Like opening email attachments — the content might look normal but could contain something that tricks the agent into doing something unintended.
  - **What students need to understand:**
    - Web search results and file search results are both flagged as untrusted in this notebook
    - This is a forward reference to M07B (Prompt Injection & Tool Safety) — just plant the seed now
    - In production, you'd validate or sanitize tool output before passing it downstream
    - For this capstone, awareness is enough — the full treatment comes in Module 7

## Key SDK pattern

The primary pattern is a multi-tool `Agent` with `output_type` for structured output:

```python
research_agent = Agent(
    name="ResearchAgent",
    instructions=research_instructions,
    model=MODEL,
    output_type=ResearchReport,
    tools=[
        WebSearchTool(),
        FileSearchTool(
            vector_store_ids=[vector_store.id],
            max_num_results=3
        ),
        CodeInterpreterTool(tool_config={ 
            "type": "code_interpreter",
            "container": {"type": "auto"}
        })
    ]
)
```

Parameter breakdown:

- **`name="ResearchAgent"`** — Label that shows up in traces and debugging. Pick something descriptive.
- **`instructions=research_instructions`** — The long string defined above that tells the agent the exact 4-step process: web search → file search → code interpreter → synthesize into report sections. These instructions are doing the heavy lifting — they're what make the agent use all three tools rather than just one.
- **`model=MODEL`** — Set to `"gpt-5-mini"` — fast and cheap, appropriate for a research task that doesn't need deep reasoning.
- **`output_type=ResearchReport`** — The Pydantic model with `executive_summary`, `key_findings`, `data_analysis`, and `sources`. Forces the agent to return structured data instead of free-form text. This is the M03A-2 pattern applied to a real use case.
- **`tools=[...]`** — The list of all three built-in tools:
  - `WebSearchTool()` — no configuration needed, no extra API key
  - `FileSearchTool(vector_store_ids=[vector_store.id], max_num_results=3)` — points to the vector store created in Phase 1; `max_num_results=3` limits how many document chunks come back per query
  - `CodeInterpreterTool(tool_config={"type": "code_interpreter", "container": {"type": "auto"}})` — sandboxed Python execution; `"container": {"type": "auto"}` lets OpenAI handle the sandbox environment automatically

The agent is then run with the standard pattern:

```python
research_result = await Runner.run(research_agent, input=research_topic)
```

The `research_result.final_output` is a `ResearchReport` object (not a string) because `output_type` was set.

## The demos and what each shows

**Phase 1 — Build the Knowledge Base:** You create a fake internal survey document (`ai_adoption_survey.txt`) with specific statistics (73% adoption rate, 847 respondents, budget trends, etc.) and upload it to an OpenAI vector store. This is the private context that web search can't find — emphasize that on camera. The document is intentionally specific with numbers so you can later verify whether the agent actually retrieved from it versus hallucinating. The `upload_and_poll()` call blocks until processing is done. Point out that this is the same vector store pattern from M04B, just now feeding into a multi-tool agent.

**Phase 2 — Build the Research Agent:** You define the `ResearchReport` Pydantic model with four fields (`executive_summary`, `key_findings`, `data_analysis`, `sources`) and create the agent with all three tools. The instructions string is the star here — walk through each numbered step and explain that these instructions are what make the agent use all three tools rather than defaulting to just web search. Emphasize that `output_type=ResearchReport` means the output will be a structured object, not free-form text. Mention the security note about untrusted input — it's a brief callout, not a deep dive.

**Phase 3 — Run the Full Pipeline:** You run the agent with the topic `"Current state of AI adoption in enterprise — trends, barriers, and outlook"`. This is deliberately chosen to require both web search (current trends) and file search (internal survey data with the 73% stat). Call out that it takes 20–30 seconds — set that expectation before running. The output is printed through `truncate_response()` for readability. Then you show the error handling version: the same `Runner.run()` wrapped in `try/except` with a helpful error message. Explain that multi-tool agents have more failure points than single-tool agents — any one tool can fail and take down the whole run.

**Cleanup:** Two cleanup steps — deleting the local `ai_adoption_survey.txt` file (happens before exercises), and deleting the vector store from OpenAI's servers (happens at the very end). Emphasize the vector store deletion — it persists and costs money until you delete it.

**Exercises (walk through the prompts, don't solve):** Exercise 1 asks students to export the research report to a markdown file — straightforward file I/O using the structured output fields. Exercise 2 is the important one: it asks students to evaluate the research agent against a golden test set using the judge-agent pattern from M03C. The `golden_tests` list is pre-defined with specific `must_contain` criteria (like `"73%"` from the internal survey). This exercise drives home that evaluation isn't a one-module concept — it follows you into every real agent.

## Gotchas worth knowing before recording

- **The agent may not use all three tools every run.** Sometimes it skips code interpreter if it doesn't see a numerical computation need. If this happens on camera, don't panic — explain that the agent decides tool use based on the task, and point to the instructions as the lever to control this. The troubleshooting section even calls this out.
- **`research_result.final_output` is a `ResearchReport` object, not a string.** If you try to print it raw, it'll show the Pydantic object representation. The `truncate_response()` helper expects a string, so the SDK must be serializing it — but if it doesn't, you may need `str(research_result.final_output)` or access individual fields like `.executive_summary`. Test this before recording.
- **The run takes 20–40 seconds.** Dead air on camera. Narrate what's happening: "The agent is searching the web, querying our vector store, and potentially running code — all in one call." Have something to say during the wait.
- **Vector store creation is a prerequisite for everything.** If you restart the kernel and try to run Phase 2 or 3 without re-running Phase 1, you'll get a vector store not found error. If this happens on camera, it's actually a good teaching moment — but better to just run cells in order.
- **The `truncate_response()` helper truncates at 1200 characters.** The actual report may be much longer. If students wonder why the output looks cut off, point to this function. It's defined in Setup.
- **The `await` keyword in `Runner.run()`.** This is an async call in a Jupyter notebook. Jupyter handles `await` at the top level automatically, but students may wonder why `await` is here. Don't go deep — just say "Jupyter notebooks handle async for us; we'll cover async properly in M05C."
- **Two separate `Runner.run()` calls in Phase 3.** The first one stores results in `research_result`, the second (error handling demo) stores in `result`. The exercises reference `research_result`, so make sure that cell ran successfully. If you only run the error-handling cell, the exercises won't have the data they need.
- **The local file cleanup happens BEFORE the exercises.** The `ai_adoption_survey.txt` file gets deleted before the practice section. This is fine because the vector store still exists — but if students are confused about where the file went, explain that the vector store has the content now.
- **Exercise 2 references `REASONING_MODEL` in the TODO comments** ("Create a judge agent using REASONING_MODEL"), but `REASONING_MODEL` is never defined in this notebook's setup cell. Only `MODEL = "gpt-5-mini"` is defined. Students will need to either define `REASONING_MODEL = "gpt-5"` themselves or use `MODEL`. Mention this on camera when walking through the exercise.

## How it connects to adjacent notebooks

**What came before:** This capstone directly combines M04A (Web Search), M04B (File Search), and M04C (Code Interpreter) — all three built-in tools students learned individually. On camera, say something like: "In M04A, M04B, and M04C you learned each tool in isolation. Now we're putting them all in one agent and letting it orchestrate." It also uses structured outputs from M03A-2 (`output_type=ResearchReport`) and error handling from M03B (`try/except` around `Runner.run()`). The evaluation exercise calls back to M03C's golden test set and judge agent pattern — say: "Remember the evaluation pattern from M03C? This is where it becomes real. Evaluation isn't something you do once and forget — it follows you into every project."

**What comes next:** M05A introduces handoffs — routing tasks between specialized agents. The natural bridge on camera: "Right now, one agent is doing everything — research, analysis, writing. In Module 5, we'll split these responsibilities across multiple specialized agents and learn how to hand off work between them. In M05E, we'll rebuild this research concept as a full multi-agent team with parallel execution." This notebook is the single-agent ceiling — Module 5 is about breaking past it.

---