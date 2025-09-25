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
