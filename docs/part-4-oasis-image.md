# Part 4: Writing and Publishing a NOMAD Oasis Image (with Plugins)

This section explains how to **create your own NOMAD Oasis distribution** from the official template and how to **trigger image builds** by adding plugins to `pyproject.toml`. The result is a container image published to **GitHub Container Registry (GHCR)** that you can deploy on a server or run locally.

---

## Concept Overview

- The repository **`FAIRmat-NFDI/nomad-distro-template`** is a **template** for building custom NOMAD Oasis images.
- When you create a new repo from this template, **GitHub Actions** will automatically build and publish:
    - an **app** image (Oasis),
    - and a **jupyter** image (for NORTH / JupyterHub).
- You **customize** the image by listing your **plugins** under `[project.optional-dependencies].plugins` in `pyproject.toml`.
- **Every push to `main`** triggers the CI workflow that **builds and publishes** new images to **GHCR**.

**Visibility & Authentication Notes**

- Depending on your org/repo settings, the produced image may be **private** (requires a **Personal Access Token**, PAT, to pull) or **public** (recommended for easy deployment).
- To keep the image private, configure a PAT with the scopes required for GHCR.
- To make it public, ensure your organization allows public packages and change the package visibility after the first successful build.

**Discoverability Tip**

- Consider adding the repo topic **`nomad-distribution`** (Repo → *About* → ⚙️ → *Topics*) so others can find your distribution.

---

## Step-by-Step: Create Your Distribution Repository

1. **Generate a new repo from the template**

    - Open: `https://github.com/FAIRmat-NFDI/nomad-distro-template`
    - Click **Use this template** (top-right), or use the “New from template” link:
      `https://github.com/new?template_name=nomad-distro-template&template_owner=FAIRmat-NFDI`
    - Choose a name (e.g., `my-nomad-oasis`) and create the repo.

2. **Wait for the template initialization workflow**

      - On first creation, a GitHub Action (Template Repository Initialization) runs automatically.
      - If the “initialization warning” in the README does not disappear after a few minutes:

          - Go to **Actions** → select **Template Repository Initialization** → **Run workflow** to trigger it manually.

      - Once finished, refresh the page (the warning disappears).

3. **Clone your new repository or run it directly in GitHub Codespaces**
   ```bash
   git clone https://github.com/<YOUR_ORG_OR_USER>/<YOUR_REPO>.git
   cd <YOUR_REPO>
   ```
GitHub Codespaces:

4. **(Optional) Make your package public**

      - After your first successful build, open your repo’s **Packages** section and change visibility to **Public** if desired.

---

## Repository Layout (What You’ll Touch Most)

- **`pyproject.toml`** → declare which **plugins** to include in the built image.
- **`.github/workflows/docker-publish.yml`** → CI that builds and pushes images.
- **`docker-compose.yaml`** and **`configs/`** → example deployment config (local/server).

You **add plugins** to `pyproject.toml`. After you **merge to `main`**, CI builds and publishes fresh images.

---

## Add a Plugin to `pyproject.toml`

Open `pyproject.toml` and add your plugin(s) under the `plugins` extra:

### A) Plugin from PyPI
```toml
[project.optional-dependencies]
plugins = [
  "nomad-material-processing>=1.0.0",
]
```

### B) Plugin from Git (pinned to a commit)
```toml
[project.optional-dependencies]
plugins = [
  "nomad-measurements @ git+https://github.com/FAIRmat-NFDI/nomad-measurements.git@71b7e8c9bb376ce9e8610aba9a20be0b5bce6775",
]
```

### C) Plugin from Git (pinned to a tag)
```toml
[project.optional-dependencies]
plugins = [
  "nomad-measurements @ git+https://github.com/FAIRmat-NFDI/nomad-measurements.git@v0.0.4",
]
```

### D) Plugin in a subdirectory of a repo
```toml
[project.optional-dependencies]
plugins = [
  "ikz_pld_plugin @ git+https://github.com/FAIRmat-NFDI/AreaA-data_modeling_and_schemas.git@30fc90843428d1b36a1d222874803abae8b1cb42#subdirectory=PVD/PLD/jeremy_ikz/ikz_pld_plugin",
]
```

**Tips**

- Prefer **tags** or **commits** for reproducible builds.
- You can list **multiple** plugins in the same array.
- Keep versions **pinned** (tags/ranges) to avoid surprises.

In our case, let's add the `pdi-nomad-plugin` to the Oasis image and remove all other plugins.
Add the following line to `[project.optional-dependencies]` and remove the existing plugins:
```

```

---

## Commit, Push, and Trigger the Image Build

1. **Create a feature branch, edit, commit**
   ```bash
   git checkout -b add-my-plugin
   # edit pyproject.toml as above
   git add pyproject.toml
   git commit -m "Add plugin(s) to distribution image"
   ```

2. **Push the branch and open a Pull Request**
   ```bash
   git push origin add-my-plugin
   ```

      - On GitHub, open a **PR** into `main`.
      - Request review if branch protection rules are in place.

3. **Merge to `main` to trigger CI**

      - Merging the PR starts the **docker-publish** workflow.
      - Watch progress under **Actions** → **docker-publish**.

---

## Where to Find the Built Images

After a successful run, images are pushed to **GHCR**:

- App: `ghcr.io/<OWNER>/<REPO>:main`
- Jupyter: `ghcr.io/<OWNER>/<REPO>/jupyter:main`

**Pulling private images requires authentication** with a PAT.

### GHCR Login (if your package is private)
Create a **Personal Access Token (classic)** with scopes:

- `read:packages` (pull)
- `write:packages` (push; not needed for deploy-only machines)

Login:
```bash
echo "<YOUR_PAT>" | docker login ghcr.io -u <YOUR_GITHUB_USERNAME> --password-stdin
```

---

## Quick Local Test: Pull & Run

From the distribution repo folder:
```bash
docker compose pull
docker compose up -d
```

Health check:
```bash
# HTTP
curl localhost/nomad-oasis/alive

# HTTPS (use --insecure only for self-signed certs)
curl --insecure https://localhost/nomad-oasis/alive
```

Open the UI: `http://localhost/nomad-oasis`

> **Linux-only note:** You may need to fix volume ownership once:
> ```bash
> sudo chown -R 1000 .volumes
> ```

---

## Updating the Image After More Plugin Changes

When you add or update plugins in `pyproject.toml` and **merge to `main`**, CI will build fresh images.

To update your local deployment:
```bash
docker compose down
docker compose pull
docker compose up -d
```

Free disk space if needed:
```bash
docker image prune -a
```

---

## Notes on the Jupyter (NORTH) Image

- CI also builds a **Jupyter image**: `ghcr.io/<OWNER>/<REPO>/jupyter:main`
- Pre-pull to avoid startup timeouts:
  ```bash
  docker pull ghcr.io/<OWNER>/<REPO>/jupyter:main
  ```
- Add global Jupyter packages by listing them under the `jupyter` extra in `pyproject.toml`:
  ```toml
  [project.optional-dependencies]
  jupyter = [
    "voila",
    "ipyaggrid",
    "ipysheet",
    "ipydatagrid",
    "jupyter-flex",
  ]
  ```

---
## Updating the PDI NOMAD Oasis Image

The NOMAD Oasis image which runs on the PDI server is managed here:
https://github.com/PDI-Berlin/PDI-NOMAD-Oasis-image

You should organize yourself regarding **responsibility** (who maintains the repo and images) and **accessibility** (who has permissions to update, push changes, and trigger new builds).

---

### Suggested Practices

- **Team access**

    - Ensure that at least 2–3 team members have *Admin* or *Maintainer* rights on the repository to avoid single points of failure.
    - Use a GitHub Team (inside the PDI organization) to manage access rather than assigning rights individually.

- **Branch protection**

    - Protect the `main` branch. Require pull requests and at least one reviewer before merging.
    - This avoids accidental pushes that would immediately trigger a new image build.

- **Versioning**

    - Tag releases (e.g., `v0.1.0`, `v0.2.0`) before merging into `main`.
    - Tags can be used for stable deployments and make it easy to roll back to a known working version.
    - Consider using GitHub Releases to document what changed in each update.

- **Documentation**

    - Keep the README updated with clear instructions on how to:

        - Add plugins (`pyproject.toml`)
        - Trigger builds (merge to `main`)
        - Pull and deploy images on the server

- **CI Monitoring**

    - Regularly check the GitHub Actions build logs after merging changes.
    - If builds fail, fixes should be prioritized because the Oasis server cannot update otherwise.

- **Security**

    - If the image is private, ensure that the required PAT or credentials for pulling are stored securely on the server (e.g., in a Docker login config file).
    - Rotate tokens if people leave the team.

- **Update procedure**

    - When updating the image on the server, follow the safe sequence:
      ```bash
      docker compose down
      docker compose pull
      docker compose up -d
      ```
    - Always test after updating:
      ```bash
      curl localhost/nomad-oasis/alive
      ```

---
### Useful Docker Commands

When working with NOMAD Oasis images, it’s helpful to know a few basic Docker commands for monitoring and troubleshooting:

- **List running containers**  
  ```bash
  docker ps
  ```
  Shows all currently running containers with their names, IDs, ports, and status.

- **Check logs of a container**  
  ```bash
  docker logs <container_name>
  ```
  Displays the output of a running container. Use this to check for errors or debug startup issues.  
  Add `-f` to follow logs in real time:
  ```bash
  docker logs -f <container_name>
  ```

- **List all containers (including stopped)**  
  ```bash
  docker ps -a
  ```

- **Stop / start / restart a container**  
  ```bash
  docker stop <container_name>
  docker start <container_name>
  docker restart <container_name>
  ```

- **Remove unused images and containers**  
  ```bash
  docker system prune
  ```
  Cleans up stopped containers, dangling images, and unused networks.  
  Add `-a` to remove all unused images (be careful, it frees space but requires re-pulling images later).

- **Check disk usage**  
  ```bash
  docker system df
  ```
  Summarizes how much disk space images, containers, and volumes are using.

- **Open a shell inside a container**  
  ```bash
  docker exec -it <container_name> /bin/bash
  ```
  Gives you direct access to the container’s shell, useful for advanced debugging.

---

**Tip:** Most NOMAD Oasis setups use `docker compose`, which manages multiple containers at once.  
- To check logs of all services together:  
  ```bash
  docker compose logs -f
  ```
- To restart everything:  
  ```bash
  docker compose down
  docker compose up -d
  ```

---

### Docker Command Cheat Sheet

| Command | Purpose |
|---------|---------|
| `docker ps` | List running containers |
| `docker ps -a` | List all containers (including stopped ones) |
| `docker logs <container_name>` | Show logs of a container |
| `docker logs -f <container_name>` | Follow container logs in real time |
| `docker stop <container_name>` | Stop a running container |
| `docker start <container_name>` | Start a stopped container |
| `docker restart <container_name>` | Restart a container |
| `docker exec -it <container_name> /bin/bash` | Open an interactive shell inside a container |
| `docker system df` | Show Docker disk usage (images, containers, volumes) |
| `docker system prune` | Remove unused containers, networks, and dangling images |
| `docker system prune -a` | Remove **all** unused images (frees more space, but requires re-pull) |
| `docker compose logs -f` | Show and follow logs of all services managed by Docker Compose |
| `docker compose down` | Stop and remove all containers defined in `docker-compose.yml` |
| `docker compose up -d` | Start all containers in detached mode |

**Tip:** Use `docker ps` to find the correct `<container_name>` before running log or exec commands.

---

### ✅ Summary

- Define **who is responsible** for maintaining the repo and images.
- Use **branch protection, tags, and reviews** to keep updates reliable.
- Keep the **server update procedure** documented and repeatable.
- Ensure **multiple team members** have access to avoid bottlenecks.

---

## Using Tags to Version the PDI NOMAD Oasis Image

Tags in Git and GitHub allow you to **mark a specific commit** with a version label (e.g. `v0.1.0`).
This is useful for NOMAD Oasis because each tag corresponds to a reproducible Docker image.
You can always deploy, roll back, or reference a tagged version.

---

### Why Use Tags?

- **Reproducibility** → the exact state of the code at the moment of tagging can always be retrieved.
- **Stability** → the server can run a known-good version instead of always tracking the moving `main` branch.
- **Traceability** → tags (together with GitHub Releases) provide a history of what changed.

---

### Step-by-Step: Creating and Using Tags

#### 1. Make sure your branch is up to date
```bash
git checkout main
git pull origin main
```

#### 2. Create a new tag
```bash
git tag v0.1.0
```

- Replace `v0.1.0` with your desired version.
- Tag names usually start with `v` (semantic versioning: `vMAJOR.MINOR.PATCH`).

#### 3. Push the tag to GitHub
```bash
git push origin v0.1.0
```

- This makes the tag visible in GitHub and triggers the **docker-publish workflow**.
- The CI builds new images and publishes them with the tag.

---

### Resulting Docker Images

If your repo is `PDI-Berlin/PDI-NOMAD-Oasis-image`, then:

- **App image:**
  `ghcr.io/pdi-berlin/pdi-nomad-oasis-image:v0.1.0`

- **Jupyter image:**
  `ghcr.io/pdi-berlin/pdi-nomad-oasis-image/jupyter:v0.1.0`

(Instead of `:main`, you now have versioned tags like `:v0.1.0`.)

---

### 4. Deploy a Tagged Image on the Server

In your `docker-compose.yaml`, update the `image:` entries from `:main` to your tag, for example:

```yaml
services:
  app:
    image: ghcr.io/pdi-berlin/pdi-nomad-oasis-image:v0.1.0
  jupyter:
    image: ghcr.io/pdi-berlin/pdi-nomad-oasis-image/jupyter:v0.1.0
```

Then update the deployment:

```bash
docker compose down
docker compose pull
docker compose up -d
```

---

### 5. (Optional) Create a GitHub Release

To add documentation to your tag:

1. Go to your GitHub repo → **Releases** → **Draft a new release**
2. Select your tag (`v0.1.0`)
3. Write release notes (what changed, new plugins, fixes)
4. Publish the release

This makes it easier for others to know what each tag contains.

---

### ✅ Best Practices

- Use **semantic versioning** (`vMAJOR.MINOR.PATCH`)
    - `MAJOR` → incompatible changes
    - `MINOR` → new features/plugins (backwards-compatible)
    - `PATCH` → bug fixes
- Always **tag before deploying to production**.
- Keep the **server pinned to a tag**, not `main`.
- Document what each tag means in **Releases**.

---

With this workflow, PDI can **safely upgrade** Oasis images and **roll back** to the previous version if something breaks.


---

## Troubleshooting

- **Template init message persists**
  *Actions* → run **Template Repository Initialization** manually.

- **Can’t pull image / image not visible**
  Ensure the workflow succeeded; check package **visibility** (public/private).
  If private, **`docker login ghcr.io`** with a PAT.

- **Plugin installation fails in CI**
  Validate `pyproject.toml` syntax; pin to valid tags/commits; ensure plugin repo is reachable/public (or CI has access).

- **HTTPS / TLS**
  For production, switch to HTTPS by providing a valid certificate and updating `docker-compose.yml` to use the HTTPS nginx config. For local testing you can use self-signed certs (browsers won’t trust them).

---

## Quick Reference: Add Plugin → Build → Use

1. **Edit `pyproject.toml`**
   Add plugin lines under `[project.optional-dependencies].plugins`.

2. **Commit & push via a branch → PR → merge to `main`**
   ```bash
   git checkout -b add-my-plugin
   git add pyproject.toml
   git commit -m "Add plugin(s)"
   git push origin add-my-plugin
   # open PR on GitHub and merge
   ```

3. **CI builds images to GHCR**

      - App: `ghcr.io/<OWNER>/<REPO>:main`
      - Jupyter: `ghcr.io/<OWNER>/<REPO>/jupyter:main`

4. **Pull & run**
   ```bash
   docker compose pull
   docker compose up -d
   ```
