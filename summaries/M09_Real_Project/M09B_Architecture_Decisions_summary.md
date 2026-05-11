# M09B: Architecture Decisions — Recording Prep

## What this notebook teaches

This notebook gives students a decision framework for choosing the right level of complexity when building agents. Instead of reaching for every pattern taught in the course, students learn to start simple (single agent, `MODEL`, no sessions, no MCP) and add complexity only when a specific, observable failure demands it. It ties together every major pattern from M02 through M08 into a single reference table and closes the course with six suggested projects.

## Concepts to explain on camera

- **Decision framework (complexity budgeting)**
  - **Term:** A systematic process for deciding which agent patterns to use before writing code, rather than bolting on patterns after the fact.
  - **Analogy:** Like packing for a trip — decide what you actually need before you leave, instead of hauling everything you own and sorting it out at the hotel.
  - **What students need to understand:**
    - Start every project with the simplest possible version: one agent, `MODEL`, no sessions, no MCP
    - Identify specific failing test cases (not vibes) before adding complexity
    - Every pattern you add is another thing to debug, monitor, and maintain
    - The sequence is: build simple, evaluate with golden test set, identify specific failures, add minimum fix, re-evaluate

- **Tool proliferation**
  - **Term:** When an agent has too many tools to choose from, causing it to make worse decisions about which tool to call.
  - **Analogy:** Like giving someone a Swiss Army knife with 50 blades — they spend more time finding the right blade than actually cutting anything.
  - **What students need to understand:**
    - The notebook sets a rule of thumb: if the model is calling more than 5-7 tools per run, consider simplifying
    - More tools means more decisions for the model, which degrades reliability
    - You can push logic into instructions instead of adding another tool
    - This is the tool-heavy vs instruction-heavy tradeoff from Part 3

- **Instruction-heavy vs tool-heavy agents**
  - **Term:** Two different design philosophies — one relies on callable functions to do the work, the other relies on detailed system instructions and the model's own reasoning.
  - **Analogy:** Tool-heavy is like a cook who follows precise recipes with specialized gadgets. Instruction-heavy is like a cook who has deep culinary knowledge and can improvise with basic equipment.
  - **What students need to understand:**
    - Tool-heavy is best for deterministic logic: calculations, API calls, data retrieval
    - Instruction-heavy is best for judgment tasks: writing, analysis, classification
    - If instructions exceed 500 words, look for places to push logic into tools or split into multiple agents
    - Most real agents are a mix — the question is where the balance sits

## Key SDK pattern

This notebook is primarily conceptual and decision-oriented, not code-heavy. The one code pattern it introduces is the **model comparison evaluation template** — a pattern for deciding whether to upgrade from `MODEL` to `REASONING_MODEL`:

```python
from agents import Agent, Runner
from config import MODEL, REASONING_MODEL

test_cases = [
    {"input": "...", "expected": "..."},
]

async def eval_model(model: str) -> dict:
    """Run test set against one model, return quality and latency."""
    import time
    results = []
    agent = Agent(name="TestAgent", instructions="...", model=model)
    for case in test_cases:
        start = time.time()
        result = await Runner.run(agent, input=case["input"])
        elapsed = time.time() - start
        results.append({
            "output": result.final_output,
            "latency": elapsed,
        })
    return results
```

- `model=model` — the key variable; you run the same agent and test set against both `MODEL` and `REASONING_MODEL` and compare
- `test_cases` — your golden test set from M03C; this is what makes the comparison objective rather than gut-feel
- `time.time()` wrapping — captures latency per case so you can weigh quality vs speed vs cost
- The template is printed but not executed (the actual `await` calls are commented out) — it is a reference pattern, not a live demo

## The demos and what each shows

**Part 1 (Model Selection):** Walks through when to use `MODEL` (`gpt-5-mini`) vs `REASONING_MODEL` (`gpt-5`) vs parallel specialists. The key message is that `MODEL` is the default for everything — tool selection, structured output, single-turn tasks, latency-sensitive agents. `REASONING_MODEL` is reserved for orchestration, multi-step planning, and ambiguous judgment calls. The code cell prints a model comparison template that students can copy into their own projects. Emphasize on camera: "Don't switch models based on intuition. Run your M03C golden test set on both and let the eval decide."

**Part 2 (Single Agent vs Multi-Agent):** A decision guide, no code. The test is simple: can you write clear instructions for one agent that solves the problem? If the instructions become contradictory or the agent can't do two things well simultaneously, that is when multi-agent earns its complexity. Reference M05A (handoffs), M05C (parallel execution), and M05D (debate/critique) as the specific multi-agent patterns students already know.

**Part 3 (Tool-Heavy vs Instruction-Heavy):** Presents both philosophies with concrete rules of thumb — more than 5-7 tools per run means simplify, more than 500 words of instructions means push logic into tools or split agents. Emphasize on camera: "Tools are not more 'production' than instructions — a well-crafted instruction set often outperforms a tool-heavy agent on judgment tasks."

**Part 4 (Memory and Sessions):** Decision guide for when to add sessions (M06A), persistent memory (M06B), or vector memory (M06C). The test: does the agent produce wrong or incomplete answers without memory? If no, skip it. Emphasize that vector memory (ChromaDB) beats SQLite when retrieval should be by meaning rather than exact key.

**Part 5 (MCP):** Decision guide for MCP vs `@function_tool`. The test: does a pre-built MCP server exist that does what you need? If yes, MCP is worth it. If you would be writing the server yourself, use `@function_tool`. Mention `cache_tools_list=True` as a practical tip for MCP startup performance.

**Part 6 (Avoiding Unnecessary Complexity):** The core philosophy of the notebook in one section. Presents the five-step sequence: start simple, evaluate, identify specific failures, add minimum complexity, re-evaluate. Lists five specific complexity traps to avoid. Emphasize on camera: "The agents that work in production are almost always simpler than they look in demos."

**Part 7 (Choosing the Right Pattern):** Two reference tables. The first is a quick-decision table mapping each pattern (reasoning model, multi-agent, sessions, guardrails, MCP, etc.) to "use when" and "skip when" criteria. The second is a full course reference mapping every need to its pattern and notebook number. This is the section students will screenshot and keep. Walk through it slowly on camera.

**Part 8 (Suggested Next Projects):** Six project ideas, each tagged with the specific patterns it exercises — personal research assistant, codebase Q&A agent, meeting notes processor, customer support bot, data analysis assistant, local file assistant. Encourage students to pick the one that matches a real problem they already have.

## Gotchas worth knowing before recording

- This notebook has almost no runnable code — just the model comparison template in one cell and the design exercise. Do not expect live demo output beyond `print()` statements. Plan the recording as a conceptual walkthrough, not a code-along.
- The model comparison template is printed, not executed. The `await` calls are commented out. If you try to run them live you will get errors because `test_cases` has placeholder `"..."` values. Mention this on camera: "This is a template you copy into your own project and fill in with real test cases."
- The `MODEL` and `REASONING_MODEL` constants are set to `gpt-5-mini` and `gpt-5` — make sure these match what M01A established. If students see different model names they will be confused.
- The two reference tables in Part 7 use HTML `<div>` wrappers for left-alignment in Jupyter. They render fine in notebooks but may look odd in other Markdown viewers. Do not worry about this on camera.
- The practice exercise (Exercise 1) is a pure design exercise with no runnable code — just `TODO` comments asking students to reason through decisions. Set expectations on camera: "There is no right answer here. The goal is to practice the decision process before writing code."
- Part 7 references every notebook in the course by number (M03A-2, M04A, M05A, etc.). Double-check that these notebook numbers still match the actual filenames if anything was renumbered.

## How it connects to adjacent notebooks

**Builds on M09A (Project Structure & CLI):** M09A moved students from notebooks to real project structure — config, tools, and agent logic in separate modules. M09B now asks: "Which patterns go into that project?" On camera, say something like: "Now that you know how to structure a project, the question is what to put in it."

**Builds on the entire course:** This notebook explicitly references patterns from M03C (golden test sets, evaluation), M05A (handoffs), M05B (agents as tools), M05C (parallel execution), M05D (debate/critique), M06A (sessions), M06B (persistent memory), M06C (vector memory), M07A (guardrails), M07B (prompt injection), M07C (human-in-the-loop), M07D (tracing), and M08A-M08C (MCP). When walking through the reference tables, briefly remind students where each pattern lives: "If you need a refresher on guardrails, that is M07A."

**Leads into M09C (Deploying Agents with Gradio):** The next and final notebook wraps an agent in a Gradio chat interface and deploys to Hugging Face Spaces. On camera, set it up: "You now know which patterns to use and how to structure the project. Next, we will give it a real UI and deploy it."
