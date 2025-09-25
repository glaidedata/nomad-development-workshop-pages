# Part 1: Setup & Environment

## Installation Paths

There are two ways to set up NOMAD development on Windows:

- **Path A (recommended):** Use **WSL2 (Windows Subsystem for Linux)** with Ubuntu
  → Provides a Linux environment close to production, fewer path and dependency issues. Could be more stable since most NOMAD developer run and test their developments in Linux.
- **Path B (alternative):** Install and run everything **directly in Windows**
  → Simpler if IT restrictions make WSL/Docker integration difficult, but can run into Windows-specific issues.

---

## Checking Software Requirements

Before we start with NOMAD distro-dev, let’s verify that all required tools are installed and working.
 Run the following checks in **Windows Terminal** (use **PowerShell** and/or **Ubuntu (WSL)** tabs depending on your chosen path). Type `wsl` in PoweShell to start WSL2.

---

### 1) Check WSL

**PowerShell:**

```
wsl --status
```

* ✅ Installed → shows version info (should include `WSL version: 2`)

* ❌ Not installed → IT must install WSL (admin rights)

**PowerShell (list distros):**

```console
wsl --list --verbose
```

* ✅ `Ubuntu` (or another distro) listed

* ❌ Missing → install with:

```console
wsl --install -d Ubuntu
```

---

### 2) Check Docker Desktop

**PowerShell:**

```python
docker --version
docker run hello-world
```

* ✅ Prints Docker version and shows “Hello from Docker!”

* ❌ Error → Docker Desktop not installed or not running

**If you use Path A (WSL), also check inside Ubuntu:**

```console
docker --version
docker run hello-world
```

* ❌ If daemon not reachable → enable **Ubuntu** under Docker Desktop → **Settings → Resources → WSL integration**, then retry

---

### 3) Check Git

**Ubuntu (Path A):**

```console
git --version
```

**Windows (Path B):**

```console
git --version
```

* ✅ Returns a version (e.g., `git version 2.x`)

---

### 4) Check Python

**Ubuntu (Path A):**

```console
python3 --version
```

**Windows (Path B):**

```console
python --version
```

* ✅ Returns `3.x.x`

* ⚠️ On Windows, ensure **Long Paths** are enabled if you’ll install lots of Python deps

---

### 5) Check Node.js & npm

**Ubuntu or Windows (your path):**

```console
node -v
npm -v
```
* ✅ Both return versions (e.g., `v20.x`, `10.x`)

---

### 6) Check Yarn

```console
yarn --version
```

* ✅ Returns a version (e.g., `1.22.x` or `4.x`)

---

### 7) Check uv

```console
uv --version
```

* ✅ Returns a version

---

### 8) Check VS Code Integration

**Ubuntu (Path A):**

```console
code .
```

* ✅ VS Code opens with green status bar: **WSL: Ubuntu**

* ❌ If `code` not found → in VS Code (Windows) run **Shell Command: Install 'code' command in PATH** from the Command Palette, then retry

**Windows (Path B):**

```console
code .
```

* ✅ VS Code opens in the current folder

---

If all checks pass, your system is ready for **NOMAD distro-dev**. If something fails, we’ll fix it before moving on.

---

### System Check: Quick Checklist

| Done | Tool | Command | Expected Output |
| ----- | ----- | ----- | ----- |
| ☐ | **WSL** | `wsl --status` | Shows version info, includes `WSL version: 2` |
| ☐ |  | `wsl --list --verbose` | `Ubuntu` (or another distro) listed |
| ☐ | **Docker** | `docker --version` | Docker version (e.g., `Docker version 27…`) |
| ☐ |  | `docker run hello-world` | Message “Hello from Docker!” |
| ☐ | **Git** | `git --version` | Version string (e.g., `git version 2.43.0`) |
| ☐ | **Python** | `python3 --version` *(Ubuntu)* | `3.x.x` |
| ☐ |  | `python --version` *(Windows)* | `3.x.x` (ensure Long Paths enabled in Windows) |
| ☐ | **Node.js** | `node -v` | Node version (e.g., `v20.11.1`) |
| ☐ | **npm** | `npm -v` | npm version (e.g., `10.5.2`) |
| ☐ | **Yarn** | `yarn --version` | Yarn version (e.g., `1.22.19` or `4.x`) |
| ☐ | **uv** | `uv --version` | uv version |
| ☐ | **VS Code** | `code .` | VS Code opens (WSL mode shows **WSL: Ubuntu**) |

---

## Installation Guide

This guide lists the tools required for NOMAD development on Windows in the **recommended order of installation**:

1. **WSL2**
2. **Docker Desktop**
3. **Other developer tools** (Git, Python, uv, Node.js, Yarn, VS Code)

---

### 1) Install WSL2

WSL2 (Windows Subsystem for Linux) provides a Linux environment on Windows.

#### 1.1 Enable WSL (run as Administrator)
```powershell
wsl --install
```
- Installs WSL and the default Linux distribution (Ubuntu).
- To explicitly install Ubuntu:
```powershell
wsl --install -d Ubuntu
```

If WSL is already installed, ensure you’re on WSL 2:
```powershell
wsl --set-default-version 2
wsl --update
```

#### 1.2 Verify WSL installation
```powershell
wsl --status
wsl --list --verbose
```

- Expect `WSL version: 2`

- `Ubuntu` should be listed

#### 1.3 Launch Ubuntu
Open **Windows Terminal → Ubuntu** (first launch prompts you to create a Linux username/password).

---

### 2) Install Docker Desktop

Docker is required to run NOMAD and build images.

#### 2.1 Download & install (requires Administrator)

- Download: [Docker Desktop for Windows](https://docs.docker.com/desktop/setup/install/windows-install/)

- During setup, **enable “Use WSL 2 instead of Hyper-V”**

- Restart when prompted

#### 2.2 Verify Docker (Windows side)
```powershell
docker --version
docker run hello-world
```

#### 2.3 Verify Docker (Ubuntu/WSL side)
```bash
docker --version
docker run hello-world
```

#### 2.4 Ensure WSL integration is enabled
Open **Docker Desktop → Settings → Resources → WSL Integration** and enable **Ubuntu**.

> If `docker run hello-world` fails inside Ubuntu with a daemon error, enable WSL integration for Ubuntu, then retry.

---

### 3) Install Other Developer Tools

#### 3.1 Git

**Ubuntu (Path A – recommended):**
```bash
sudo apt update
sudo apt install -y git
git --version
```

**Windows (Path B – alternative):**

- Download: [Git for Windows](https://git-scm.com/download/win)

- Choose **“Install for me only”** if you want to avoid admin rights

- Verify:
```powershell
git --version
```

---

#### 3.2 Python

**Ubuntu (Path A):**
```bash
sudo apt install -y python3 python3-pip python3-venv
python3 --version
```

**Windows (Path B):**

- Download: [Python for Windows](https://www.python.org/downloads/windows/)

- Check **Add python.exe to PATH**

- Check **Enable long path support** (or enable later as below)

- Choose **“Install for me only”** to avoid admin rights

- Verify:
```powershell
python --version
```

**Windows long path fix (important for deep dependency trees):**

- Group Policy (admin):
      - `Computer Configuration → Administrative Templates → System → Filesystem → Enable Win32 long paths = Enabled`

- Registry (admin):
```powershell
reg add HKLM\SYSTEM\CurrentControlSet\Control\FileSystem `
  /v LongPathsEnabled /t REG_DWORD /d 1 /f
```

---

#### 3.3 uv (fast Python package manager)

**Ubuntu (Path A):**
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.bashrc
uv --version
```

**Windows (Path B):**
```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
uv --version
```

---

#### 3.4 Node.js & npm

**Ubuntu (Path A) – via nvm (no root needed):**
```bash
curl -fsSL https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc
nvm install --lts
node -v
npm -v
```

**Windows (Path B) – ZIP (no admin):**

1. Download **Windows x64 ZIP** from [Node.js downloads](https://nodejs.org/en/download)

2. Extract to: `C:\Users\<YourUsername>\nodejs`

3. Add this folder to **User PATH** (Environment Variables → User → Path → Add)

4. Verify:
```powershell
node -v
npm -v
```

---

#### 3.5 Yarn

**Ubuntu (Path A):**
```bash
corepack enable
corepack prepare yarn@stable --activate
yarn --version
```

**Windows (Path B):**

- Option A (admin): install via MSI → [Yarn](https://classic.yarnpkg.com/lang/en/docs/install/#windows-stable)

- Option B (no admin, recommended): via Corepack
```powershell
corepack enable
corepack prepare yarn@stable --activate
yarn --version
```

---

#### 3.6 Visual Studio Code

**Install (User Installer; no admin required):**

- Download: [Visual Studio Code](https://code.visualstudio.com/Download)

**Extensions:**

- **Remote – WSL** (for Path A)
- **Docker**
- **Python**
- **JavaScript/TypeScript**

**Verify `code` command:**

**Ubuntu (Path A):**
```bash
code .
```

- Should open VS Code with **“WSL: Ubuntu”** in the status bar

**Windows (Path B):**
```powershell
code .
```

- Should open VS Code in the current folder

If `code` is not found:

- In VS Code (Windows): **Command Palette → “Shell Command: Install 'code' command in PATH”**, then reopen the terminal.

---

### 4) Optional: Quick PATH Verification (sanity check)

**Ubuntu (Path A):**
```bash
which git
which python3
which node
which npm
which yarn
which uv
```

**Windows (Path B):**
```powershell
where git
where python
where node
where npm
where yarn
where uv
```

All commands should resolve to valid paths (user or system locations), and the corresponding `--version` checks should print versions.

---

### ✅ Ready for NOMAD distro-dev

At this point, your system has WSL2, Docker Desktop, and all required developer tools installed and verified. Continue with the **NOMAD distro-dev** setup using the official repository documentation.

## NOMAD Dev Distribution

The **NOMAD Dev Distribution** provides a streamlined way to develop NOMAD and its plugins.
Instead of managing multiple environments, everything is set up in one editable workspace, making development and testing much easier.

---

### Benefits

- **One-step installation:** All packages are installed in editable mode, so code changes take effect immediately.
- **Centralized codebase:** Easier navigation across NOMAD and plugins.
- **Better editor support:** Improved autocompletion and refactoring.
- **Consistent tooling:** Shared linting, testing, and formatting rules.
- **Flexible plugin management:** Easily manage different plugin sets via branches.

---

### Installation Steps

1. **Fork the repository**
   Fork [nomad-distro-dev](https://github.com/FAIRmat-NFDI/nomad-distro-dev) into your own GitHub namespace.

2. **Clone your fork**
   ```bash
   git clone https://github.com/<your-username>/nomad-distro-dev.git
   cd nomad-distro-dev
   ```

3. **Install required tools**
   Make sure you have installed:

      - Docker (with `docker compose`)
      - uv (>= 0.5.14)
      - Node.js (v20) + Yarn (v1.22)

4. **Update submodules**
   This pulls in the `nomad-lab` core package.
   ```bash
   git submodule update --init --recursive
   ```

5. **Add local plugins (as submodules)**
   Put your plugins inside the `packages/` folder. Example:
   ```bash
   git submodule add https://github.com/FAIRmat-NFDI/nomad-measurements.git packages/nomad-measurements
   git submodule add https://github.com/PDI-Berlin/pdi-nomad-plugin.git packages/pdi-nomad-plugin
   ```

6. **Register plugins with uv**
   Add them to your environment so that changes are picked up:
   ```bash
   uv add packages/nomad-measurements
   uv add packages/pdi-nomad-plugin
   ```
   or edit `pyproject.toml`:
   ```toml
   [tool.uv.sources]
   nomad-measurements = { workspace = true }
   pdi-nomad-plugin   = { workspace = true }
   ```

7. **Set up the environment**
   ```bash
   uv run poe setup
   ```
   This installs dependencies and creates a `nomad.yaml` config file.

8. **Start NOMAD**
   Make sure Docker Desktop is running and the Docker daemon is active.
   Start the containers:
   ```bash
   docker compose up -d
   ```
   Then start the backend and frontend locally:
   ```bash
   uv run poe start      # backend
   uv run poe gui start  # frontend
   ```
   NOMAD will open in your browser at `http://localhost:3000/nomad-oasis/gui`.
   You will need a central NOMAD login to test properly.

---

### Day-to-Day Development

- **Update dependencies:**
  ```bash
  uv sync
  ```
- **Run tests for a plugin:**
  ```bash
  uv run --directory packages/pdi-nomad-plugin pytest
  ```
- **Lint and format code:**
  ```bash
  uv run poe lint
  ```
- **Stop and restart NOMAD:**
  ```bash
  docker compose down
  docker compose up -d
  ```

---

### Keeping Your Fork Up to Date

1. Add upstream once:
   ```bash
   git remote add upstream https://github.com/FAIRmat-NFDI/nomad-distro-dev.git
   ```

2. Fetch and merge updates:
   ```bash
   git fetch upstream
   git checkout main
   git merge upstream/main
   ```

3. Push back to your fork:
   ```bash
   git push origin main
   ```

---

### Common Issues

- **Python package build failures** → Install missing system dependencies (e.g. `clang` for `pycifrw`).
- **Long paths on Windows** → Enable [long path support](https://learn.microsoft.com/en-us/windows/win32/fileio/maximum-file-path-limitation).
- **Phonopy install fails** →
  ```bash
  uv venv -p 3.12
  uv pip install 'numpy>=1.25'
  uv pip install 'phonopy==2.11.0' --no-build-isolation
  ```

---

### ✅ Summary

With `nomad-distro-dev`, you get a **single development environment** for NOMAD and its plugins.

- Fork → clone → add plugins (`nomad-measurements`, `pdi-nomad-plugin`) → `uv run poe setup` → start NOMAD with Docker running.
- Plugins are editable immediately.
- Updates and testing are straightforward.

