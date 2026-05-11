

---

# M01B: OpenAI Setup — Recording Prep

## What this notebook teaches

This notebook walks students through creating an OpenAI account, generating an API key, storing it securely in a `.env` file, and verifying the connection by making their very first agent call. It matters because nothing else in the course works without a valid API key and a working environment — this is the gate that must open before any learning happens. It also establishes the `.env` pattern that every subsequent notebook depends on.

## Concepts to explain on camera

- **API Key**
  - **Term:** A unique secret string (starts with `sk-`) that OpenAI uses to identify your account and charge your credits when you make requests. Think of it as your personal password for the OpenAI API.
  - **Analogy:** Like a hotel key card — it identifies you, gives you access to specific services, and if someone else gets it, they can charge everything to your room.
  - **What students need to understand:**
    - Every API call costs money against your prepaid credit balance
    - The key is shown exactly once when you create it — copy it immediately
    - Never paste it into code cells, commit it to GitHub, or share it in screenshots
    - If compromised, revoke it immediately on the OpenAI dashboard and create a new one

- **.env File**
  - **Term:** A plain text file named `.env` that stores configuration secrets (like your API key) outside your code. Libraries like `python-dotenv` load these values into your program at runtime.
  - **Analogy:** Like a locked drawer next to your desk — your notebook knows to check the drawer for the key, but the key itself never sits out on the desk where anyone can see it.
  - **What students need to understand:**
    - The file is named exactly `.env` (starts with a dot, no file extension)
    - It lives in the project root (`openai-agents/`), NOT inside `M01_Setup/`
    - The format is `OPENAI_API_KEY=sk-proj-...` with no spaces around the `=` and no quotes
    - Files starting with `.` are hidden by default on macOS/Linux — you may need to toggle visibility

- **`load_dotenv()`**
  - **Term:** A function from the `python-dotenv` package that reads your `.env` file and makes its values available via `os.getenv()`. Without calling it, Python has no idea your `.env` file exists.
  - **Analogy:** Like plugging in a USB drive — the files are there, but your computer can't read them until you plug it in.
  - **What students need to understand:**
    - Must be called before you try to read `OPENAI_API_KEY`
    - The `dotenv_path=Path("..") / ".env"` parameter tells it where to look (one folder up from the notebook)
    - The `override=True` parameter means if you update your `.env` and re-run, the new value takes effect
    - Every notebook in this course will have a `load_dotenv()` call near the top

- **`Agent` and `Runner`**
  - **Term:** The two core classes from the OpenAI Agents SDK. `Agent` defines *who* the agent is (name, instructions, model). `Runner` actually *executes* the agent against a user input and returns a result.
  - **Analogy:** `Agent` is the job description you write for a new hire. `Runner` is telling them "OK, go do this task" and getting back their work.
  - **What students need to understand:**
    - These are imported from the `agents` package (installed in M01A as `openai-agents`)
    - `Agent` takes `name`, `instructions`, and `model` parameters
    - `Runner.run()` is called with `await` because it's asynchronous (Jupyter handles this automatically)
    - The response lives in `result.final_output` — this is where the agent's text answer is
    - This is a preview — M02A covers Agent, Runner, and `result.final_output` in full detail

- **Prepaid Credits / Billing**
  - **Term:** OpenAI requires you to add money to your account before you can make API calls. Each call deducts a small amount based on the model used and how much text is processed.
  - **Analogy:** Like a prepaid phone plan — you load $5 onto the account and each call uses a tiny fraction of that balance.
  - **What students need to understand:**
    - $5 is more than enough for the entire course
    - Budget alerts at Settings → Limits are recommended but are soft limits (won't auto-stop spending)
    - If credits run out, API calls return billing errors — the notebook's error handler catches this
    - `gpt-5-mini` (used throughout most of the course) is the cheapest model

## Key SDK pattern

This notebook introduces the foundational Agent → Runner → result pattern that every subsequent notebook builds on:

```python
from pathlib import Path
from dotenv import load_dotenv
from agents import Agent, Runner

load_dotenv(dotenv_path=Path("..") / ".env", override=True)

MODEL = "gpt-5-mini"

agent = Agent(
    name="SetupTest",
    instructions="You are a helpful assistant.",
    model=MODEL
)

result = await Runner.run(agent, input="Say 'Setup complete!' and nothing else.")

print(result.final_output)
```

Parameter breakdown:
- **`name="SetupTest"`** — A label for the agent. Shows up in tracing/logs. Can be any string. Students will see meaningful names like `"Researcher"` or `"Critic"` in later modules.
- **`instructions="You are a helpful assistant."`** — The system prompt that shapes the agent's behavior. Intentionally minimal here. M02C is entirely about writing effective instructions.
- **`model=MODEL`** — Which OpenAI model to use. Set to `"gpt-5-mini"` via the `MODEL` constant established in M01A. Mini is fast and cheap — the default for most of the course.
- **`input="Say 'Setup complete!' and nothing else."`** — The user message the agent responds to. Deliberately simple so the expected output is predictable and students can confirm it worked.
- **`result.final_output`** — The agent's text response. This is the one attribute students need right now; M02A will explore the `result` object more deeply.
- **`await`** — Required because `Runner.run()` is asynchronous. Jupyter notebooks handle `await` at the top level automatically; no `asyncio.run()` needed. Don't dwell on this — M05C covers async in detail.

## The demos and what each shows

**Step 1 (Markdown only — no code): Create OpenAI Account & Add Credits.** This is a walkthrough section with links. On camera, either show the OpenAI dashboard briefly or tell students to pause the video and follow the steps. Emphasize: add at least $5 in credits, set a budget alert at Settings → Limits, and note that the alert is a *soft* limit — it sends a notification but doesn't stop spending. This is the one place where students spend real money, so be clear and reassuring about costs.

**Step 2: Create Your .env File (interactive code cell).** The cell uses `getpass()` to accept the API key without displaying it on screen, validates that it starts with `"sk-"`, and writes it to `../.env`. If a `.env` already exists, it prompts the user to keep or replace it. On camera, emphasize the file location — it must be in the project root, one level up from the notebook. Point out the directory diagram in the markdown. Mention that `getpass` hides the input, so when you paste your key you won't see anything — that's normal. If the input box doesn't appear (some Jupyter environments have issues), there's a manual terminal alternative below. Show both paths briefly.

**Step 3: Verify Your Setup (dotenv load + key check).** This cell loads the `.env` file, checks that `OPENAI_API_KEY` exists and starts with `"sk-"`, checks it's not the placeholder text `"sk-proj-your-actual-key-here"`, and displays a masked version like `sk-proj-...xyz1`. Emphasize that the full key is never printed — this is intentional security hygiene. On camera, point out the masking: `f"{api_key[:8]}...{api_key[-4:]}"`. If students see ❌ here, the troubleshooting section at the bottom covers every common failure.

**Step 4: Verify Your Connection with Your First Agent Call.** This is the payoff moment. The cell creates an `Agent` named `"SetupTest"` with minimal instructions, runs it with `Runner.run()`, and prints `result.final_output`. The expected output is `"Setup complete!"`. The `try/except` block catches authentication errors, billing errors, and rate limits with specific fix instructions for each. On camera, celebrate this — it's the student's first working agent. Mention that M02A will unpack exactly how `Agent`, `Runner`, and `result.final_output` work. Don't go deep on the SDK here — just confirm it works.

**Final Check (comprehensive verification).** This cell runs three boolean checks: `.env` file exists, `openai-agents` package imports correctly (`from agents import Agent, Runner`), and API key is present and starts with `"sk-"`. It prints a pass/fail checklist and a 🎉 banner if everything passes. On camera, point out that this does NOT re-test the live connection — it just confirms the local setup. The live connection was already verified in Step 4. This is a quick "all systems go" confirmation students can re-run anytime.

## Gotchas worth knowing before recording

- **`getpass()` shows no characters while typing/pasting.** This will look like nothing is happening on camera. Say explicitly: "You won't see anything when you paste — that's the security feature working. Just paste and press Enter."
- **The `.env` file goes in the parent directory, not the notebook's directory.** The code uses `Path("..") / ".env"`. If a student creates `.env` inside `M01_Setup/`, nothing will load. The notebook has a directory diagram — point to it on camera.
- **`override=True` in `load_dotenv()`.** Without this, if a stale value is already in the environment (from a previous kernel session), updating `.env` won't take effect. This is already in the code but worth mentioning: "We use `override=True` so if you ever update your key, you just re-run this cell."
- **The budget alert is a soft limit.** Students might assume setting a $10 limit means spending stops at $10. Clarify: it's an email notification, not a hard cutoff.
- **`await Runner.run(...)` works in Jupyter but not in plain Python scripts.** Don't explain this in depth — just say "Jupyter lets us use `await` directly; when we move to scripts later in the course, we'll handle this differently." M05C and M09A address this.
- **The placeholder key check** (`"your-actual-key-here" in api_key`) catches students who copy the example literally from the manual instructions. Worth briefly mentioning: "If you see 'placeholder key detected,' you copied my example instead of your real key."
- **Windows `.env` encoding issues.** The manual PowerShell command uses `-Encoding ascii` for a reason — some Windows editors save `.env` with BOM or UTF-16 encoding that `python-dotenv` can't read. If a Windows student's key "looks right" but won't load, this is likely why. The notebook mentions this in troubleshooting.
- **No `import os` in the agent call cell.** The agent call cell (Step 4) doesn't need `import os` or `os.getenv` — the SDK reads `OPENAI_API_KEY` from the environment automatically once `load_dotenv()` has run. Students might wonder how the agent knows the key. Say: "The SDK automatically looks for `OPENAI_API_KEY` in your environment — that's why we loaded the `.env` file earlier."
- **The final check does NOT test the live API connection.** It only checks the file, the package, and the key format. The live test was Step 4. Don't let students think the final check replaces Step 4.

## How it connects to adjacent notebooks

**Builds on M01A (Setup & Environment):** M01A installed the `openai-agents` package and `python-dotenv`, created the conda environment, and introduced the `MODEL = "gpt-5-mini"` constant. On camera, you can say: "In the last notebook we set up our Python environment and installed the packages. Now we're going to get the API key that lets us actually talk to OpenAI."

**Leads into M02A (How Agents Work):** M02A is where students learn what `Agent`, `Runner`, and `result.final_output` actually do in detail, plus how `instructions` shape behavior and a light intro to tracing. On camera at the end of M01B, say: "We just made our first agent call, but we skipped over what `Agent`, `Runner`, and `result.final_output` actually mean. That's exactly what the next notebook covers — we'll take this apart piece by piece." The `.env` loading pattern introduced here (`load_dotenv(dotenv_path=Path("..") / ".env", override=True)`) will appear at the top of every notebook going forward — worth flagging so students expect it.

---