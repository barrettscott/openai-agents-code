

---

# M03B: Error Handling & Recovery — Recording Prep

## What this notebook teaches

This notebook teaches students how to prevent their agents from crashing when things go wrong — bad inputs, flaky APIs, missing data, and OpenAI API failures. It introduces four layers of defense: try/except inside tools, retry loops for transient failures, fallback tools via agent instructions, and wrapping `Runner.run()` itself. This matters because every tool students build from here forward (web search, file search, MCP servers) will eventually fail, and an agent without error handling is a demo that breaks on the first real user.

## Concepts to explain on camera

### Transient failure
- **Term:** A failure that's temporary — it works if you try again in a few seconds. Network timeouts, rate limits, and brief API outages are transient. A missing database record is *not* transient.
- **Analogy:** Like getting a busy signal on a phone call. The person is there, the line is just jammed. Try again in a minute and it'll go through.
- **What students need to understand:**
  - Not all errors deserve a retry — only transient ones
  - Retrying a "product not found" error will never succeed; retrying a "service unavailable" might
  - The notebook's `unreliable_api_call` with its 60% failure rate simulates this
  - Rate limits from OpenAI are the most common transient failure students will actually hit

### Idempotent (and why retry safety matters)
- **Term:** An action that gives the same result whether you do it once or five times. Reading a price is idempotent. Placing an order is not — retry and you get five orders.
- **Analogy:** Checking your bank balance is idempotent — checking it three times doesn't change anything. Transferring $100 is not — doing it three times moves $300.
- **What students need to understand:**
  - The retry pattern in Part 3 is safe because `unreliable_api_call` is a read operation
  - Never blindly retry tools that create, send, charge, or delete
  - The notebook's warning callout ("Only retry read-only or idempotent actions") is a production rule, not just theory
  - This concept returns in M07B when we discuss write tools and side effects

### Exponential backoff with jitter
- **Term:** Instead of waiting the same amount of time between retries, you wait longer each time (1s, 2s, 4s) and add a small random delay so multiple retrying clients don't all hammer the server at the same moment.
- **Analogy:** If everyone in a crowded store tries to check out at exactly 2:00, the line jams. If everyone waits a random extra 1-30 seconds, traffic smooths out.
- **What students need to understand:**
  - The notebook uses a fixed `wait_seconds = 1` for simplicity
  - The notebook explicitly says to use exponential backoff with jitter in production
  - Students don't need to implement it here — just know the concept exists
  - Libraries like `tenacity` handle this automatically (worth mentioning but not teaching)

### Graceful degradation
- **Term:** When a system provides a reduced but still useful response instead of failing completely. Cached data instead of live data. A partial answer instead of no answer.
- **Analogy:** If the restaurant's espresso machine is broken, they offer you drip coffee instead of telling you to leave.
- **What students need to understand:**
  - Part 4's weather agent demonstrates this — cached forecast instead of live data
  - The agent tells the user the data is cached, not live — transparency matters
  - "Partial information delivered gracefully beats a crash" is the design principle
  - This pattern shows up again in M05E where one specialist fails and the pipeline keeps going with partial results

### Runner-level failure
- **Term:** An error that happens not inside your tool, but in the communication between your code and the OpenAI API — authentication failures, rate limits, network issues, or structured output that can't be parsed.
- **Analogy:** Your tool is the kitchen. Runner-level failure is the delivery truck breaking down — the food is fine, it just can't get to the customer.
- **What students need to understand:**
  - Tool-level try/except doesn't catch these — the tool never even runs
  - You need a *separate* try/except around `Runner.run()` itself
  - This is what `run_safely()` in Part 5 demonstrates
  - Structured output parse failures (from M03A-2) also surface at this level

## Key SDK pattern

The primary pattern is **try/except inside a `@function_tool` that returns an error string instead of raising an exception**:

```python
@function_tool
def lookup_product(product_id: str) -> str:
    """Look up product details by ID."""
    try:
        product = PRODUCT_DB[product_id]
        stock_status = "in stock" if product["stock"] > 0 else "out of stock"
        return f"{product['name']}: ${product['price']:.2f} ({product['stock']} units {stock_status})"
    except KeyError:
        return f"Product '{product_id}' not found. Available products: {', '.join(PRODUCT_DB.keys())}"
    except Exception as e:
        return f"Error looking up product: {e}"
```

**What students need to understand about each piece:**

- **`@function_tool`** — same decorator from M02B; nothing new about how the tool is registered
- **`try:` block** — wraps the risky operation (`PRODUCT_DB[product_id]` can raise `KeyError`)
- **`except KeyError:`** — catches the specific, expected failure mode first; returns a *string* with helpful context (available product IDs), not a stack trace
- **`except Exception as e:`** — safety net for anything unexpected; always put this *after* specific catches
- **Return type is always `str`** — the agent receives this string as the tool's output. If you return an error message string, the agent sees it and can respond intelligently ("That product wasn't found, here are the available ones"). If you let the exception propagate, the entire `Runner.run()` crashes.

The secondary pattern is **wrapping `Runner.run()` itself**:

```python
async def run_safely(agent, user_input: str) -> str:
    try:
        result = await Runner.run(agent, input=user_input)
        return result.final_output
    except Exception as e:
        error_type = type(e).__name__
        print(f"    ⚠️  Run failed: {error_type}: {e}")
        return "I'm having trouble right now. Please try again in a moment."
```

- This catches API errors, auth failures, rate limits, network issues — anything the SDK raises
- The user gets a safe message; the developer gets the error type logged
- This is a different layer of defense than tool-level try/except — both are needed

## The demos and what each shows

**Part 1: What Happens Without Error Handling** — This is the "before" demo. `lookup_product_fragile` does a raw dictionary lookup with `PRODUCT_DB[product_id]` and no try/except. When the agent asks for product `P999` (which doesn't exist), the `KeyError` propagates up, `Runner.run()` fails, and the entire run crashes. The outer try/except in the demo cell catches it so the notebook doesn't halt, but the point is clear: the user gets nothing. Emphasize on camera: "This is what every unhandled tool error looks like — the agent crashes, the user gets an error instead of an answer."

**Part 2: Handling Errors Inside the Tool** — This is the "after" demo with the same scenario. `lookup_product` wraps the dictionary lookup in try/except and returns a helpful string when `P999` isn't found: the product ID that failed plus the list of available IDs. The demo runs two queries back-to-back — `P001` (valid, returns price and stock) and `P999` (missing, triggers the error path). The agent sees the error string and responds helpfully, suggesting alternatives. Emphasize: "Same bad input, completely different user experience. The agent keeps running and helps the user recover."

**Part 3: Retry Pattern** — `fetch_with_retry` wraps `unreliable_api_call` (which fails 60% of the time via `random.random() < 0.6`) in a retry loop with `max_attempts = 3` and `wait_seconds = 1`. The `random.seed(42)` makes the demo reproducible — you'll see specific attempts fail and succeed in the same order every time. The print statements inside the loop show each attempt's outcome in real time. Emphasize two things: (1) the retry logic lives entirely inside the tool — the agent never knows retries happened, and (2) after 3 failures, the tool returns a graceful message instead of crashing. Point out the warning callout about only retrying read-only/idempotent operations — "don't retry a 'place order' tool or you'll get three orders."

**Part 4: Fallback Strategy** — `get_live_weather` always returns the string `"Error: Weather API is currently unavailable"` (it never raises — it returns an error message). The agent's instructions tell it: if `get_live_weather` fails or returns an error, use `get_weather_forecast` (which returns cached data). When the student asks about London weather, the agent calls the live tool, sees the error string, then calls the fallback tool and gets `"[Cached forecast] Overcast with occasional rain, highs around 55°F"`. The instructions also tell the agent to be transparent about which source it used. Emphasize: "The fallback logic is in the instructions, not in code. The agent decides to try the second tool. This is the agent being an agent — making decisions based on context."

**Part 5: Runner-Level Recovery** — `run_safely()` wraps `Runner.run()` in a try/except. The demo just runs a normal question ("What is the capital of France?") and succeeds, showing the pattern works for the happy path. On camera, explain: "This will succeed because our API key is valid and the network is fine. But if any of that breaks — rate limit, auth failure, network timeout — the user gets 'I'm having trouble right now' instead of a stack trace." You can't easily force an API failure live, so explain the failure case verbally. Emphasize the log-internally-return-safely principle.

## Gotchas worth knowing before recording

- **`random.seed(42)` in Part 3 makes the retry demo deterministic** — if you re-run that cell multiple times without re-running the seed cell, you'll get different attempt patterns. If the demo doesn't match what you expect, re-run the seed cell first. Consider mentioning this on camera: "I'm setting a random seed so we all see the same result."

- **Part 4's `get_live_weather` returns a string, not raises an exception** — this is intentional and worth calling out. It simulates an API that returns an error message rather than throwing. The agent reads the string, recognizes it as an error, and falls back. If a student asks "why not raise an exception?" the answer is: because we already showed that pattern in Parts 1-2. This demo shows the agent making a *decision* based on tool output.

- **Part 5 will succeed every time** (unless your API key is actually broken) — there's no simulated failure here. You're showing the *pattern*, not the failure. Don't promise students they'll see an error; say "here's how we protect against it" and explain what would happen.

- **`time.sleep(1)` in the retry loop will cause a visible pause** — the notebook will feel frozen for 1-2 seconds per retry attempt. Mention this on camera: "You'll see a pause here — that's the sleep between retry attempts. In a notebook, it feels slow. In production, it saves you from hammering a struggling server."

- **The `PRODUCT_DB` dictionary is defined in Part 1 and reused in Part 2** — if you restart the kernel and jump straight to Part 2, it won't exist. Run cells in order.

- **`eval()` appears in Exercise 1** — the notebook explicitly marks it as "unsafe in production, fine for this exercise only." If a student asks, emphasize it's for learning the error-handling pattern, not for real calculator tools.

- **The `fragile_agent` demo uses an outer try/except** — this might confuse students into thinking they've "handled" the error. Clarify: "I wrapped this in try/except so the notebook doesn't crash, but the *agent run* still failed. The user got nothing useful. That's the problem we're solving."

- **Agent fallback behavior is non-deterministic** — the agent in Part 4 *should* call the fallback tool based on its instructions, but LLMs don't follow instructions 100% of the time. If the agent doesn't fall back on camera, it's an instruction-following issue. Re-running usually fixes it. You could strengthen the instructions if needed during recording.

## How it connects to adjacent notebooks

**What came before:** M03A-1 (Pydantic Basics) and M03A-2 (Structured Outputs) taught students how to define and enforce output structure. When you introduce Part 5's `run_safely()`, call back to structured outputs: "Remember when we used `output_type` to get structured output? If the model returns something that can't be parsed into your Pydantic model, that failure happens at the Runner level — and `run_safely()` catches it." Also call back to M02B (Building Tools): "We learned to build tools with `@function_tool` — now we're learning to make those tools bulletproof."

**What comes next:** M03C (Testing & Evaluating Agents) builds directly on this. Say on camera: "Now that our agents don't crash, how do we know they're actually working *correctly*? That's what testing and evaluation is for." Also plant a forward reference to M07D (Tracing & Observability): "When something does fail in a complex pipeline, tracing shows you exactly which tool failed and what it returned — we'll cover that in Module 7." The retry safety warning connects forward to M07B (Prompt Injection & Tool Safety) where write tools and side effects get full treatment.

---