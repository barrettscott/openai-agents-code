

---

# M06A: Sessions & Conversation State — Recording Prep

## What this notebook teaches

This notebook teaches students why agents forget everything between `Runner.run()` calls and how to fix it with `SQLiteSession` — the SDK's built-in conversation history mechanism. It matters because every multi-turn assistant (chatbots, interview bots, onboarding flows) is broken without memory. By the end, students can create in-memory and file-based sessions, share sessions across agents, and inspect/clear session state.

## Concepts to explain on camera

- **Session (in the context of the Agents SDK):**
  - **Term:** A session is a container that stores the conversation history (user messages and agent responses) so the agent can see prior turns on the next `Runner.run()` call. Without one, every call is a blank slate.
  - **Analogy:** Imagine calling a help desk where the person has no notes from your last call — you have to re-explain everything. A session is the notes file they keep between calls.
  - **What students need to understand:**
    - Each `Runner.run()` call is stateless by default — the agent literally cannot see anything from prior calls
    - A session automatically loads history before the run and saves new turns after
    - The session ID (e.g., `"demo_session"`) is the unique key that identifies which conversation to load
    - Sessions store conversation turns, not long-term facts — that distinction is M06B's topic

- **SQLiteSession:**
  - **Term:** The SDK's built-in session implementation that uses SQLite (a lightweight database stored in a single file) to persist conversation history. With no `db_path`, it runs in-memory; with a `db_path`, it writes to disk.
  - **Analogy:** Think of it like a notebook. In-memory is writing in pencil on a whiteboard — gone when you erase. File-based is writing in a physical notebook — it's there when you come back tomorrow.
  - **What students need to understand:**
    - `SQLiteSession("id")` → in-memory, lost on kernel restart
    - `SQLiteSession("id", db_path="file.sqlite")` → file-based, survives restarts
    - The session ID is case-sensitive — `"Demo"` and `"demo"` are different sessions
    - You don't need to install SQLite separately — it's built into Python

- **Session ID:**
  - **Term:** A string you choose (like `"demo_session"` or `"persistent_demo"`) that uniquely identifies a conversation. Any `SQLiteSession` object created with the same ID (and same `db_path`) connects to the same history.
  - **Analogy:** Like a ticket number at a support desk — give the same number, get the same conversation record.
  - **What students need to understand:**
    - You pick the ID — it's just a string
    - For file-based sessions, both the ID and the `db_path` must match to reconnect to the same history
    - Different IDs = completely separate conversations, even in the same database file
    - This is how you'd give each user their own conversation in a real app

- **Context window:**
  - **Term:** The maximum amount of text (measured in tokens) that a model can process in a single call. Every prior conversation turn loaded from the session counts toward this limit.
  - **Analogy:** Like a desk that can only hold so many papers — if the conversation history is too long, some of it won't fit. The notebook mentions this is covered in depth in M06B.
  - **What students need to understand:**
    - Sessions replay the *entire* history on every call — long conversations use more tokens
    - More tokens = higher cost and eventually hitting the model's limit
    - This notebook flags the issue; M06B covers strategies for managing it
    - `clear_session()` is the brute-force solution shown here

## Key SDK pattern

The primary pattern is adding `session=session` to `Runner.run()`:

```python
# Create a session — in-memory by default
session = SQLiteSession("demo_session")

# Turn 1
result1 = await Runner.run(agent, input="My name is Alex and I work in finance.", session=session)

# Turn 2 — agent remembers Turn 1
result2 = await Runner.run(agent, input="What's my name and what do I do?", session=session)
```

**Parameter breakdown:**

- `"demo_session"` — The session ID. A string you choose. Any `SQLiteSession` created with this same string (and same `db_path`) connects to the same conversation history.
- `session=session` — Tells the Runner to load all prior conversation turns before this call and save the new turn after. Without this parameter, the call is stateless.
- For persistent sessions: `SQLiteSession("persistent_demo", db_path="demo_sessions.sqlite")` — The `db_path` tells SQLite to write to a file instead of keeping everything in memory. Both the ID and the path must match to reconnect.

Students need to understand: `session=` is the *only* change to their existing `Runner.run()` pattern. Everything else — `agent`, `input`, getting `result.final_output` — stays exactly the same as in Module 2.

## The demos and what each shows

**Part 1: Without Sessions — The Forgetting Problem.** This is the "before" demo. The agent is told `"My name is Alex and I work in finance."` in Turn 1, then asked `"What's my name and what do I do?"` in Turn 2. The agent can't answer because each `Runner.run()` is a clean slate. Emphasize on camera: this isn't a bug — it's how the SDK works by default. Every call is independent. Pause here and let students see the failure before showing the fix.

**Part 2: With Sessions — Automatic Memory.** This is the "after" demo using the same agent and similar inputs, but now with `session=session` added. Turn 1 introduces Alex, Turn 2 asks for recall (and succeeds), Turn 3 asks for AI tool recommendations based on Alex's finance background. Emphasize that the *only code difference* from Part 1 is two lines: creating the `SQLiteSession` and passing `session=session`. Point out that Turn 3 proves the agent sees the *entire* history, not just the previous turn. Call out the forward reference: session growth and context window management are in M06B.

**Part 3: Persistent Sessions — Surviving Restarts.** This demo creates a file-based session with `db_path="demo_sessions.sqlite"`, stores a preference ("My favorite programming language is Python"), then creates a *new* `SQLiteSession` object called `reconnected_session` with the same ID and path. The agent still remembers. On camera, emphasize that `persistent_session` and `reconnected_session` are different Python objects — but they connect to the same history because the ID and path match. Point out the `.sqlite` file appearing in the file browser. Mention that in a real app, this is how conversation survives between server restarts or notebook sessions.

**Part 4: Shared Sessions Across Agents.** Two different agents — `onboarding_agent` and `advisor_agent` — share a single `shared_session`. The onboarding agent collects Jordan's info ("marketing manager trying to learn AI tools"), then the advisor agent is asked "What should I focus on first?" and can give personalized advice because it sees the full conversation. Emphasize on camera: the session is a conversation store, not an agent store. Call out the tradeoff mentioned in the notebook — all turns are visible to every agent, so a specialist might see irrelevant context. Connect this to handoffs from M05A: this is how multi-agent systems maintain context without manual state passing.

**Part 5: Managing Session State.** Two utility methods: `get_items()` to inspect the session history (showing role and content for each item), and `clear_session()` to wipe it. The demo prints item count and a preview of each stored item, then clears and confirms 0 items remaining. Emphasize on camera: `get_items()` is your debugging tool — if the agent isn't remembering, check here first. `clear_session()` deletes history but keeps the session object usable.

**Cleanup.** The final cell deletes `demo_sessions.sqlite` using `Path("demo_sessions.sqlite").unlink(missing_ok=True)`. Mention this is good hygiene — the persistent file from Part 3 doesn't need to stick around.

## Gotchas worth knowing before recording

- **All cells use `await` directly** — this works in Jupyter notebooks but not in regular Python scripts. Don't get tripped up if a student asks; in a script you'd need `asyncio.run()`. This is the same pattern from Module 2 forward.
- **The Part 1 "forgetting" demo is nondeterministic** — the agent might say "I don't know your name" or it might hallucinate a name. Either outcome proves the point, but if it hallucinates, address it: "See, it guessed wrong because it has no memory of Turn 1."
- **In-memory sessions use SQLite's `":memory:"` mode** — there's no file created. If students look in the file browser after Part 2, they won't see anything. Only Part 3 creates a visible `.sqlite` file.
- **Session ID is case-sensitive** — `"persistent_demo"` and `"Persistent_Demo"` are different sessions. The troubleshooting section calls this out, but worth mentioning on camera during Part 3.
- **The `reconnected_session` in Part 3 is a simulation, not an actual kernel restart** — you're creating a new Python variable pointing to the same database. If you actually restart the kernel between the two cells, you'd also need to re-run the setup cell (imports, env loading, agent creation). Consider mentioning this: "In a real restart you'd re-run setup first, but the session data in the file survives."
- **`get_items()` returns raw dictionaries** — the code accesses `item.get("role")` and `item.get("content")`. The exact structure depends on the SDK version. If the output format looks slightly different from what's expected, don't panic — the session is working, the internal format may just vary.
- **The cleanup cell at the end deletes `demo_sessions.sqlite`** — if you want to demo persistence across an actual restart, don't run cleanup first. Consider skipping it during recording and running it after.
- **The `shared_session` in Part 4 is in-memory** (no `db_path`), so it won't survive a restart. This is intentional — it's demonstrating the sharing concept, not persistence.
- **Variable naming**: `result1`, `result2`, `result3` in Part 2 vs plain `result` in Parts 3-4. The notebook reuses `result` in later parts. Not a problem, but be aware if you need to reference earlier outputs.

## How it connects to adjacent notebooks

**What came before:** Module 5 (Multi-Agent Systems) introduced handoffs and shared context between agents. On camera, say something like: *"Remember in Module 5 when we built multi-agent systems with handoffs? The agents could pass control to each other, but between separate Runner.run() calls, everything was forgotten. Sessions fix that."* Also reference Module 2A where `Runner.run()` and `result.final_output` were introduced — this notebook adds exactly one parameter (`session=`) to that same pattern.

**What comes next:** M06B (Persistent Memory) covers storing long-term facts and preferences across *different* sessions — things like "the user prefers concise answers" that should persist even after clearing a session. On camera: *"Sessions store conversation history — the back-and-forth turns. But what if you want the agent to remember that you prefer Python, even in a brand-new conversation? That's persistent memory, and that's next in M06B."* Also note that the notebook explicitly defers context window management and session growth strategies to M06B. M06C (Vector Memory) will extend memory further with semantic search using ChromaDB.

---