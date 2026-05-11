

---

# M04C: Code Interpreter — Recording Prep

## What this notebook teaches

Code Interpreter lets an agent write and execute Python code in a secure, sandboxed container hosted by OpenAI — so answers come from actual computation, not the model's best guess. This matters because LLMs are notoriously unreliable at arithmetic, statistics, and data analysis when forced to "think" their way to an answer. By the end of this notebook, students can attach `CodeInterpreterTool` to any agent and have it solve math problems, analyze CSV data, generate plots, and work with uploaded files.

## Concepts to explain on camera

- **Sandbox / Sandboxed Container**
  - **Term:** An isolated computing environment that OpenAI spins up to run the agent's Python code. It has no internet access, can't reach your local files, and is destroyed after the session ends.
  - **Analogy:** Think of it like a disposable laptop that OpenAI hands to your agent — it can run Python on it, but it can't browse the web or peek at your desktop. When the session's over, OpenAI throws the laptop away.
  - **What students need to understand:**
    - The code does NOT run on your machine — it runs on OpenAI's servers
    - The sandbox has no internet access, so the agent can't pip install packages or call APIs from inside it
    - Common data-science libraries (pandas, matplotlib, numpy) are pre-installed in the sandbox
    - The container is discarded after the session — nothing persists between runs

- **Container Session (and 20-minute billing increments)**
  - **Term:** A container session is the lifespan of one sandbox environment. OpenAI bills $0.03 per 20-minute session. If the container sits idle and expires, the next `Runner.run` call spins up a fresh one.
  - **Analogy:** It's like renting a meeting room in 20-minute blocks — even if you only use 5 minutes, you pay for the block. If you come back after the block expires, you get a new room.
  - **What students need to understand:**
    - Cost is separate from token costs — you pay for sandbox time on top of normal API usage
    - Multiple `Runner.run` calls within 20 minutes can reuse the same container
    - If the container expires between cells, the next run starts a fresh one (no error, just a new session)
    - Always verify current pricing at OpenAI's pricing page since rates change

- **Files API / File ID**
  - **Term:** The OpenAI Files API lets you upload a file from your local machine to OpenAI's servers and get back a unique identifier (file ID). You pass that ID into the container config so the sandbox has the file available before the agent runs.
  - **Analogy:** It's like emailing an attachment to a coworker before a meeting — they need the file on their machine before they can open it. The file ID is the tracking number that tells the sandbox which file to pull in.
  - **What students need to understand:**
    - Local files are NOT automatically visible inside the container — you must upload them explicitly
    - `openai_client.files.create(file=f, purpose="assistants")` returns an object with an `.id` attribute
    - That `.id` goes into the `file_ids` list in the container config
    - Uploaded files persist in your OpenAI account until you delete them — the notebook includes a cleanup cell

- **`CodeInterpreterTool`**
  - **Term:** The SDK class that enables Code Interpreter on an agent. You import it from `agents` and add it to the agent's `tools` list.
  - **Analogy:** It's like giving your agent a calculator app — except the calculator can run any Python code, not just arithmetic.
  - **What students need to understand:**
    - It replaces `@function_tool` for computation tasks — you don't write the tool function, the agent writes its own code
    - The `tool_config` dict tells OpenAI what kind of tool it is and how to configure the container
    - `"container": {"type": "auto"}` means OpenAI manages the container lifecycle for you
    - You can optionally pass `"file_ids"` inside the container config to pre-load files

## Key SDK pattern

The primary pattern is creating an agent with `CodeInterpreterTool` attached:

```python
agent_with_code = Agent(
    name="CodeAgent",
    instructions="You are a helpful assistant. Always use code to answer math and data questions.",
    model=MODEL,
    tools=[CodeInterpreterTool(tool_config={
        "type": "code_interpreter",
        "container": {"type": "auto"}
    })]
)

result = await Runner.run(agent_with_code, input="What is the sum of all prime numbers below 1000?")
print(result.final_output)
```

Parameter-by-parameter:

- **`name="CodeAgent"`** — Display name for the agent, appears in traces and logs.
- **`instructions=`** — System prompt. The phrase "Always use code to answer math and data questions" is critical — without it, the agent may try to answer from memory instead of writing code. This is the single most important instruction for Code Interpreter agents.
- **`model=MODEL`** — Uses `gpt-5-mini` (set in the setup cell). Mini is sufficient for code generation tasks.
- **`tools=[CodeInterpreterTool(...)]`** — A list containing one `CodeInterpreterTool` instance.
  - **`"type": "code_interpreter"`** — Tells OpenAI which built-in tool to enable.
  - **`"container": {"type": "auto"}`** — Lets OpenAI automatically manage container creation, reuse, and teardown.

For file input, the pattern extends the container config:

```python
tools=[CodeInterpreterTool(tool_config={
    "type": "code_interpreter",
    "container": {
        "type": "auto",
        "file_ids": [uploaded_file.id]
    }
})]
```

- **`"file_ids": [uploaded_file.id]`** — List of file IDs (from the Files API) to pre-load into the container's filesystem before the agent runs. The agent's code can then read them directly (e.g., `pd.read_csv("sales_report.csv")`).

## The demos and what each shows

**Part 2 — Math and Computation (Before/After comparison):** This is the hook for the whole notebook. First, `agent_no_code` (no Code Interpreter) is asked "What is the sum of all prime numbers below 1000?" — it will either guess, hallucinate a number, or hedge with caveats. Then `agent_with_code` (with Code Interpreter) gets the same question and writes a Python sieve, runs it, and returns the exact answer (76127). Emphasize on camera: same model, same question, wildly different reliability. The only difference is one line — `tools=[CodeInterpreterTool(...)]`. This is the clearest possible demonstration of why Code Interpreter exists.

**Part 3 — Data Analysis (inline CSV):** The `analysis_agent` receives a 6-month sales dataset as a string pasted directly into the message (no file upload). First it computes total revenue, best/worst months, averages, and growth trends. Then a second cell asks it to generate a matplotlib bar chart. Emphasize two things on camera: (1) for small datasets you don't need file uploads — just paste the data in the prompt, and (2) the agent can generate visualizations, not just numbers. Note that the plot is generated inside the sandbox — students won't see the image rendered in the notebook, but the agent will describe what it created. Mention that file output retrieval is possible but beyond this notebook's scope.

**Part 4 — Iterative Problem Solving:** The agent tackles a classic constraint problem: find combinations of apples ($1.20), bananas ($0.35), and oranges ($2.10) totaling exactly 100 items for exactly $100.00. This demonstrates that Code Interpreter doesn't just run code once — the agent can write code, examine output, and refine its approach across multiple iterations within a single `Runner.run` call. Emphasize on camera: this iteration loop is automatic. The agent decides on its own whether it needs another pass. You don't have to build a retry loop.

**Part 5 — File Input:** This is the most mechanically complex demo. Three steps: (1) create a local CSV file with `Path.write_text()`, (2) upload it to OpenAI with `openai_client.files.create()` to get a file ID, (3) pass that file ID into `file_ids` in the container config. Then the `file_agent` reads the uploaded file and analyzes it. Emphasize on camera: the local file is cleaned up immediately after upload (`csv_path.unlink()`), so the agent is reading from OpenAI's copy, not your disk. Also call out the cleanup cell at the bottom that deletes the uploaded file from OpenAI — good hygiene for managing your account.

## Gotchas worth knowing before recording

- **The "without Code Interpreter" cell may still give the correct answer.** GPT-5-mini might get "sum of primes below 1000" right by coincidence or memorization. If this happens on camera, emphasize that it's unreliable — try a different run or a harder math problem and it will fail. The point is verified computation vs. probabilistic guessing, not that it always gets it wrong.

- **The plot won't render visually in the notebook.** Code Interpreter generates the matplotlib chart inside the sandbox, but the image file stays in the container. `result.final_output` will contain the agent's text description of the chart, not the chart itself. Don't promise students they'll see a rendered image — say "the agent generated the plot and here's its description of what it created."

- **All cells use `await Runner.run(...)` — the notebook must be running in an async-capable Jupyter environment.** Standard JupyterLab and Google Colab support top-level `await`. If students get `SyntaxError: 'await' outside function`, they're running a non-async kernel.

- **`purpose="assistants"` in the file upload is required.** Students might wonder why — it's just how OpenAI categorizes files for different features. Don't overthink it on camera, but do mention it's not optional.

- **The `truncate_response` helper in the setup cell will silently cut off long outputs at 1200 characters.** If a result looks incomplete on camera, it might just be truncated. You can temporarily increase `max_length` or remove the wrapper if you want full output for a specific demo.

- **Container sessions may expire between cells if you pause recording.** If you take a break between Part 3 and Part 4, the container may be gone. The next `Runner.run` will spin up a new one silently — no error, but a brief delay. Mention this if it happens on camera.

- **The `openai_client = OpenAI(...)` is created separately from the Agent SDK.** Students may wonder why there are two ways to talk to OpenAI. Explain: the Agent SDK (`Runner.run`) handles agent orchestration, but the Files API is a direct REST operation on the platform — you need the standard `OpenAI` client for that.

- **The cleanup cell at the bottom (`openai_client.files.delete(uploaded_file.id)`) will fail if the file upload cell hasn't been run.** If you restart the kernel and jump to cleanup, you'll get a `NameError`. Not a big deal, but be aware.

- **Instructions matter more than you'd expect.** The `agent_with_code` includes "Always use code to answer math and data questions" — without this, the agent may try to answer from memory even with Code Interpreter available. Call this out explicitly on camera as a best practice.

## How it connects to adjacent notebooks

**What came before:** M04A (Web Search) and M04B (File Search) introduced the first two built-in tools. Students already know the pattern of adding tools to the `tools=[]` list. On camera, say something like: "In 4A and 4B we gave our agent eyes — it could search the web and read documents. Now in 4C we're giving it hands — it can actually compute and execute code." Also callback to M02B where students built `@function_tool` functions — Code Interpreter is different because the agent writes its own tool code at runtime instead of calling a function you predefined.

**What comes next:** M04D (Capstone #1 — Research Agent) combines all three built-in tools — web search, file search, and code interpreter — into a single agent that researches a topic, analyzes documents, and produces a structured report. On camera, say: "Now that you've seen each built-in tool individually, the capstone puts them all together. Your agent will decide on its own which tool to use for each part of the task." Also note that Code Interpreter reappears in M05E where the Analyst agent in the multi-agent research team uses it for data analysis.

---