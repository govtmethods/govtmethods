# A step-by-step onboarding for collaborators 
(Quarto + GitHub + CI deployment)

## Hi there 👋

Read this first (what this guide is doing)

This guide is written so a new co-editor (e.g., **Beyonce**) can go from “I’ve never touched this repo” to “I can safely make changes and get them deployed” with **institution-level reproducibility and review discipline**.

A few notes about tone and assumptions:

- When this guide says **“run”** a command, it means: open **Terminal** and paste the command.
- Most commands are shown in **bash** blocks. On macOS, Terminal defaults to **zsh**, but the commands are the same.
- I’ll explicitly mark steps that are **macOS-specific** vs cross-platform.
- The deploy repo is intentionally **hands-off**: you edit source repos, and the website updates through **GitHub Actions**.

---

## The big picture (how the system works)

### Repos (there are two kinds)

#### 1) Source repos (where humans edit content)
These contain Quarto sources: `.qmd`, `styles.scss`, `_quarto.yml`, images, etc.

- `gm-landing` — landing site (index/about/ask)
- `gm-mathcamp` — Math Camp site/book
- `gm-methodsbrownbag` — Methods Brown Bag site/book
- `gm-researchnote` — (future) research notes site/book

#### 2) Deploy repo (CI-managed output only)
This contains rendered HTML only (built artifacts), and is what GitHub Pages serves publicly.

- `govtmethods.github.io` — deploy target (public site)

**Rule:** almost all editing happens in the **source repos**, not the deploy repo.

### URLs (what maps to what)

- `https://govtmethods.github.io/`  
  Published from: `gm-landing`

- `https://govtmethods.github.io/mathcamp/`  
  Published from: `gm-mathcamp`

- `https://govtmethods.github.io/methodsbrownbag/`  
  Published from: `gm-methodsbrownbag`

- `https://govtmethods.github.io/researchnote/`  
  Published from: `gm-researchnote` (future)

### What happens when you merge changes

When you merges a Pull Request (PR) into `main` in a source repo:

1. GitHub Actions runs in that source repo (it’s basically an automated build machine).
2. It installs Quarto (and R if needed).
3. It renders the Quarto project into HTML.
4. It checks out `govtmethods.github.io` in the background.
5. It copies the rendered HTML into the correct folder in `govtmethods.github.io`.
6. It commits and pushes the deploy output.
7. GitHub Pages serves the updated site.

---

## Collaboration norms (two modes)

You can collaborate in two ways. We prefer **Option A** for institutional discipline, but Option B is included as an explicit “if we need it” alternative.

### Option A — PR-based workflow (recommended)
- Work on a branch
- Open a Pull Request
- Get at least 1 approval
- Merge triggers deployment

This gives you:
- review discipline
- a clean record of changes
- fewer “oops we broke the homepage” moments

### Option B — direct-push workflow (optional)
- Push directly to `main`
- Deployment happens immediately

Use only if the team explicitly agrees. (It’s fast, but you lose the safety rails.)

---

## One-time onboarding (do these once)

### Step 0 — Accept collaborator invitations (GitHub UI)

You’ll be added as a collaborator from the **`govtmethods`** account.

You must accept the invitation (GitHub email or notifications).  
If you don’t accept, pushes will fail with confusing messages like:

- “Repository not found”
- “Write access not granted”

---

## Step 1 — Install the tools (Quarto + Git + R)

### Step 1A — Git (cross-platform)
Check:

```bash
git --version
```

If Git isn’t installed:
- **macOS:** installing Xcode Command Line Tools usually works:
  ```bash
  xcode-select --install
  ```
- Or install Git via Homebrew (macOS):
  ```bash
  brew install git
  ```

### Step 1B — Quarto (cross-platform)
Check:

```bash
quarto --version
```

If Quarto isn’t installed:
- Install the Quarto installer for your OS (recommended), or
- **macOS (optional):**
  ```bash
  brew install --cask quarto
  ```

### Step 1C — R (needed for Mathcamp / BrownBag; optional for landing)
Check:

```bash
R --version
```

If missing, install from CRAN for your OS.

### Step 1D — RStudio (optional but nice)
RStudio is optional, but it can make Quarto editing smoother.

---

## Step 2 — Set up SSH (recommended, and what we use)

We use **SSH** for pushing to GitHub because it’s reliable and avoids token restrictions (especially around workflow files).

### Step 2A — Check if you already have an SSH key (cross-platform)

```bash
ls -al ~/.ssh
```

Look for:
- `id_ed25519`
- `id_ed25519.pub`

If both exist, you can skip to **Step 2D**.

### Step 2B — Create an SSH key (cross-platform)

```bash
ssh-keygen -t ed25519 -C "YOUR_EMAIL"
```

When prompted:
- “Enter file in which to save the key …” → press **Enter** (accept default)
- “Enter passphrase …” → recommended to set one (you can also leave blank)

### Step 2C — Start the SSH agent and add your key

#### macOS-specific (recommended)
macOS can store the key in Keychain so you don’t re-enter passphrases constantly.

```bash
eval "$(ssh-agent -s)"
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
```

#### Linux (typical)
```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

#### Windows
Use Git Bash or Windows Terminal with OpenSSH. The steps are similar but may require enabling the OpenSSH agent.

### Step 2D — Add your public key to GitHub (GitHub UI)

Copy the public key:

#### macOS (copy to clipboard)
```bash
pbcopy < ~/.ssh/id_ed25519.pub
```

#### Linux (print and copy)
```bash
cat ~/.ssh/id_ed25519.pub
```

Then on GitHub (your personal account):
- Settings → **SSH and GPG keys**
- **New SSH key**
- Paste your key
- Save

### Step 2E — Test SSH authentication (cross-platform)

```bash
ssh -T git@github.com
```

Expected success message:
> “Hi USERNAME! You've successfully authenticated, but GitHub does not provide shell access.”

That means SSH is working.

---

## Step 3 — Create a local workspace folder (recommended structure)

This is not a git repo. It’s just a folder that contains multiple repos side-by-side.

```bash
mkdir -p ~/Documents/GitHub/govtmethods-workspace
cd ~/Documents/GitHub/govtmethods-workspace
```

After cloning, you’ll have:

- `gm-landing/`
- `gm-mathcamp/`
- `gm-methodsbrownbag/`
- (optional) `govtmethods.github.io/` (deploy repo; usually not needed locally)

---

## Step 4 — Clone the repos (source repos only)

From inside the workspace folder:

```bash
git clone git@github.com:govtmethods/gm-landing.git
git clone git@github.com:govtmethods/gm-mathcamp.git
git clone git@github.com:govtmethods/gm-methodsbrownbag.git
```

You typically **do not need** to clone the deploy repo:

- `govtmethods.github.io`

Because CI manages it. (Maintainers might clone it occasionally to inspect output, but contributors usually don’t.)

---

## Step 5 — “Sanity check” that you can pull and push

Pick one repo (landing is simplest):

```bash
cd gm-landing
git status
git pull
```

Now make a harmless local change to verify your workflow (example: add a blank line to README if it exists), then:

```bash
git checkout -b test/ssh-works
git add -A
git commit -m "Test: verify SSH push works"
git push -u origin HEAD
```

If that push succeeds, you’re good.

---

# Day-to-day editing workflow

## Option A (recommended) — PR-based workflow

### Step A1 — Update your local `main`

Always start by syncing with upstream:

```bash
git checkout main
git pull
```

### Step A2 — Create a new branch

Branch naming convention examples:
- `feature/add-spring-2026-schedule`
- `fix/broken-link`
- `content/update-about-text`
- `chore/rename-files`

Create your branch:

```bash
git checkout -b feature/short-description
```

### Step A3 — Make edits

Edit `.qmd`, `.scss`, `_quarto.yml`, etc.

Common file locations:
- **Landing:** `index.qmd`, `about.qmd`, `ask.qmd`, `styles.scss`, `_includes/`
- **Mathcamp:** `index.qmd`, `chapters/*.qmd`, `styles.scss`
- **Brown bag:** `index.qmd`, `sessions/*.qmd`, `styles.scss`

### Step A4 — Preview locally

#### Websites
```bash
quarto preview
```

#### Books
```bash
quarto preview
```

If you prefer a one-time build:
```bash
quarto render
```

### Step A5 — What NOT to commit

Do **not** commit build artifacts or caches:

- `_site/` (website output)
- `_book/` (book output)
- `.quarto/`
- `.Rproj.user/`
- `.DS_Store`

These should be in `.gitignore` already. If you see them in `git status`, stop and fix before committing.

### Step A6 — Commit changes

```bash
git status
git add -A
git commit -m "Describe what changed (clear and specific)"
```

### Step A7 — Push your branch

```bash
git push -u origin HEAD
```

### Step A8 — Open a Pull Request (GitHub UI)

On GitHub:
- Open a PR from your branch into `main`
- Explain what changed and why
- Request review (Maintainer/site administrator)

### Step A9 — Merge and deploy

Once approved, merge the PR.  
Merging triggers CI deployment automatically.

---

## Option B (optional) — direct push to `main`

Only do this if branch protections are disabled and the team explicitly agrees.

```bash
git checkout main
git pull

# make edits

git add -A
git commit -m "Update content"
git push origin main
```

Deployment will happen immediately.

---

# Reproducibility: R environments with `renv`

For `gm-mathcamp` and `gm-methodsbrownbag`, we typically use **renv**.

### What this means
- The repo includes `renv.lock`
- CI restores packages exactly as specified
- Builds are more stable across machines and over time

### If you add/update packages
In R (inside that project), run:

```r
renv::snapshot()
```

Then commit the lockfile:

```bash
git add renv.lock
git commit -m "Update renv lockfile"
git push
```

### If CI says “This project does not contain a lockfile.”
That means a workflow tried to restore renv but `renv.lock` is missing.

Fix options:
- commit `renv.lock` (preferred if the project uses R), or
- remove the renv restore step (common for landing-only sites)

---

# Deployment details (what contributors should know)

### Deploy token (you don’t handle this locally)
Deployments use a GitHub Actions secret stored in each source repo:

- `PAGES_DEPLOY_PAT`

This token is owned by the `govtmethods` admin account and allows CI to push into `govtmethods.github.io`.

Contributors do **not** need this token on their laptops.

### Where deployments go
- `gm-landing` → deploy repo root (`/`)
- `gm-mathcamp` → `/mathcamp/`
- `gm-methodsbrownbag` → `/methodsbrownbag/`

---

# Branch protection (maintainer policy; recommended)

For each source repo, maintainers should enable branch protection for `main`:

GitHub repo → Settings → Branches → Add rule for `main`:

Recommended settings:
- Require PR before merging
- Require at least 1 approval
- Require branches to be up to date (recommended)
- Disable force pushes
- Disable branch deletion

This enforces review discipline and prevents accidental deploy breaks.

---

# Troubleshooting (most common problems)

## “Repository not found”
Usually means:
- you didn’t accept the collaborator invite yet, or
- the remote points to the wrong URL

Check:
```bash
git remote -v
```

## “Permission denied (publickey)”
SSH key not loaded, or not added to GitHub.

Fix (macOS):
```bash
eval "$(ssh-agent -s)"
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
ssh -T git@github.com
```

## CI fails with `_site/` missing
Your project might build to `_book/` (books).  
The workflow needs to rsync the correct folder.

Local check:
```bash
quarto render
ls
```

## CI fails with “Write access to repository not granted” (403)
Usually:
- deploy token missing/incorrect, or
- workflow pushing to the wrong remote

Fix:
- confirm `PAGES_DEPLOY_PAT` exists in the repo secrets
- ensure workflow sets deploy remote explicitly before pushing

---

# Practical quick-reference

### Update local main
```bash
git checkout main
git pull
```

### Create a branch
```bash
git checkout -b feature/thing
```

### Commit and push branch
```bash
git add -A
git commit -m "Message"
git push -u origin HEAD
```

### Preview
```bash
quarto preview
```

### Test GitHub SSH
```bash
ssh -T git@github.com
```

---

## Who to contact

If something breaks, send:
- which repo you were in
- the command you ran
- the full error output
- (if CI) the Actions run link + failing step log

Maintainers:
- Government Methods Leads
<!--
**govtmethods/govtmethods** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.

Here are some ideas to get you started:

- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
-->
