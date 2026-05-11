

---

# M06B: Persistent Memory — Recording Prep

## What this notebook teaches
This notebook teaches students how to make agent memory survive kernel restarts by writing session state to a SQLite file on disk. It solves the problem introduced in M06A — where in-memory sessions vanish the moment the notebook restarts — by showing that one additional parameter (`db_path`) turns ephemeral memory into durable memory. Students also learn how to manage that persistent memory over time: summarizing noisy histories, correcting stale facts, and knowing what should never be stored.

## Concepts to explain on camera

- **SQLite:**
  - A lightweight database that stores everything in a single file on your computer — no server, no installation, no configuration.
  - **Analogy:** It's like a notebook saved as a file on your desktop. You don't need to run a database server — the file *is* the database.
  - **What students need to understand:**
    - SQLite is already built into Python — no `pip install` needed for the database itself
    - The `.sqlite` file created by this notebook is the entire database — you can see it in your file explorer
    - Deleting that file deletes all stored memory (demonstrated in the cleanup cell)
    - It's a real database used in production by apps like Firefox and iOS, not a toy

- **Session ID:**
  - A string you choose (like `"user_alex"`) that acts as the lookup key for a specific user's conversation history in the database.
  - **Analogy:** It's like a folder label in a filing cabinet. The cabinet is the SQLite file, and the label tells you which folder to pull out. Same label, same folder. Different label, different folder.
  - **What students need to understand:**
    - Same session ID + same `db_path` = agent remembers everything from before
    - Change either one and you get a completely blank session
    - You choose the ID — it can be a username, a UUID, or any string
    - One SQLite file can hold many different session IDs (like many folders in one cabinet)

- **Context window:**
  - The maximum amount of text (measured in tokens) that the model can "see" at once — including the full conversation history stored in the session.
  - **Analogy:** It's like the agent's working desk. Everything in the session gets spread out on the desk. If there's too much paper, things fall off the edge and the agent loses track.
  - **What students need to understand:**
    - Every turn you add to a persistent session adds tokens to what gets sent to the model
    - There's a hard limit — once history exceeds the context window, older items get dropped or quality degrades silently
    - This is why the summarize → clear → store pattern exists — it compresses a messy desk into a tidy summary
    - Quality degradation from a bloated session is subtle — the agent won't throw an error, it just starts getting worse

- **Persistent memory vs in-memory sessions:**
  - In-memory sessions (M06A) live in RAM and vanish on restart. Persistent memory writes to disk and survives indefinitely.
  - **Analogy:** In-memory is a whiteboard — powerful during the meeting, erased when you leave the room. Persistent is a journal — you close it, come back tomorrow, and everything's still there.
  - **What students need to understand:**
    - The API is nearly identical — you're adding one parameter (`db_path`)
    - Persistence is automatic after every turn — no manual "save" call needed
    - The tradeoff is that you now need to manage what accumulates (Parts 3 and 4)
    - In-memory is still better for throwaway demos and testing

## Key SDK pattern

The primary pattern this notebook introduces is `SQLiteSession` with a `db_path`:

```python
session = SQLiteSession(
    "user_alex",
    db_path="memory.sqlite"
)

result = await Runner.run(assistant, input="My name is Alex and I'm learning Python.", session=session)
```

Parameter breakdown:
- **`"user_alex"`** — The session ID. This is the lookup key that identifies which user's conversation history to load. You pick this string. Same ID + same file = same memory.
- **`db_path="memory.sqlite"`** — The path to the SQLite file on disk. If the file doesn't exist, it's created automatically. If it does exist, the session loads existing history from it. This is the *only* difference from the in-memory `SQLiteSession("user_alex")` pattern in M06A.
- **`session=session`** — Passed to `Runner.run()` exactly as in M06A. The Runner reads history from the session before the call and writes the new turn back after.

The reload pattern is the same constructor call:
```python
reloaded_session = SQLiteSession(
    "user_alex",            # same ID
    db_path="memory.sqlite"  # same file
)
```
No special "load" method — reconstructing with matching arguments reconnects to existing history.

Two session inspection methods are also introduced:
- **`await session.get_items()`** — Returns all stored items (messages) in the session. Used to inspect size and content.
- **`await session.clear_session()`** — Deletes all items from this session ID. Used in the summarize → clear → store pattern.

## The demos and what each shows

**Part 1: Persistent Sessions with SQLite** — This is the core "aha" demo. You create a `SQLiteSession` with `db_path="memory.sqlite"`, tell the agent "My name is Alex and I'm learning Python," then simulate a restart by creating a *new* `SQLiteSession` variable (`reloaded_session`) with the same ID and path. When you ask "What do you know about me?" the agent recalls Alex's name and Python interest. Emphasize on camera: the variable is brand new, the kernel could have restarted, but the memory file on disk has everything. Point out both the session ID `"user_alex"` and the file path `"memory.sqlite"` — change either one and you'd get a blank session.

**Part 2: Storing User Preferences** — This demo shows a practical use case: a writing assistant that remembers communication style. The user tells `prefs_agent` they want "short answers, bullet points over prose, and plain language — no jargon." In a later session (recreated with same ID `"user_prefs_demo"` and path `"preferences.sqlite"`), the user asks "Explain what a REST API is" and the agent should respond in bullet-point, jargon-free format. Emphasize on camera: the instructions tell the agent to "remember the user's communication preferences and apply them in every response" — this is where instructions and persistent memory work together. Without the instruction, the agent might store the preference text but not actively apply it.

**Part 3: Avoiding Noisy Memory** — This is a three-step demo that teaches the summarize → clear → store pattern. First, you build a noisy session (`"user_noisy_demo"`) by running 5 messages about a Python web scraper project. You inspect with `get_items()` and show how many items accumulated. Then a separate `summarizer` agent extracts just the key facts into a bullet list. Finally, you call `clear_session()` and store only the summary back. The before/after item count proves the compression worked. Emphasize on camera: this is proactive maintenance — don't wait for the agent to start giving bad answers. Mention the "every 20 turns" heuristic from the explanation cell. Also call out that the summarizer is a *separate* agent with its own instructions focused purely on extraction — a nice callback to M05's agents-as-tools concept.

**Part 4: What NOT to Store — Privacy and Correction** — Two things happen here. First, the Keep/Drop table and security warning set the conceptual frame — emphasize the table on camera and read the security note aloud. Second, the correction demo uses a `correction_agent` whose instructions say "treat the most recent statement as the current truth." The user says "I work as a data analyst," then corrects to "I'm now a machine learning engineer," then asks "What's my current role?" The agent should return the updated role. Emphasize: the old fact doesn't disappear from the session — both are stored — but the instructions tell the agent to prioritize recency. If contradictions accumulate over many sessions, apply the Part 3 pattern to reset. The user-controlled forgetting section is text-only (no code demo) — explain it verbally and connect it back to the summarize → clear → store pattern.

**Cleanup** — The final cell deletes all four `.sqlite` files. Mention on camera: "These files ARE the persistent storage. Deleting them is like shredding the journal — completely fresh start." This also proves the point that the database is just a file.

## Gotchas worth knowing before recording

- **The `reloaded_session` variable name might confuse students.** It's just a new Python variable pointing at the same underlying data. Make clear on camera: "This is simulating what would happen after a restart — we're creating a fresh variable, but because the session ID and file path match, we get the same memory back."

- **`await` is required on `get_items()` and `clear_session()`.** These are async methods. If you forget `await`, you'll get a coroutine object printed instead of the actual items. This will look strange on camera.

- **The noisy session demo uses the generic `assistant` agent (from Part 1) — not a dedicated agent.** This is fine but could look inconsistent since Part 2 and Part 4 create purpose-built agents. Just be aware you're reusing `assistant` from earlier.

- **`history_text` truncates each item to 200 characters** with `str(item)[:200]`. If students look closely, they'll notice this truncation. Mention it briefly: "We're trimming each item to keep the summarizer input manageable — in production you'd want a smarter chunking strategy."

- **The correction demo works because both the old and new facts are in the session and the instructions say to prefer the latest.** If a student asks "but doesn't the old fact confuse the agent?" — the answer is: sometimes, yes, especially if the session grows long. That's why the Part 3 summarize pattern exists as the cleanup mechanism.

- **Four different `.sqlite` files are created** (`memory.sqlite`, `preferences.sqlite`, `noisy.sqlite`, `correction.sqlite`). Each demo uses a separate file so they don't interfere. Students might wonder why not use one file with different session IDs — that would work too, but separate files make the demos independent.

- **The `get_items()` output format might vary.** The notebook prints `str(item)[:100]` which depends on the `__str__` representation of session items. If the output looks different from what you expect, don't panic — the item count is the important thing.

- **No explicit `save()` call exists.** Students coming from traditional database patterns might expect one. Emphasize: "SQLiteSession saves automatically after every turn — there's no manual save step."

- **The summarizer agent runs without a session** — `Runner.run(summarizer, input=...)` has no `session=` parameter. This is intentional — it's a one-shot utility call. Worth noting briefly so students see the contrast.

## How it connects to adjacent notebooks

**Builds on M06A (Sessions & Conversation State):** This notebook directly extends M06A. On camera, say something like: "In M06A we built sessions that remembered within a single kernel session — great for demos, but everything disappeared on restart. Now we're adding one parameter to make that memory permanent." Students should already understand `SQLiteSession` without `db_path`, session IDs, and `Runner.run()` with `session=`. The `get_items()` and `clear_session()` methods may have been previewed in M06A — if so, reference that; if not, introduce them here as new.

**Enables M06C (Vector Memory with ChromaDB):** The Key Takeaways cell explicitly sets up the distinction: "Use SQLite for exact facts; use vector memory for meaning." On camera, say: "SQLite is perfect when you know exactly which session to load — you look up by ID. But what if you want to find memories that are *similar* to a question, even if the words don't match exactly? That's semantic search, and that's what we'll build in M06C with ChromaDB and embeddings." This gives students a clear reason to keep going.

**Forward references to M07B (Prompt Injection & Tool Safety):** The privacy/security note in Part 4 says "Prompt injection and input validation are covered in M07B." On camera, briefly mention: "We'll cover security properly in Module 7 — for now, the rule is simple: don't store anything in agent memory that you wouldn't want exposed if the system were compromised."

---