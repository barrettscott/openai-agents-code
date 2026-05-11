# M09A: Project Structure & CLI — Recording Prep

## What this notebook teaches

This notebook teaches students how to move an agent from a Jupyter prototype into a real, runnable project with separated concerns. It solves the problem of notebook code being un-deployable by introducing a four-file structure (`config.py`, `tools.py`, `agent.py`, `main.py`), showing how to stream agent output in a terminal, when to bypass the Agents SDK with `client.responses.create()`, and how to manage secrets properly. Everything here is template code printed to screen rather than executed — students copy it into real files.

## Concepts to explain on camera

- **`asyncio.run()`**
  - **Plain English:** Python's way of starting an async function from a normal (synchronous) script. In Jupyter, top-level `await` just works; in a `.py` file, you need `asyncio.run(main())` to kick off the event loop.
  - **Analogy:** Jupyter has a built-in engine running at all times. A script is a parked car — `asyncio.run()` is turning the key in the ignition.
  - **What students need to understand:**
    - Every script with `await` calls needs exactly one `asyncio.run()` at the entry point
    - It goes in `main.py` inside the `if __name__ == "__main__":` block
    - You never call `asyncio.run()` inside Jupyter — it already has a running loop
    - Forgetting it gives `RuntimeError: no running event loop`

- **Entry point (`if __name__ == "__main__":`)**
  - **Plain English:** The line that says "only run this code if I'm the file being executed directly, not imported by another file."
  - **Analogy:** A light switch on the wall — the wiring is always there, but the light only turns on when you flip the switch yourself.
  - **What students need to understand:**
    - `main.py` uses this to start the CLI loop or the FastAPI app
    - Without it, importing `main.py` from another file would accidentally run your whole program
    - This is standard Python, not specific to the Agents SDK

- **Streaming (`Runner.run_streamed()` + event filtering)**
  - **Plain English:** Instead of waiting for the full response and printing it all at once, you get each token as it's generated and print it immediately — like watching someone type in real time.
  - **Analogy:** Regular mode is like getting a letter in the mail. Streaming mode is like reading a text message as the other person types it.
  - **What students need to understand:**
    - You swap `Runner.run()` for `Runner.run_streamed()` — same agent, same tools, one method change
    - You iterate with `async for event in result.stream_events()`
    - You filter for `RawResponsesStreamEvent` then check if `event.data` is a `ResponseTextDeltaEvent`
    - `print(data.delta, end="", flush=True)` is critical — without `flush=True`, output buffers and nothing appears

- **`client.responses.create()` (direct Responses API)**
  - **Plain English:** A raw, single-turn model call that skips all the Agents SDK machinery — no tools, no handoffs, no guardrails, no sessions. Just send a prompt, get a response.
  - **Analogy:** The Agents SDK is a full kitchen with a chef, prep cooks, and a menu. `client.responses.create()` is a microwave — fast and simple when all you need is to heat something up.
  - **What students need to understand:**
    - Use it for simple classification, extraction, or transformation tasks
    - It uses `OpenAI()` client directly, not `Agent` or `Runner`
    - Returns `response.output_text` — no `final_output`, no run result object
    - Default to the Agents SDK; only drop down when the overhead genuinely isn't worth it

- **Secret management (`.env` + `os.getenv()`)**
  - **Plain English:** A pattern for keeping API keys out of your code. Locally, keys live in a `.env` file that never gets committed. In production, keys live in your platform's secret manager. Your code reads them the same way either way.
  - **Analogy:** A `.env` file is like a sticky note in your desk drawer — it has the password, but it never leaves your desk. Platform secrets are like a combination safe at work — same password, more secure storage.
  - **What students need to understand:**
    - `.env` must be in `.gitignore` immediately — if you commit it once, the key is in your git history forever
    - `load_dotenv()` reads `.env` into environment variables; `os.getenv("OPENAI_API_KEY")` retrieves them
    - The same `os.getenv()` call works whether the variable came from `.env` or a platform secret manager
    - The notebook pattern includes a `raise ValueError` check if the key is missing — good defensive practice

## Key SDK pattern

The primary pattern is the four-file project structure itself, but the key new SDK call is **`Runner.run_streamed()`** for streaming output in a script:

```python
from agents import Agent, Runner
from agents.stream_events import RawResponsesStreamEvent
from openai.types.responses import ResponseTextDeltaEvent

agent = Agent(
    name="MyAgent",
    instructions="Your agent instructions here.",
    model=MODEL,
    tools=[my_tool]
)

async def run_streaming(message: str) -> None:
    """Stream agent output token by token."""
    print("Agent: ", end="", flush=True)

    result = Runner.run_streamed(agent, input=message)

    async for event in result.stream_events():
        if isinstance(event, RawResponsesStreamEvent):
            data = event.data
            if isinstance(data, ResponseTextDeltaEvent):
                print(data.delta, end="", flush=True)

    print()  # newline after streaming completes
```

- **`Runner.run_streamed(agent, input=message)`** — replaces `Runner.run()`. Returns a streaming result object instead of awaiting the full response. Note: no `await` on this call.
- **`result.stream_events()`** — async iterator yielding events as they arrive. You loop with `async for`.
- **`RawResponsesStreamEvent`** — the event wrapper from the SDK. You check `event.data` for the specific delta type.
- **`ResponseTextDeltaEvent`** — contains `data.delta`, the actual text fragment (token) to print.
- **`flush=True`** — forces Python to write to the terminal immediately instead of buffering. Without this, nothing appears until the buffer fills.

## The demos and what each shows

**Part 1 (What Changes When You Leave Jupyter):** No code execution — just a conceptual framing. Explain three things notebooks hide: top-level `await`, global state across cells, and `.env` loading order. This sets up the motivation for the rest of the notebook. Emphasize that none of this is hard, it just becomes explicit.

**Part 2 (A Reusable Project Structure):** Presents the directory layout: `.env`, `.gitignore`, `requirements.txt`, `config.py`, `tools.py`, `agent.py`, `main.py`. No code runs — it's a reference diagram. Point out that this same structure works for a CLI tool or a web service; only `main.py` changes.

**Part 3 (The Four Files):** Four code cells, each printing a template string (`CONFIG_TEMPLATE`, `TOOLS_TEMPLATE`, `AGENT_TEMPLATE`, `MAIN_TEMPLATE`). None of them execute as real modules — they print the source code students will copy. Walk through each file's single responsibility: `config.py` owns `MODEL` and `REASONING_MODEL` plus `load_dotenv()`, `tools.py` owns `@function_tool` definitions, `agent.py` creates the `Agent` and exposes `run_agent(message)`, and `main.py` runs the CLI loop with `asyncio.run(main())`. Emphasize that `run_agent()` is the bridge — it's the one function every entry point calls.

**Part 4 (Streaming Agent Output in a Script):** Prints `STREAMING_TEMPLATE` — a modified `main.py` that uses `Runner.run_streamed()` instead of `Runner.run()`. Emphasize the two imports (`RawResponsesStreamEvent`, `ResponseTextDeltaEvent`) and the event filtering pattern. This is a fixed pattern students copy once and reuse. Mention that M09C extends this exact streaming pattern into a Gradio chat interface.

**Part 5 (When to Drop Down to `client.responses.create()`):** Prints `RESPONSES_TEMPLATE` showing a `classify()` function that calls `client.responses.create()` directly with `model=MODEL`. Emphasize this is for single-turn calls with no agent machinery. The example classifies text as positive/negative/neutral. Make the decision rule clear: Agents SDK by default, direct API only when you genuinely don't need tools, handoffs, guardrails, or sessions.

**Part 6 (Secret Management):** Prints `SECRETS_TEMPLATE` showing the `.env` + `load_dotenv()` + `os.getenv()` + validation pattern. Stress the `.gitignore` requirement. Show that `os.getenv()` works identically whether the key came from `.env` or a platform secret manager — same code, different environments.

**Part 7 (Simple API Wrapper):** Prints `FASTAPI_TEMPLATE` — a `main.py` variant that wraps `run_agent()` in a FastAPI endpoint. Uses `ChatRequest` and `ChatResponse` Pydantic models. The key point: `run_agent()` is already async, so FastAPI just awaits it directly. The agent code in `agent.py` doesn't change at all — only the entry point changes. Clarify this is an entry-point variation, not a deployment lesson; M09C covers actual deployment.

## Gotchas worth knowing before recording

- **No code actually executes as modules.** Every code cell prints a template string. If you accidentally try to import from `config` or `tools`, it will fail — those files don't exist in the notebook directory. Be clear with students that these are templates to copy into separate `.py` files.
- **`Runner.run_streamed()` is NOT awaited.** Unlike `Runner.run()` which uses `await`, `run_streamed()` is called without `await` — it returns a result object you then iterate over. This will confuse students who expect consistency.
- **`flush=True` is easy to forget.** Without it, streaming output looks broken — nothing appears until the buffer fills or the stream ends. Mention this explicitly on camera.
- **The `AGENT_TEMPLATE` uses `Runner.run()` with `await`, but `STREAMING_TEMPLATE` uses `Runner.run_streamed()` without `await`.** Students may wonder why the calling convention differs — explain that `run_streamed()` returns a stream object immediately rather than a coroutine.
- **The FastAPI template requires `pip install fastapi uvicorn`** — not in the standard `requirements.txt` from the course. Mention this dependency if students want to try it.
- **`Path(__file__).parent / ".env"` won't work in Jupyter.** `__file__` is not defined in notebooks. This is by design — the template is for `.py` files only. If a student asks, that's the answer.
- **The `classify()` example in Part 5 uses synchronous `OpenAI()` client**, not the async `AsyncOpenAI()`. This is intentional for simplicity, but students familiar with the async patterns from earlier modules may ask about it.

## How it connects to adjacent notebooks

**Before (Module 8 — MCP):** Students have been building increasingly sophisticated agents in notebooks — tools, multi-agent systems, memory, guardrails, MCP servers. All of that was prototyping. This notebook is the bridge to making it real. On camera: "Everything we've built so far has lived inside Jupyter. Now we're going to take it out."

**After (M09B — Architecture Decisions):** M09B provides a decision framework for model selection (mini vs reasoning vs parallel specialists), single vs multi-agent, when memory adds value, and when MCP is worth the setup. On camera: "Now that you know how to structure a project, M09B helps you decide what goes inside it." Also reference that M09B is the course wrap-up with suggested next projects.

**After (M09C — Deploying Agents with Gradio):** M09C takes the streaming pattern from Part 4 of this notebook and wraps it in a Gradio chat interface, then deploys to Hugging Face Spaces. On camera: "The streaming pattern you just saw — `Runner.run_streamed()` plus event filtering — is exactly what M09C uses inside Gradio."
