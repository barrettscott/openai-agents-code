# AI Agents with Python & the OpenAI Agents SDK

## Setup Guide

![Download from Resources tab](images/course-setup-link.png)

---

## What We're Setting Up

| Tool | Purpose |
|------|---------|
| **Conda** | Manages Python and course libraries |
| **Course Libraries** | OpenAI Agents SDK, Pydantic, ChromaDB, Gradio, and more — all installed from `environment.yml` |
| **VS Code** | Where you'll open notebooks and run agent code |
| **VS Code Extensions** | Python + Jupyter support for running notebooks |

---

## Before You Begin

> [!IMPORTANT]
> Follow these steps in order — each one depends on the previous.

**Your setup checklist:**

- [ ] Step 1: Open your terminal
- [ ] Step 2: Download course materials
- [ ] Step 3: Install Conda
- [ ] Step 4: Create your course environment
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
> The commands below assume the file is in your **Downloads** folder and is named exactly `openai-agents.zip`.

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

## 🐍 Step 3: Install Conda

Conda installs Python and manages all the libraries we'll need for the course.

**Check if Conda is already installed:**

```bash
conda --version
```

> [!TIP]
> If you see something like `conda 23.1.0`, **skip to Step 4** — you're already set.

### If Conda is NOT installed:

<details>
<summary><b>🪟 Windows</b></summary>

1. Go to the [Miniconda Windows installer page](https://www.anaconda.com/docs/getting-started/miniconda/install/windows-gui-install) and download the **Windows 64-Bit Graphical Installer**
2. Run the installer. When asked who to install for, choose **"Just Me"** (the default)
3. Accept the defaults until **Advanced Installation Options**, then check this box:
   - ☑️ **"Add Miniconda3 to my PATH environment variable"**

   > We're checking this so `conda` works directly in PowerShell throughout the course. The installer labels it "not recommended" — we're doing it on purpose for a simpler workflow.

   Leave this unchecked:
   - ☐ "Register Miniconda3 as my default Python 3.x"

4. Complete installation, close PowerShell completely and reopen it

</details>

<details>
<summary><b>🍎 macOS</b></summary>

1. Go to the [Miniconda download page](https://docs.conda.io/en/latest/miniconda.html)
2. **Apple Silicon (M1/M2/M3/M4):** Download the macOS Apple Silicon (arm64) `.pkg` installer
3. **Intel Macs:** Use this direct link: [Miniconda3-latest-MacOSX-x86_64.pkg](https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-x86_64.pkg)

   > *Not sure which Mac you have? Click Apple menu → About This Mac and look for Chip or Processor.*

4. Double-click the downloaded file and follow the installation steps
5. Close Terminal completely and reopen it

</details>

<details>
<summary><b>🐧 Linux</b></summary>

```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
```

- Accept the license and choose install location (default is fine)
- When asked "Do you wish the installer to initialize Miniconda3?", type `yes`
- Close terminal and reopen it

</details>

### ✅ Verify Installation (Everyone)

> [!WARNING]
> Close your terminal completely and reopen it before running this — conda won't be found otherwise.

```bash
conda --version
```

If you see a version number (e.g., `conda 24.1.2`), you're ready for Step 4.

---

## 🌿 Step 4: Create Your Course Environment

**Navigate to the course folder:**

**🪟 Windows:**
```powershell
cd "$env:USERPROFILE\courses\openai-agents"
```

**🍎 macOS & 🐧 Linux:**
```bash
cd ~/courses/openai-agents
```

**Confirm you can see the environment file:**

**🪟 Windows:**
```powershell
dir
```

**🍎 macOS & 🐧 Linux:**
```bash
ls
```

You should see `environment.yml` in the output. If you don't, go back and make sure you navigated to the correct folder.

**Create the environment (5–15 minutes depending on internet speed):**

```bash
conda env create -f environment.yml
```

**What's being installed:**

| Package | Purpose |
|---------|---------|
| Python 3.11 | The language the course runs on |
| OpenAI Agents SDK | Core library for every notebook |
| python-dotenv | Secure API key loading |
| ChromaDB | Vector memory (Week 4) |
| Gradio | Chat UI deployment (Week 5) |

> [!TIP]
> If this fails, run `conda clean --all` then try again.

**Verify the environment was created:**

```bash
conda env list
```

You should see `openai-agents` in the list.

**Register the kernel for VS Code:**

```bash
conda run -n openai-agents python -m ipykernel install --user --name openai-agents --display-name "openai-agents"
```

*This registers the environment as a Jupyter kernel so VS Code can find it in the notebook kernel picker.*

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

> [!NOTE]
> **Extensions:** VS Code will prompt you to install recommended extensions (Python and Jupyter) — click **Install**. If no prompt appears, continue to the next step.

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
- Make sure conda is initialized: run `conda init zsh` (macOS) or `conda init bash` (Linux) or `conda init powershell` (Windows), then close and reopen terminal
- Restart your computer

> **Note:** For notebooks, selecting the `openai-agents` kernel in VS Code is what matters. You only need `conda activate openai-agents` when running course commands directly in a terminal.

</details>

<details>
<summary><b>Environment creation failed</b></summary>

- Run `conda clean --all` then try again
- If the environment already exists: `conda env remove -n openai-agents -y` then recreate it
- **🪟 Windows:** Disable antivirus temporarily if installation hangs

</details>

<details>
<summary><b>Conda activate doesn't work</b></summary>

- Close terminal and reopen it
- Run `conda init zsh` (macOS) or `conda init bash` (Linux) or `conda init powershell` (Windows)
- Close and reopen terminal, then retry

</details>

<details>
<summary><b>VS Code can't find the openai-agents kernel</b></summary>

- Close VS Code completely, reopen it, and try the kernel picker again — newly created environments sometimes need a moment to appear
- Press `Cmd+Shift+P` (Mac) or `Ctrl+Shift+P` (Windows/Linux), type **"Reload Window"**, and press Enter
- Try **Select Kernel → Python Environments** and look for `openai-agents` in the list

</details>

<details>
<summary><b>Notebook won't run / kernel errors</b></summary>

- Check the kernel indicator in the top-right shows `openai-agents` — if not, select it
- Click the kernel indicator → **Restart** to restart the kernel
- Close the notebook tab and reopen it

</details>

<details>
<summary><b>Import errors for libraries</b></summary>

- Make sure `openai-agents` is selected as the kernel (not `base` or another environment)
- Open VS Code's integrated terminal, navigate to the course folder, and run: `conda env update -n openai-agents -f environment.yml`
- Click the kernel indicator → **Restart** after updating

</details>

<details>
<summary><b>Platform-specific issues</b></summary>

**🪟 Windows:**
- **Permission errors:** Try running PowerShell as Administrator
- **Path issues:** Your files are in `C:\Users\YourName\courses\openai-agents`

**🐧 Linux:**
- **Permission errors:** Avoid using `sudo` with conda commands
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
