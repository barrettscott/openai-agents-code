# M09C: Deploying Agents with Gradio -- Recording Prep

## What this notebook teaches

This notebook takes an agent that only works inside a Jupyter notebook and turns it into a real chat interface anyone can use, deployed to a public URL. It teaches the Gradio library -- specifically `gr.ChatInterface` -- as the bridge between an OpenAI Agents SDK agent and a web UI, then walks through streaming, tool visibility, error handling, and deployment to Hugging Face Spaces. This is the final technical notebook in the course: by the end, students have gone from a 5-line agent in M02A to a live, publicly hosted chat app.

## Concepts to explain on camera

- **Gradio**
  - **Term:** A Python library for building web UIs around AI models and agents. You write a Python function, Gradio wraps it in a browser-based chat interface with no frontend code.
  - **Analogy:** Gradio is like a picture frame for your agent -- the agent is the painting, Gradio is the frame that makes it presentable and hangable on a wall (a public URL).
  - **What students need to understand:**
    - Gradio is not a web framework like Flask or Django -- it is purpose-built for AI demos
    - `gr.ChatInterface` expects a function that takes `message` (str) and `history` (list) and returns a string
    - Gradio handles the conversation loop, message display, and UI automatically
    - A working chat app takes roughly 10 lines of code

- **Python generator / `yield`**
  - **Term:** A function that produces values one at a time using `yield` instead of returning all at once with `return`. Each `yield` sends a partial result to the caller.
  - **Analogy:** A `return` is like handing someone a finished letter. A `yield` is like reading the letter aloud sentence by sentence -- the listener gets information as it arrives, not all at the end.
  - **What students need to understand:**
    - Gradio treats any function that uses `yield` as a streaming source
    - Each `yield` updates the chat bubble in the UI in real time
    - You accumulate text into `response_text` and yield the growing string on every delta
    - Switching from `return result.final_output` to `yield response_text` is the only change needed on the Gradio side

- **Hugging Face Spaces**
  - **Term:** Free hosted environments from Hugging Face that run Gradio apps. You push your files, Spaces installs dependencies and gives you a public URL.
  - **Analogy:** Hugging Face Spaces is like GitHub Pages but for AI apps -- push your code, get a live URL, no server setup.
  - **What students need to understand:**
    - The minimum file set is `app.py`, `requirements.txt`, and `.gitignore`
    - `demo.launch()` is called with no arguments -- Spaces handles port and host
    - API keys go in Space Settings as Repository Secrets, never in committed files
    - `os.getenv("OPENAI_API_KEY")` reads the secret the same way it reads from `.env` locally

- **Repository Secrets (Hugging Face)**
  - **Term:** Environment variables stored securely in the Space settings UI, injected at runtime. The equivalent of a `.env` file but managed by the platform.
  - **Analogy:** Like a hotel safe for your API key -- your code knows the combination (`os.getenv`), but the key is never left lying around in your luggage (your Git repo).
  - **What students need to understand:**
    - The secret name must match exactly: `OPENAI_API_KEY`
    - You must restart the Space after adding a secret
    - Never commit a `.env` file -- add it to `.gitignore`
    - This pattern was introduced in M09A as "platform secrets for deployment"; this notebook is where students actually use it

## Key SDK pattern

The primary pattern is bridging `Runner.run_streamed()` into Gradio's generator-based streaming. The exact code from the notebook:

```python
async def chat(message: str, history: list):
    """Stream the agent response token by token."""
    response_text = ""

    result = Runner.run_streamed(agent, input=message)

    async for event in result.stream_events():
        if isinstance(event, RawResponsesStreamEvent):
            data = event.data
            if isinstance(data, ResponseTextDeltaEvent):
                response_text += data.delta
                yield response_text


demo = gr.ChatInterface(
    fn=chat,
    title="My Agent (Streaming)",
    description="Responses appear as they arrive."
)
```

Parameter-by-parameter:
- **`fn=chat`** -- the async generator function Gradio calls on every user message. Must accept `message` (str) and `history` (list). Gradio detects `yield` and treats it as streaming.
- **`title=`** and **`description=`** -- text displayed at the top of the chat UI. Cosmetic only.
- **`Runner.run_streamed(agent, input=message)`** -- the SDK's streaming entry point (introduced in M09A). Returns a result object whose `.stream_events()` produces events as they arrive.
- **`RawResponsesStreamEvent`** and **`ResponseTextDeltaEvent`** -- the event types that carry text deltas. Each `data.delta` is a small chunk of generated text.
- **`response_text += data.delta`** then **`yield response_text`** -- accumulate the full response so far and yield it. Gradio replaces the entire chat bubble content on each yield, so you must yield the growing string, not just the delta.

## The demos and what each shows

**Part 2 -- Minimal `gr.ChatInterface` (cell 10, `MINIMAL_APP`).** Defines an `Agent` named `"Assistant"` with basic instructions, wraps it in an `async def chat(message, history)` that calls `Runner.run(agent, input=message)` and returns `result.final_output`. This is the simplest possible Gradio agent app. Emphasize on camera: the agent code does not change at all -- you are just changing what calls it. Also call out the caveat in cell 12: `history` is UI state only. The agent is stateless unless you reconstruct history into the input (the notebook shows the `context = "\n".join(...)` pattern for including the last 3 turns).

**Part 3 -- Streaming (cell 15, `STREAMING_APP`).** Replaces `Runner.run()` with `Runner.run_streamed()` and changes `return` to `yield`. The chat function becomes an async generator that yields `response_text` on every `ResponseTextDeltaEvent`. Emphasize: two changes -- `run()` becomes `run_streamed()`, `return` becomes `yield`. Gradio handles the rest.

**Part 4 -- Tool visibility (cell 19, `TOOL_VISIBILITY_APP`).** Adds a `get_weather` tool (returns `"Weather in {location}: 72F and sunny"`) and checks for `RunItemStreamEvent` in the event loop. When a tool call starts, the function yields `"Calling {item.name}..."` as a status message. Then text deltas replace it with the actual response. Emphasize: without this, the user sees dead air while the tool runs. The status yield fills that gap.

**Part 5 -- Failure handling (cell 23, `FAILURE_HANDLING_APP`).** Wraps the entire streaming loop in `try/except Exception as e` and yields a friendly message: `"Something went wrong. Please try again. (Error: {type(e).__name__})"`. Emphasize: expose the error type for debugging but never the full stack trace. The app stays running and the next message works normally.

**Part 6 -- Deployment files (cells 27, `APP_PY` and `REQUIREMENTS_TXT`).** Shows the complete `app.py` for Hugging Face Spaces -- reads `OPENAI_API_KEY` from `os.getenv()`, no `.env` loading, calls `demo.launch()` with no arguments. Also shows the two-line `requirements.txt` (`openai-agents`, `gradio`). Emphasize the deployment checklist in cell 28: test locally with `python app.py` before pushing.

## Gotchas worth knowing before recording

- All the app code in this notebook is stored as **multiline strings** (`MINIMAL_APP`, `STREAMING_APP`, etc.) and printed -- the cells do not actually launch Gradio servers. Be ready to explain that you are showing the code pattern, not running a live app inside the notebook. If you want to demo live, copy the code into a standalone `app.py` and run it from the terminal.
- **`history` is not agent memory.** `gr.ChatInterface` passes `history` to the chat function, but the agent itself is stateless between calls. Students will assume the agent "remembers" because the UI shows past messages. Call this out explicitly and reference M06A sessions if they want real memory.
- The `yield response_text` pattern yields the **entire accumulated string** each time, not just the new delta. If you accidentally yield only `data.delta`, Gradio will replace the previous text with just the new chunk -- the response will flicker to single tokens.
- **`Runner.run_streamed()` vs `Runner.run()`** -- students may confuse which to use. `run()` returns a completed result; `run_streamed()` returns an object you iterate over. Using `run()` in a generator function means the full response loads first, then appears all at once -- defeating the purpose of streaming.
- The `RunItemStreamEvent` check in Part 4 uses `hasattr(item, "name")` to detect tool calls. This is a duck-typing shortcut -- not every `RunItemStreamEvent` item has a `name` attribute. If students add more event handling, they may need more specific type checks.
- The deployment `app.py` does **not** call `load_dotenv()` -- this is intentional for Spaces. If students copy it for local development, they will need to add `load_dotenv()` back or set the env var manually.
- `demo.launch()` must be called with **no arguments** on Hugging Face Spaces. Passing `server_port=` or `server_name=` will conflict with Spaces' configuration and the app will fail to start.
- The model is set to `"gpt-5-mini"` throughout. In the deployment `app.py`, it reads from `os.getenv("MODEL", "gpt-5-mini")` -- the fallback default means students do not need to set a MODEL secret unless they want a different model.

## How it connects to adjacent notebooks

**Builds on M09A (Project Structure & CLI):** M09A introduced `Runner.run_streamed()` and streaming output in a script context, plus secret management with `.env` for dev and platform secrets for deployment. This notebook takes that same streaming pattern and plugs it into Gradio. On camera, say something like: "In M09A we streamed to the terminal -- now we are streaming to a real chat UI."

**Builds on M09B (Architecture Decisions):** M09B covered when to use different patterns (single agent vs multi-agent, mini vs reasoning model). Students arrive here having already decided what their agent looks like -- this notebook is about shipping it. Reference: "You have made your architecture decisions -- now let us put it in front of users."

**Builds on M06A (Sessions) and M06B (Persistent Memory):** The notebook explicitly calls out that `gr.ChatInterface` history is UI-only, not agent memory. Reference M06A for sessions and M06B for persistent memory if students want the agent to actually remember across turns.

**Builds on M07E (Capstone 3 -- Customer Service Agent):** Exercise 1 suggests wrapping the customer service agent from M07E or the MCP assistant from M08C in a Gradio interface. Students are connecting back to agents they already built.

**Course finale:** This is the last technical notebook. The closing cell (cell 40) explicitly marks the course as complete and lists what students have built: four capstone projects, tools, memory, guardrails, multi-agent orchestration, human oversight, MCP integration, and now a live public URL. There is no "next notebook" -- this is the finish line.
