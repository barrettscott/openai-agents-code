# AI Agents with Python & the OpenAI Agents SDK

## Setup Guide

![Download from Resources tab](images/course-setup-link.png)

---

## What We're Setting Up

| Tool | Purpose |
|------|---------|
| **Python 3.11+** | The language the course runs on |
| **Course Libraries** | OpenAI Agents SDK, Pydantic, ChromaDB, Gradio, and more — all installed from `requirements.txt` |
| **VS Code** | Where you'll open notebooks and run agent code |
| **VS Code Extensions** | Python + Jupyter support for running notebooks |

---

## Before You Begin

> [!IMPORTANT]
> Follow these steps in order — each one depends on the previous.

**Your setup checklist:**

- [ ] Step 1: Open your terminal
- [ ] Step 2: Download course materials
- [ ] Step 3: Verify Python 3.11 or higher
- [ ] Step 4: Create your virtual environment and install packages
- [ ] Step 5: Install VS Code
- [ ] Step 6: Open the course in VS Code

---

## 🖥️ Step 1: Open Your Terminal

<details>
<summary><b>🪟 Windows</b></summary>

1. Press the `Windows key` on your keyboard
2. Type "PowerShell"
3. Press Enter

</details>

<details>
<summary><b>🍎 macOS</b></summary>

1. Press `Cmd + Space` to open Spotlight search
2. Type "Terminal"
3. Press Enter

</details>

<details>
<summary><b>🐧 Linux</b></summary>

1. Press `Ctrl + Alt + T` (most distributions)
2. Or search for "Terminal" in your application menu

</details>

---

## 📦 Step 2: Download Course Materials

Download `openai-agents.zip` from the Resources tab in the course player.

![Download from Resources tab](images/download-resources.png)

> [!NOTE]
> The commands below assume the file is in your `📁 Downloads` folder and is named exactly `openai-agents.zip`.

![Terminal commands to unzip files](images/dir_path.png)

<details>
<summary><b>🪟 Windows</b></summary>

Run the commands below to create a `courses` folder and extract the course files:

```powershell
cd "$env:USERPROFILE"
mkdir -Force courses
cd courses
Expand-Archive -Path "$env:USERPROFILE\Downloads\openai-agents.zip" -DestinationPath . -Force
cd openai-agents
```

</details>

<details>
<summary><b>🍎 macOS & 🐧 Linux</b></summary>

Run the commands below to create a `courses` folder and extract the course files:

```bash
mkdir -p ~/courses
cd ~/courses
unzip ~/Downloads/openai-agents.zip
cd openai-agents
```

</details>

When extracted, you should see a folder named `openai-agents`.

![Directory Path](images/dir_path2.png)

**Your folder structure should look like this:**

![Folder structure](images/folder-structure.png)

---

## 🐍 Step 3: Verify Python 3.11 or Higher

Check whether Python is already installed and which version:

**🪟 Windows:**
```powershell
python --version
```

**🍎 macOS & 🐧 Linux:**
```bash
python3 --version
```

If you see Python 3.11 or higher, **skip to Step 4**.

### If Python is NOT installed (or too old):

<details>
<summary><b>🪟 Windows</b></summary>

1. Go to [python.org/downloads](https://www.python.org/downloads/)
2. Download the latest Python 3.11 (or higher) Windows installer
3. Run the installer and **check "Add Python to PATH"** at the bottom of the first screen
4. Close PowerShell completely and reopen it

</details>

<details>
<summary><b>🍎 macOS</b></summary>

Easiest path: install from [python.org/downloads](https://www.python.org/downloads/) — the universal2 macOS installer works on both Intel and Apple Silicon Macs.

If you already use Homebrew: `brew install python@3.11`

After installation, close Terminal and reopen it.

</details>

<details>
<summary><b>🐧 Linux</b></summary>

```bash
sudo apt install python3.11 python3.11-venv   # Ubuntu / Debian
sudo dnf install python3.11                   # Fedora
sudo pacman -S python                          # Arch (usually 3.11+ already)
```

</details>

### ✅ Verify Installation

Close your terminal completely and reopen it before running this — Python may not be on PATH otherwise.

**🪟 Windows:**
```powershell
python --version
```

**🍎 macOS & 🐧 Linux:**
```bash
python3 --version
```

If you see `Python 3.11.x` (or higher), you're ready for Step 4.

---

## 🌿 Step 4: Create Your Virtual Environment

**Navigate to the course folder:**

**🪟 Windows:**
```powershell
cd "$env:USERPROFILE\courses\openai-agents"
```

**🍎 macOS & 🐧 Linux:**
```bash
cd ~/courses/openai-agents
```

**Confirm you can see the requirements file:**

**🪟 Windows:**
```powershell
dir
```

**🍎 macOS & 🐧 Linux:**
```bash
ls
```

You should see `requirements.txt` in the output. If you don't, go back and make sure you navigated to the correct folder.

**Create the virtual environment:**

```bash
python3 -m venv .venv
```

This creates a `.venv` folder inside the course directory. It isolates the course's packages from the rest of your system, so they won't conflict with anything else you have installed.

**Activate the virtual environment:**

**🪟 Windows (PowerShell):**
```powershell
.\.venv\Scripts\Activate.ps1
```

> [!NOTE]
> If PowerShell blocks this with an "execution policy" error, run this once: `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`, then try the activate command again.

**🍎 macOS & 🐧 Linux:**
```bash
source .venv/bin/activate
```

Your prompt should now start with `(.venv)`.

**Install course packages (2–5 minutes):**

```bash
pip install -r requirements.txt
```

**What's being installed:**

| Package | Purpose |
|---------|---------|
| OpenAI Agents SDK | Core library for every notebook |
| python-dotenv | Secure API key loading |
| ChromaDB | Vector memory (Module 6) |
| Gradio | Chat UI deployment (Module 9) |
| FastAPI + uvicorn | API wrapper (Module 9) |
| ipykernel | Lets VS Code see this environment |

> [!TIP]
> If `pip install` fails, see Troubleshooting at the bottom of this page.

**Register the kernel for VS Code:**

```bash
python -m ipykernel install --user --name openai-agents --display-name "openai-agents"
```

This registers the virtual environment as a Jupyter kernel so VS Code can find it in the notebook kernel picker as `openai-agents`.

---

## 💻 Step 5: Install VS Code

1. Go to [code.visualstudio.com](https://code.visualstudio.com)
2. Click the download button for your operating system
3. Run the installer and follow the default options
4. Open VS Code when installation is complete

---

## 📂 Step 6: Open the Course in VS Code

1. In VS Code, go to **File → Open Folder**
2. Navigate to your `courses/openai-agents` folder
3. Click **Open**

![Directory Path](images/file-open-folder.png)

You should see all the course notebooks and files in the Explorer panel on the left.

> [!NOTE]
> **Trust prompt:** VS Code will ask whether you trust the authors — click **"Yes, I trust the authors"**. This only appears once.

![Directory Path](images/vscode-trust-authors-dialog.png)

> [!NOTE]
> **Extensions:** VS Code will prompt you to install recommended extensions (Python and Jupyter) — click **Install**. If no prompt appears, continue to the next step.

![Directory Path](images/vscode-install-recommended-extensions.png)

![Directory Path](images/vscode-workspace-recommendations-python-jupyter.png)

**Open the first notebook** from the Explorer panel: `01_Environment_Check.ipynb`

**Select the kernel:** The kernel indicator in the top-right should show **openai-agents**.

- If it doesn't, click **Select Kernel → Python Environments → openai-agents**

---

## 📍 What's Next

1. **Notebook 01** — Verify your Python environment and packages
2. **Notebook 02** — Connect to OpenAI, set up your API key, and run your first agent

---

## 🔧 Troubleshooting

<details>
<summary><b>"Command not found" / "not recognized" errors</b></summary>

- Close terminal/PowerShell completely and reopen it
- Make sure Python is on PATH — re-run the installer with "Add Python to PATH" checked (Windows)
- Restart your computer

> **Note:** For notebooks, selecting the `openai-agents` kernel in VS Code is what matters. You only need to activate the virtual environment when running course commands directly in a terminal.

</details>

<details>
<summary><b>venv activation fails on Windows (execution policy error)</b></summary>

PowerShell sometimes blocks scripts by default. Run this once to allow signed local scripts:

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

Then retry `.\.venv\Scripts\Activate.ps1`.

</details>

<details>
<summary><b><code>pip install</code> fails</b></summary>

- Make sure you've activated the virtual environment first — your prompt should start with `(.venv)`
- Upgrade pip itself: `python -m pip install --upgrade pip`
- Clear pip cache if downloads are corrupted: `pip cache purge`
- Network/proxy issues: try again on a different connection
- **🪟 Windows:** Disable antivirus temporarily if installation hangs

</details>

<details>
<summary><b>VS Code can't find the openai-agents kernel</b></summary>

- Close VS Code completely, reopen it, and try the kernel picker again — newly created environments sometimes need a moment to appear
- Press `Cmd+Shift+P` (Mac) or `Ctrl+Shift+P` (Windows/Linux), type **"Reload Window"**, and press Enter
- Confirm the kernel is registered: run `jupyter kernelspec list` from the activated venv — you should see `openai-agents` in the output
- If missing, re-run: `python -m ipykernel install --user --name openai-agents --display-name "openai-agents"`

</details>

<details>
<summary><b>Notebook won't run / kernel errors</b></summary>

- Check the kernel indicator in the top-right shows `openai-agents` — if not, select it
- Click the kernel indicator → **Restart** to restart the kernel
- Close the notebook tab and reopen it

</details>

<details>
<summary><b>Import errors for libraries</b></summary>

- Make sure `openai-agents` is selected as the kernel (not the system Python or another environment)
- Open VS Code's integrated terminal, navigate to the course folder, activate the venv, and run: `pip install -r requirements.txt`
- Click the kernel indicator → **Restart** after updating

</details>

<details>
<summary><b>Platform-specific issues</b></summary>

**🪟 Windows:**
- **Permission errors:** Try running PowerShell as Administrator
- **Path issues:** Your files are in `C:\Users\YourName\courses\openai-agents`

**🐧 Linux:**
- **`python3 -m venv` fails:** install the venv module — `sudo apt install python3.11-venv` (Ubuntu/Debian)
- **Missing build tools:** install via your distro's package manager — e.g. `sudo apt install build-essential` (Ubuntu/Debian), `sudo dnf groupinstall "Development Tools"` (Fedora), `sudo pacman -S base-devel` (Arch)
- **`unzip` or `wget` not found:** install via your distro's package manager — e.g. `sudo apt install unzip wget` (Ubuntu/Debian)

**🍎 macOS:**
- **Xcode issues:** Run `xcode-select --install` if you get compiler errors
- **Chip confusion:** Use Activity Monitor to check if you have Intel or Apple Silicon

</details>

<details>
<summary><b>Still stuck?</b></summary>

1. Restart your computer
2. Make sure you followed each step in order
3. Copy the error message and paste it into an AI assistant (Claude, ChatGPT, Gemini, Grok) — they're great at debugging setup issues
4. Post to the Q&A with your error message and output

</details>
