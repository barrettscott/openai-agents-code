

---

# M02A: How Agents Work — Recording Prep

## What this notebook teaches

This notebook teaches the three fundamental building blocks of every agent in the OpenAI Agents SDK: `Agent` (defines behavior), `Runner.run()` (executes it), and `result.final_output` (gets the answer). It proves that `instructions` are the single lever that controls what an agent does — same model, same input, different instructions, completely different output. It also introduces tracing as a freebie: every run is automatically recorded in the OpenAI dashboard with zero configuration.

## Concepts to explain on camera

- **Agent (the SDK class)**
  - **Term:** A Python object that wraps a model, a name, and a set of instructions into a single reusable unit. It doesn't do anything on its own — it's a blueprint.
  - **Analogy:** An Agent is like a job description. It says who this worker is and how they should behave, but it doesn't start any work until someone hands it to a manager (the Runner).
  - **What students need to understand:**
    - `Agent()` takes three key parameters right now: `name`, `instructions`, and `model`
    - `name` is a label for tracing/debugging — it shows up in the dashboard
    - `instructions` is the system prompt — the single most important parameter
    - Creating an Agent doesn't call the API; it just configures one

- **Runner**
  - **Term:** The execution engine that takes an Agent and a user message, sends them to the OpenAI API, manages the full lifecycle of the call, and returns a result object.
  - **Analogy:** If the Agent is the job description, the Runner is the manager who hands the worker the task, waits for the answer, and brings it back to you.
  - **What students need to understand:**
    - You always call `Runner.run(agent, input=message)` — the Runner does the actual API call
    - It returns a `RunResult` object, not raw text
    - You need `await` because it's an async call (Jupyter handles this automatically)
    - Later notebooks will use other fields on the result object, but for now `final_output` is all you need

- **`result.final_output`**
  - **Term:** The field on the `RunResult` object that contains the agent's final text response — the answer you actually want.
  - **Analogy:** When you send someone to do research, `final_output` is the report they hand back. The result object is the whole folder including notes, but `final_output` is the finished document on top.
  - **What students need to understand:**
    - It's a string (plain text) by default — later in M03A-2 we'll make it structured with Pydantic
    - It can be `None` if something goes wrong — the troubleshooting section covers this
    - You access it with dot notation: `result.final_output`, not `result["final_output"]`

- **Instructions (system prompt)**
  - **Term:** A plain-text string that tells the agent how to behave, what tone to use, what constraints to follow, and what kind of output to produce. It's sent as the system message on every API call.
  - **Analogy:** Instructions are like the brief you give a freelancer before they start — "write formally," "keep it under one paragraph," "explain it to a kid." Same freelancer, different brief, different output.
  - **What students need to understand:**
    - Instructions are the highest-leverage thing you can change — they define persona, tone, format, and constraints
    - They're set once on the Agent and apply to every run of that agent
    - M02C is an entire notebook dedicated to writing effective instructions — this is just the intro
    - In multi-agent systems (M04–M05), instructions become the difference between a working system and a broken one

- **Tracing**
  - **Term:** Automatic logging of every agent run — the input, instructions, model used, and output — visible in the OpenAI dashboard with no code or configuration required.
  - **Analogy:** It's like a security camera for your agent. Every time the agent runs, a recording appears in the dashboard showing exactly what went in and what came out.
  - **What students need to understand:**
    - It's on by default — you don't write any tracing code
    - You view traces at platform.openai.com under the Traces section
    - Traces have a 10–15 second delay before appearing
    - Deep tracing (debugging tool calls, handoffs, costs) is covered in M07D — this is just "hey, it exists"

- **Model constants (`MODEL` and `REASONING_MODEL`)**
  - **Term:** Two variables defined at the top of every notebook that control which OpenAI model the agent uses. `MODEL = "gpt-5-mini"` is the fast/cheap default; `REASONING_MODEL = "gpt-5"` is the powerful model reserved for complex tasks.
  - **Analogy:** `MODEL` is your everyday commuter car — gets you there efficiently. `REASONING_MODEL` is the heavy-duty truck — you only bring it out when you need serious hauling power. Using the truck for every errand wastes money and time.
  - **What students need to understand:**
    - Almost every agent in this course uses `MODEL` — it's fast and cheap enough for learning
    - `REASONING_MODEL` shows up in later modules for orchestration and complex judgment (M08C, M09B)
    - These are just Python variables, not SDK concepts — they keep model names in one place so you can swap them easily
    - The full decision framework for choosing between them is in M09B

- **`await` (in context of `Runner.run()`)**
  - **Term:** A Python keyword that tells the program to wait for an asynchronous operation to finish before continuing. Required because `Runner.run()` is an async function that makes a network call to OpenAI's API.
  - **Analogy:** Ordering at a restaurant — `await` means "I'll wait here for my food" instead of walking away. Jupyter handles the kitchen logistics for you.
  - **What students need to understand:**
    - Every `Runner.run()` call needs `await` in front of it
    - JupyterLab supports top-level `await` natively — no `asyncio.run()` wrapper needed
    - If you see an "await outside async context" error, it's a kernel issue, not a code issue — restart kernel
    - Async becomes important in M05C when we run agents in parallel; for now it's just syntax

## Key SDK pattern

The core pattern introduced in this notebook:

```python
agent = Agent(
    name="Assistant",
    instructions="You are a helpful assistant.",
    model=MODEL
)

result = await Runner.run(agent, input=message)

print(result.final_output)
```

**Parameter-by-parameter:**

- **`name="Assistant"`** — A human-readable label. Shows up in the OpenAI tracing dashboard. Pick something descriptive. Not sent to the model as part of the prompt (it's metadata).
- **`instructions="You are a helpful assistant."`** — The system prompt. Defines persona, tone, constraints, and behavior. This is the single most important parameter on an Agent. Everything the agent does flows from this string.
- **`model=MODEL`** — Which OpenAI model to use. Set to the `MODEL` constant (`"gpt-5-mini"`) defined in setup. Pass `REASONING_MODEL` when you need heavier reasoning.
- **`input=message`** — The user's message (a string). This is the task or question being sent to the agent.
- **`result.final_output`** — The agent's final text response. A string by default. Later (M03A-2) this becomes a structured Pydantic object when you set `output_type=`.

## The demos and what each shows

**Setup cell:** Imports `Agent` and `Runner` from the `agents` package, loads the `.env` file for the API key, and defines `MODEL` and `REASONING_MODEL` as constants. Emphasize on camera that these two constants appear at the top of every notebook in the course — this is intentional so students always know which model they're using and can change it in one place. The `print("✅ Ready!")` is a quick visual confirmation that nothing errored during import.

**"How an Agent Works" demo:** Sends `"What is Python?"` to a generic assistant agent with the instruction `"You are a helpful assistant."`. This is the first time students deliberately construct all three pieces — `Agent`, `Runner.run()`, `result.final_output` — as opposed to the verification call in M01B. Emphasize that this is the pattern they'll use hundreds of times. Point out that `Runner.run()` returns a `RunResult` object, not a string — `final_output` is one field on that object.

**Part 1 — Concise Agent:** Same `"What is Python?"` message, but now `instructions` is `"Answer every question in exactly one sentence."` The agent should return a single sentence. This is the "before" in the before/after comparison. On camera, pause after the output and point out: same model, same question, completely different answer. The only thing that changed is `instructions`.

**Part 1 — Friendly Agent:** Same `"What is Python?"` message again, but `instructions` is `"Explain concepts to a 10-year-old using a fun analogy. Keep the answer to one short paragraph."` The output should be a playful, accessible explanation. This is the payoff demo — three agents, three personalities, one line of config. Emphasize that this is what makes instructions the most important parameter on an Agent. Say explicitly: "We'll spend all of M02C learning to write great instructions."

**Part 2 — Tracing:** No code to run — this is a screencast moment. After running the demos above, switch to the OpenAI dashboard and show the Traces page. Students should see the three runs listed. Click into one and show the input, the instructions, and the output side by side. Emphasize: "You didn't write any tracing code. This is automatic." Mention the 10–15 second delay. Say: "We'll come back to tracing in M07D when we use it to debug complex agents."

**Practice exercises:** Exercise 1 asks students to create a meeting-notes summarizer agent — given a block of meeting notes, extract decisions and action items. Exercise 2 asks students to build two tone-transformer agents (formal and casual) that rewrite the same message. Both exercises reinforce the core pattern and the power of instructions. Don't solve these on camera — just read the instructions clearly and let students work.

## Gotchas worth knowing before recording

- **`await` is required and easy to forget.** If you accidentally run `Runner.run(agent, input=message)` without `await`, you'll get a coroutine object printed instead of a result. This looks confusing on camera. Always double-check the `await` is there before running.
- **`result` is a `RunResult` object, not a string.** If you accidentally `print(result)` instead of `print(result.final_output)`, you'll get a repr dump of the whole object. Not wrong, just messy. Might be worth doing once on purpose to show students the difference.
- **The output of each demo is non-deterministic.** Running the same cell twice will give different wording. Don't promise students will see the exact same text. Say "yours might look slightly different — that's normal."
- **The concise agent might not give exactly one sentence.** LLMs don't always follow constraints perfectly. If it gives two sentences, acknowledge it on camera: "Close enough — the model tries to follow the instruction but it's not a hard rule. We'll see how to enforce structure more strictly in M03A-2 with structured outputs."
- **The friendly agent uses a multi-line string with parenthetical concatenation** (`friendly_instructions = ("..." "...")`). Some students may not have seen this Python syntax. Worth a quick aside: "This is just Python's implicit string concatenation — two strings next to each other inside parentheses get joined into one."
- **Variable naming changes across demos.** The first demo uses `agent` and `result`. Part 1 uses `agent_concise`/`concise_result` and `agent_friendly`/`friendly_result`. This is deliberate (to allow all results to coexist in memory) but might confuse students who expect consistent naming. Call it out: "I'm using different variable names so we can compare the outputs later."
- **Tracing delay.** Traces can take 10–15 seconds to appear. If you switch to the dashboard immediately after running a cell, it might not be there yet. Have a plan — either wait, or run the cells first and then switch to the dashboard after all three are done.
- **The `.env` path uses `Path("..") / ".env"`** — the `.env` file is one directory up from the notebook. If students moved the notebook or have a different folder structure, this will silently fail (no error, but the API key won't load). The resulting error will be an authentication error, not a "file not found."
- **No `REASONING_MODEL` is actually used in this notebook.** It's defined in setup but never called. That's intentional — it's there so students know it exists. Mention it briefly and say "we'll use this in later modules."

## How it connects to adjacent notebooks

**Builds on M01B (OpenAI Setup):** In M01B, students made their first agent call as a setup verification step. On camera, say: "In the last notebook, you made your first agent call to verify everything was working. Now we're going to slow down and understand exactly what that code was doing — what each piece is, and how changing one parameter changes everything." Students should already have a working `.env` file and a verified API key.

**Leads into M02B (Building Tools for Agents):** This notebook establishes the Agent + Runner + final_output pattern with text-only agents. M02B adds the first new capability: tools. On camera, close with: "Right now our agent can only talk — it has no abilities beyond generating text. In the next notebook, we'll give it superpowers by turning Python functions into tools it can call on its own." This sets up the `@function_tool` decorator introduction.

**Forward references worth mentioning:** When discussing `instructions`, say "We'll spend all of M02C on writing effective instructions." When discussing model constants, say "The full framework for choosing between these is in M09B." When showing tracing, say "We'll use tracing to debug complex agents in M07D." When the concise agent doesn't perfectly follow the one-sentence constraint, say "In M03A-2, we'll use structured outputs to enforce exact formats."

---