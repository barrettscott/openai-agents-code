

---

# M02B: Building Tools for Agents — Recording Prep

## What this notebook teaches

This notebook teaches students how to give agents real-world capabilities by turning ordinary Python functions into tools using the `@function_tool` decorator. It matters because without tools, agents can only generate text from training data — they can't compute, fetch live data, or interact with external systems. Students learn that the decorator auto-generates a tool schema from the function's name, docstring, and type hints, so there's no manual JSON wiring.

## Concepts to explain on camera

- **`@function_tool` decorator**
  - **Term:** A one-line annotation you put above a Python function that registers it as a tool the agent can call. The SDK reads the function's name, docstring, and type hints and builds a JSON schema behind the scenes.
  - **Analogy:** Like putting a name tag on a volunteer at an event — the coordinator (the agent) reads the tag to figure out who can help with what.
  - **What students need to understand:**
    - You write a normal Python function; the decorator does the rest
    - The docstring is required — it's literally what the agent reads to decide whether to call the tool
    - Type hints on every parameter are required — without them the schema generation fails
    - The return value should be a string (the agent consumes tool output as text)

- **Tool schema / automatic schema generation**
  - **Term:** A structured description of what a tool does, what inputs it expects, and what types those inputs are. The SDK generates this automatically from your function signature — you never write JSON by hand.
  - **Analogy:** Like an auto-generated instruction manual — Python reads your function's name, docstring, and type hints and writes the manual for the agent.
  - **What students need to understand:**
    - The schema is what the model actually sees when deciding which tool to use
    - Function name → tool name the model reads
    - Docstring → tool description the model reads
    - Type hints → parameter types and names the model reads
    - If any of these are missing or vague, the agent may not call the tool correctly

- **`tools=[]` parameter on Agent**
  - **Term:** The list you pass to `Agent()` that tells it which tools are available. If a tool isn't in this list, the agent doesn't know it exists.
  - **Analogy:** Like handing someone a toolbox — they can only use what's inside it.
  - **What students need to understand:**
    - You pass the decorated function objects directly: `tools=[calculate, get_current_datetime]`
    - Order doesn't matter — the agent picks based on relevance, not position
    - An empty list (or omitting the parameter) means the agent has no tools
    - You can give different agents different tool sets

- **`eval()`**
  - **Term:** A built-in Python function that takes a string and executes it as Python code. Used here as a quick way to evaluate math expressions like `"847 * 293 / 17"`.
  - **Analogy:** Like handing someone a calculator that can run any code — convenient but dangerous if you let strangers type into it.
  - **What students need to understand:**
    - It's used here only to keep the demo short and focused on tools, not math parsing
    - Never use `eval()` on untrusted input in production — it can execute arbitrary code
    - The notebook calls this out explicitly; mention `simpleeval` as the safe alternative
    - This security concern comes back in M07B (Prompt Injection & Tool Safety)

## Key SDK pattern

The primary pattern is defining a tool and wiring it to an agent:

```python
@function_tool
def calculate(expression: str) -> str:
    """Evaluate a mathematical expression and return the result."""
    try:
        result = eval(expression)
        return str(result)
    except Exception as e:
        return f"Error: {e}"

agent_with_tools = Agent(
    name="CalculatorAgent",
    instructions="You are a helpful assistant. Use the calculate tool for all math.",
    model=MODEL,
    tools=[calculate]
)

result = await Runner.run(agent_with_tools, input="What is 847 times 293 divided by 17, rounded to 2 decimal places?")
print(result.final_output)
```

Parameter-by-parameter:

- **`@function_tool`** — Decorator that registers `calculate` as a tool. Reads `calculate` as the tool name, the docstring `"Evaluate a mathematical expression..."` as the description the model sees, and `expression: str` as the single required parameter with type `string`.
- **`name="CalculatorAgent"`** — Human-readable label for this agent (shows up in traces and logs).
- **`instructions="You are a helpful assistant. Use the calculate tool for all math."`** — System prompt. The second sentence explicitly steers the agent to use the tool rather than attempting mental math.
- **`model=MODEL`** — Uses `gpt-5-mini`, the course default set in the Setup cell.
- **`tools=[calculate]`** — Passes the decorated function object. The agent now knows this tool exists and can call it.
- **`await Runner.run(...)`** — Same runner pattern from M02A. Sends the input, the agent decides to call `calculate`, gets the result, and formulates `final_output`.

## The demos and what each shows

**Part 1 — Before/After comparison (calculator tool).** This is the hook for the whole notebook. First, `agent_no_tools` is asked `"What is 847 times 293 divided by 17, rounded to 2 decimal places?"` and the model attempts mental math — it may get it right, it may not, and the point is you *can't rely on it*. Then the same question goes to `agent_with_tools` which has the `calculate` tool, and the answer is guaranteed correct because actual Python math runs. Emphasize on camera: "Language models reason about math, they don't compute it." The before/after contrast is the entire motivation for tools. If the no-tools agent happens to get it right on your recording take, say so — "It got lucky this time, but you can't guarantee that."

**Part 2 — A tool with no parameters (date/time).** The `get_current_datetime` tool takes zero arguments and just returns `datetime.now()` formatted as a readable string. The agent is asked `"What day of the week is it today?"` and calls the tool to get live data. Emphasize: tools don't have to take input. Some just give the agent access to information it couldn't otherwise have — the model's training data has no idea what today's date is. This is a clean, minimal example that shows the decorator works the same way regardless of parameter count.

**Part 3 — How the agent decides which tool to call.** A single `agent_multi` has *both* `calculate` and `get_current_datetime` in its `tools=[]` list. Three questions are fired at it: a math question (`"What is 1024 divided by 32?"`), a time question (`"What time is it right now?"`), and a general knowledge question (`"What is the capital of France?"`). The agent should route the first to `calculate`, the second to `get_current_datetime`, and answer the third from its own knowledge with no tool call. Emphasize: the agent reads the tool names and docstrings to decide. No tool needed means no tool called. This is where you reinforce that **the docstring is part of your prompt** — it's not just documentation for humans, it's the description the model uses to make routing decisions.

**Writing Good Tool Descriptions section.** This is a markdown-only teaching moment between Parts 2 and 3. Call out the three things the agent reads: function name, docstring, and type hints. Give the concrete anti-examples from the notebook: `tool1` is bad, `calculate` is good; `get_info` is bad, `get_current_datetime` is good. This sets up students to write their own tools well in the exercises and in every future module.

## Gotchas worth knowing before recording

- **The no-tools agent might get the math right.** `gpt-5-mini` may correctly compute `847 * 293 / 17`. If it does, don't panic — say "It got it right this time, but language models don't reliably compute math. Try a harder expression and you'll see it break." The point is reliability, not that it always fails.
- **`eval()` will alarm experienced developers.** The notebook has a security warning markdown cell. Read it aloud or paraphrase it on camera: "We're using `eval()` here to keep the demo simple. Never do this with untrusted input. Use `simpleeval` or a proper math parser in real code." This preempts Q&A questions.
- **Tools must return strings.** The `calculate` tool wraps its result in `str(result)`. If you forget this and return a number, the SDK may still work (it coerces), but the notebook is teaching the explicit pattern. Mention this when walking through the code.
- **Docstrings are required.** If you accidentally delete the docstring during a live edit, `@function_tool` will raise an error. The troubleshooting section covers this, but be aware so it doesn't surprise you mid-recording.
- **Type hints are required on all parameters.** Omitting `expression: str` and writing just `expression` will break schema generation. This is the most common student mistake — call it out proactively.
- **`await` at top level only works in Jupyter.** Students who try to run this in a plain `.py` file will get a `SyntaxError`. The troubleshooting section mentions this, but if you address it on camera you'll reduce support questions.
- **The "capital of France" question in Part 3 should produce no tool call.** If the model unexpectedly calls a tool, it's not broken — it's a teaching moment about how tool descriptions influence routing. But it's unlikely with these clear docstrings.
- **Variable naming: `agent_no_tools`, `agent_with_tools`, `agent_datetime`, `agent_multi`.** Each demo section creates a new agent variable. Don't accidentally reuse a previous one — the names are intentionally different to avoid confusion.

## How it connects to adjacent notebooks

**Builds on M02A (How Agents Work).** Students already know `Agent`, `Runner.run()`, `result.final_output`, and the `MODEL` constant. On camera, say something like: "In the last notebook we built agents that could only answer from their training data. Now we're giving them the ability to actually *do things*." You can assume students understand the basic Agent/Runner pattern — no need to re-explain it.

**Leads into M02C (Writing Effective Agent Instructions).** This notebook shows that `instructions` steer tool usage (e.g., `"Use the calculate tool for all math"`), but M02C goes deep on writing durable system instructions — when to ask clarifying questions, when NOT to use a tool, response style, refusal patterns. On camera, tease: "We wrote simple instructions here like 'use this tool for math.' In the next notebook, we'll learn how to write instructions that make agents reliable in complex situations — and trust me, instructions become critical once we hit multi-agent systems."

**The `@function_tool` pattern introduced here is the foundation for everything that follows.** Custom tools appear in M03B (error handling inside tools), M04D (capstone), M05 (multi-agent tools), M07 (guardrails and safety around tools), and M08 compares `@function_tool` to MCP servers. This is a good moment to say: "Every agent you build in this course will use tools. This decorator is something you'll see in almost every notebook from here on."

---