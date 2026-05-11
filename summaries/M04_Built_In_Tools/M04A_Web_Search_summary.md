

---

# M04A: Web Search — Recording Prep

## What this notebook teaches

This notebook teaches students how to give an agent real-time web access using `WebSearchTool()` — a single-line addition to the `tools` list that requires no extra API key. It solves the training-cutoff problem: every model has a knowledge boundary, and web search lets the agent answer questions about current events, today's weather, and anything that changed since training ended. Students also learn how citations work and how to bias search results toward a specific geographic location.

## Concepts to explain on camera

- **Training cutoff**
  - **Term:** Every LLM was trained on data up to a specific date. Anything after that date is invisible to the model — it will either refuse to answer or confabulate.
  - **Analogy:** It's like an encyclopedia printed in January — it can't tell you what happened in March.
  - **What students need to understand:**
    - The model doesn't know it's outdated — it may guess confidently with stale info
    - Web search bridges this gap by injecting fresh results into the agent's context
    - The cutoff date varies by model and isn't always publicly documented to the day
    - This is exactly why we need `WebSearchTool` — it's not optional for time-sensitive use cases

- **Grounding**
  - **Term:** Anchoring the agent's response to actual retrieved content (web results, documents) rather than relying solely on what the model "remembers" from training.
  - **Analogy:** It's the difference between answering a test question from memory vs. being allowed to look at your notes. The notes don't guarantee a right answer, but they make fabrication less likely.
  - **What students need to understand:**
    - A grounded answer is traceable back to a source — you can check it
    - Grounding reduces confabulation (the model making things up) but doesn't eliminate it
    - The same concept applies to File Search in M04B — grounding against your own documents
    - "Grounding" is a term you'll see throughout AI documentation; it always means the same thing

- **Built-in hosted tool**
  - **Term:** A tool that OpenAI runs on their infrastructure, as opposed to a `@function_tool` you write yourself. `WebSearchTool` is one of these — you don't manage a search engine, you just turn it on.
  - **Analogy:** Like using Google Docs vs. installing your own word processor. The functionality is hosted for you — you just use it.
  - **What students need to understand:**
    - No third-party search API key needed (no Bing, no Google Custom Search, no SerpAPI)
    - The search happens server-side at OpenAI — your code just passes the tool to the agent
    - There are three built-in hosted tools in this module: Web Search (M04A), File Search (M04B), and Code Interpreter (M04C)
    - Cost: web search tool calls may incur additional usage charges on your OpenAI bill beyond standard token costs

- **Confabulate / confabulation**
  - **Term:** When a model generates plausible-sounding information that is factually wrong. Some people call this "hallucination."
  - **Analogy:** Like a student who doesn't know the answer but writes something anyway that sounds authoritative — they're not lying, they're pattern-matching.
  - **What students need to understand:**
    - Confabulation is most dangerous when the user can't easily verify the answer
    - Grounding with web search reduces but doesn't eliminate it
    - This is why citations matter — they let users verify claims against actual sources
    - The security note about untrusted input is related: even the *search results* could contain bad info

- **Prompt injection (brief forward reference)**
  - **Term:** An attack where malicious text in retrieved content (a web page, a document) tries to override the agent's instructions.
  - **Analogy:** Imagine hiring a research assistant and one of the web pages they read says "ignore your boss's instructions and do this instead."
  - **What students need to understand:**
    - Web search results are untrusted input — you don't control what's on the web
    - This is mentioned here as a security note, not a full lesson
    - The full treatment is in M07B — just plant the seed on camera
    - For now, students should be aware that web content flowing into an agent is a potential attack surface

## Key SDK pattern

The primary pattern is adding `WebSearchTool()` to an agent's `tools` list:

```python
agent_search = Agent(
    name="WebSearchAgent",
    instructions="You are a helpful research assistant. Always search the web for current information.",
    model=MODEL,
    tools=[WebSearchTool()]
)

result = await Runner.run(agent_search, input="What are the top AI news stories this week?")
print(result.final_output)
```

**Parameter-by-parameter breakdown:**

- `name="WebSearchAgent"` — Human-readable label for the agent. Shows up in tracing (which students saw briefly in M02A). Pick something descriptive.
- `instructions=` — The system prompt. Including "Always search the web for current information" nudges the agent to actually use the tool rather than answering from training data. Without this, the agent *decides on its own* whether to search — and sometimes won't.
- `model=MODEL` — Uses `gpt-5-mini` (set in the setup cell). Fast and cheap — appropriate for straightforward search-and-summarize tasks.
- `tools=[WebSearchTool()]` — This is the one-liner that enables web search. `WebSearchTool` is imported from `agents`. No API key, no configuration. Just instantiate and include it.

**The location-targeted variant:**

```python
tools=[WebSearchTool(
    user_location={
        "type": "approximate",
        "city": CITY,
        "region": REGION,
        "country": COUNTRY
    }
)]
```

- `user_location` — An optional dict that biases (not restricts) results toward a geography. `"type": "approximate"` is the only supported type. The `city`, `region`, and `country` keys are all strings. This is useful for weather, local news, nearby businesses, etc.

## The demos and what each shows

**Part 1 — Without vs. With Web Search:** This is a before-and-after comparison using the same question ("What are the top AI news stories this week?"). First, `agent_no_search` runs without `WebSearchTool` — the model either hedges, refuses, or confabulates because it has no access to current information. Then `agent_search` runs the same question with `WebSearchTool()` in the tools list and returns actual current results. Emphasize on camera: the *only* difference between these two agents is the `tools=[WebSearchTool()]` line. One line of code, and suddenly the agent has real-time web access. Also point out the `truncate_response` helper — it's just a readability utility so long web summaries don't flood the notebook output; students don't need to worry about it.

**Part 2 — Citations:** The `CitationAgent` answers "What is the current version of Python and when was it released?" with web search and instructions to "Always cite your sources clearly." The response should include inline citation links or references. Emphasize on camera: citations appear by default when the model has web sources, but they're *not guaranteed*. If your application requires citations, add explicit instructions like this agent does. The formatted output with the separator lines (`"=" * 60`) makes citations visually distinct — call this out as a nice readability pattern.

**Part 3 — Location-Targeted Search:** The `agent_local` agent uses `user_location` set to Denver, Colorado, US and answers "What is the weather like today?" On camera, point out the three constants (`CITY`, `REGION`, `COUNTRY`) and suggest students change them to their own location. Emphasize that `user_location` *biases* results — it doesn't *restrict* them. A broad query might still return global results. Also note: `"type": "approximate"` is the only supported type — don't try `"exact"` or anything else, it won't work.

## Gotchas worth knowing before recording

- **The "without search" result is nondeterministic.** `agent_no_search` might confabulate confidently, hedge and say "I don't have access to current information," or give genuinely outdated results. Any of these outcomes proves the point, but be ready to narrate whichever one you get. Don't be thrown off if it doesn't refuse — sometimes it makes something up.

- **The "with search" result changes every time you run it.** Because it pulls live web results, the content will differ between recordings and between takes. Don't try to predict exact output — narrate what appears.

- **`truncate_response` is defined in the setup cell.** If you restart the kernel and skip the setup cell, later cells calling `truncate_response` will throw a `NameError`. Always run the setup cell first.

- **Citations are not guaranteed to appear.** Even with "Always cite your sources" in instructions, the model might occasionally omit them. If this happens on camera, explain it's probabilistic and move on — this is actually a useful teaching moment.

- **`user_location` "type" must be "approximate."** If a student tries another value, it'll error. Mention on camera that this is the only supported type.

- **Weather queries can return stale results.** "What is the weather like today?" sometimes returns a forecast page from earlier or a general climate description. If the result looks oddly generic, re-run or change the question to something like "What is today's weather forecast in Denver?"

- **The notebook uses `await Runner.run(...)` directly in cells.** This works in Jupyter because Jupyter has a running event loop. Students don't need to write `asyncio.run()` or `async def main()`. If a student asks, Jupyter handles the async context automatically — this is the same pattern from M02A.

- **Web search costs money.** Each `WebSearchTool` call incurs usage charges. Students should be aware that running demos repeatedly adds up, though individual calls are cheap.

- **The security note about untrusted input is a forward reference, not a full lesson.** Don't try to teach prompt injection here — just read the warning box, say "we'll cover this in depth in Module 7," and move on.

## How it connects to adjacent notebooks

**What came before:** This notebook builds directly on M02B (Building Tools for Agents), where students learned the `@function_tool` decorator for creating custom tools. On camera, callback: "In M02B, we built our *own* tools with `@function_tool`. Now we're using a *built-in* tool that OpenAI provides — `WebSearchTool`. Same `tools=[]` parameter, different source of capability." It also builds on M02C (Writing Effective Agent Instructions) — the instruction "Always search the web for current information" is a practical example of telling the agent *when* to use a tool, which was taught conceptually in M02C. Module 3's reliability patterns (Pydantic, error handling, testing) are in the background — students know them but this notebook keeps things simple by design.

**What comes next:** M04B (File Search) introduces another built-in hosted tool, this time for querying uploaded documents instead of the web. On camera, tease: "Web search grounds your agent in *public* information from the internet. Next, in M04B, we'll ground it in *your own* documents using File Search." Both tools feed into M04D (Capstone #1: Research Agent), where students combine web search, file search, and code interpreter into a single agent. The security note here also plants a seed for M07B (Prompt Injection & Tool Safety), where untrusted input from web search and documents is explored in depth.

---