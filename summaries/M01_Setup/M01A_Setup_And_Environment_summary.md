

---

# M01A: Setup & Environment — Recording Prep

## What this notebook teaches
This notebook verifies that the student's local development environment is correctly configured before any agent code is written. It runs three diagnostic checks — Python version, required packages, and Jupyter kernel — and gives students clear pass/fail feedback with fix-it instructions. This matters because a broken environment is the #1 reason students get stuck in the first hour of a course, and catching it here means zero debugging during the actual agent-building notebooks.

## Concepts to explain on camera

- **Conda environment**
  - **Term:** A self-contained Python installation where you can install specific package versions without affecting anything else on your computer. The course uses one called `openai-agents`.
  - **Analogy:** Think of it like a sealed toolbox. You put exactly the tools you need inside, and nothing from your other projects leaks in or gets overwritten.
  - **What students need to understand:**
    - You must *activate* the environment (`conda activate openai-agents`) before launching JupyterLab — otherwise you'll get the wrong Python and missing packages
    - The environment is defined by `environment.yml` — that file pins every package version so everyone has the same setup
    - If packages are missing later, `conda env update -f environment.yml` brings you back to the correct state
    - The environment name `openai-agents` will show up in terminal prompts and kernel selectors — that's how you confirm it's active

- **Jupyter kernel**
  - **Term:** The behind-the-scenes engine that actually runs the code in your notebook cells. When you open a notebook, it connects to a kernel, and that kernel determines which Python installation and packages are available.
  - **Analogy:** The notebook is a steering wheel; the kernel is the engine underneath. If the steering wheel is connected to the wrong engine, nothing works the way you expect.
  - **What students need to understand:**
    - A notebook can *look* open and fine but be connected to the wrong kernel — you'll get `ImportError` even though you installed everything
    - You can change kernels without closing the notebook: Kernel → Change Kernel
    - The correct kernel for this course will have `openai-agents` in its name or path
    - After switching kernels, re-run all cells from the top

- **`sys.executable` / `sys.prefix`**
  - **Term:** Python variables that tell you *which* Python installation is currently running your code. `sys.executable` is the path to the Python binary; `sys.prefix` is the path to the environment folder.
  - **Analogy:** It's like asking "which kitchen am I cooking in right now?" — even if you have three kitchens in the house, this tells you which one is active.
  - **What students need to understand:**
    - The kernel check in this notebook looks for the string `"openai-agents"` inside these paths — that's how it confirms the right environment is active
    - If neither path contains `"openai-agents"`, you likely launched JupyterLab before activating the conda environment
    - You don't need to memorize these variables; the notebook does the check for you

- **`environment.yml`**
  - **Term:** A file that lists every package and version the course needs. Conda reads it and installs everything in one shot.
  - **Analogy:** It's a recipe card — hand it to conda and it assembles the exact same pantry every time, on any machine.
  - **What students need to understand:**
    - Students should have already created the environment from this file in the setup lecture before opening this notebook
    - `conda env update -f environment.yml` is the fix-all command when something is missing
    - They never need to `pip install` individual packages manually for this course

- **`python-dotenv`**
  - **Term:** A small Python package that loads secrets (like API keys) from a `.env` file into your code, so you never have to hardcode them.
  - **Analogy:** It's like a locked drawer next to your desk — your code knows to check the drawer for the key, rather than taping the key to the wall where anyone can see it.
  - **What students need to understand:**
    - This package is verified here but not *used* until M01B, where you'll create the `.env` file and store your OpenAI API key
    - The `.env` file should never be committed to GitHub — it contains secrets
    - `load_dotenv()` is the function call that reads the file; you'll see it at the top of nearly every notebook going forward

- **`openai-agents` (the package, imported as `agents`)**
  - **Term:** The OpenAI Agents SDK — the Python library that lets you create AI agents with tools, handoffs, guardrails, and more. It's the core library for this entire course.
  - **Analogy:** If the OpenAI API is a phone line to a smart assistant, the Agents SDK is a control room that lets you wire up multiple assistants, give them tools, and coordinate their work.
  - **What students need to understand:**
    - You install it as `openai-agents` but import it as `agents` — that naming mismatch trips people up
    - The version is printed during the check (`agents.__version__`) — this matters because the SDK is actively evolving
    - This notebook only checks it's installed; you'll first *use* it in M01B and then properly learn it in M02A

## Key SDK pattern
There is no SDK usage pattern in this notebook — it is purely a diagnostic/verification notebook. The primary code pattern is the **environment check pattern**, which students will see used as a sanity-check approach throughout the course:

```python
try:
    import agents
    print(f"✅ openai-agents {agents.__version__} installed")
except ImportError:
    print("❌ openai-agents missing — run: conda env update -f environment.yml")
    all_ok = False
```

**What to explain about this pattern:**
- `try/except ImportError` is how you safely check whether a package exists without crashing the notebook
- `agents.__version__` prints the installed version — useful for debugging ("I'm on version X and getting this error")
- The `all_ok` boolean acts as a cumulative flag: if *any* package fails, the final block prints consolidated fix instructions
- This same `try/except` defensiveness shows up later when building tool functions (M03B) — get comfortable seeing it now

## The demos and what each shows

**Check 1: Python Version.** This cell imports `sys` and reads `sys.version_info.major` and `sys.version_info.minor` to confirm Python 3.10+. On camera, emphasize that the check is looking for *at least* 3.10 — the actual environment ships 3.11.13, so students will see something like "Python 3.11 detected." If someone sees 3.9 or lower, it means they didn't activate the conda environment. Point out the helpful error message that says exactly what to do: `conda activate openai-agents`.

**Check 2: Required Packages.** This cell attempts to import three packages — `agents` (the OpenAI Agents SDK), `dotenv` (for loading API keys from `.env`), and `pydantic` (for structured data validation). Call out on camera that `openai-agents` is installed as one name but imported as `agents` — students will see this discrepancy and wonder if something is wrong. Also call out that `pydantic` shows up this early even though it's formally taught in M03A — it's a dependency of the SDK itself, so it must be present from day one. Mention the `all_ok` flag pattern: all three must pass, and if any fail, the consolidated fix instructions appear.

**Check 3: Jupyter Kernel.** This cell checks whether the string `"openai-agents"` appears in `sys.executable` or `sys.prefix`. On camera, explain that this is a heuristic — it's checking the *path* to the running Python. If someone launched JupyterLab from a different environment, this cell will warn them. Note that it prints ⚠️ (warning) not ❌ (failure) — it's possible to have everything working even if the path doesn't match exactly, but it's a strong signal something is off. Walk through the fix: Kernel → Change Kernel → select the openai-agents kernel.

**Bonus: Material Darker Theme.** This is an optional cosmetic section. On camera, mention it briefly — "If you want your JupyterLab to look like mine in the videos, here's how." Don't spend more than 15 seconds. Emphasize it's purely visual and has zero effect on code.

## Gotchas worth knowing before recording

- **The `agents` import name vs. `openai-agents` install name.** This *will* confuse students. Say it explicitly on camera: "You install `openai-agents` but import `agents` — different name, same thing."
- **The kernel check can give a false warning.** If a student installs everything correctly but their conda path uses a different casing or a symlink, the `"openai-agents" in sys.prefix.lower()` check might still say ⚠️. Reassure students: "If Checks 1 and 2 both pass, you're almost certainly fine even if Check 3 gives a warning."
- **`sys` is imported twice.** It's imported in both Check 1 and Check 3. This is fine — Python caches imports — but if a student notices, just say "importing it again is harmless; we do it so each check cell can run independently."
- **No `all_ok` summary for Check 1 or Check 3.** The `all_ok` flag only exists in Check 2 (packages). Checks 1 and 3 are standalone. Don't accidentally say "if all three pass" while pointing at the `all_ok` variable — it only tracks packages.
- **Students may not have run the setup lecture yet.** This notebook *assumes* the conda environment already exists. If a student jumps straight here, every check will fail. On camera, say: "If you haven't watched the setup lecture and created the conda environment yet, do that first — this notebook checks your work, it doesn't do the setup for you."
- **The `.lower()` call in Check 3.** On Windows, paths often have mixed casing. The `.lower()` is there to handle that. If you're recording on a Mac, the path will already be lowercase, but mention briefly that this handles Windows users too.
- **No model constants defined here.** The outline mentions `MODEL = "gpt-5-mini"` and `REASONING_MODEL = "gpt-5"` for M01A, but these do **not** appear in this notebook. They are likely defined in M01B or M02A. Don't mention model constants in this recording — wait for the notebook where they actually appear.
- **`pydantic` is checked but not explained.** Students might wonder why it's here. One sentence: "Pydantic is a data validation library — we'll learn it properly in Module 3, but the Agents SDK depends on it, so it needs to be installed now."

## How it connects to adjacent notebooks

**What came before:** Module 0 (slides only) introduced what agents are, the chatbot-to-autonomous-agent spectrum, and course prerequisites. On camera, you can say: *"In the welcome video, I mentioned you'd need Python and an API key. This notebook is where we confirm Python and packages are ready — next notebook we'll get the API key."*

**What comes next:** M01B (OpenAI Setup) is the direct continuation. That notebook creates the `.env` file, stores the API key, and makes the first actual agent call to verify the connection works. On camera, close with: *"All three checks passed — your environment is solid. In the next notebook, we'll set up your OpenAI account, get your API key, and talk to your first agent."* The `python-dotenv` package verified here is the one that powers the `.env` loading in M01B. The `agents` package verified here is what M01B will use for the first `Agent()` and `Runner.run()` call.

---