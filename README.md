# A step-by-step onboarding for collaborators 
(Quarto + GitHub + CI deployment)

## Hi there 👋

This guide is written so a new co-editor (e.g., **Beyonce**) can go from “I’ve never touched this repo” to “I can safely make changes and get them deployed” with **institution-level reproducibility and review discipline**.

# Read this first

This guide takes a new collaborator — someone who may have never used Git professionally, set up SSH, or worked in a multi-repo publishing system — from zero to confidently contributing content that deploys automatically to the web.

**Assumptions:**

- You have a Mac (most steps are cross-platform; macOS-specific steps are labeled).
- You have a GitHub account. If not, create one at [github.com](https://github.com) before continuing.
- You do not need to know how to code. You will write in `.qmd` files, which are plain text with some formatting marks.^[`.qmd` stands for "Quarto Markdown." Markdown is a lightweight way of formatting text using plain characters — `**bold**` becomes **bold**, `# Heading` becomes a heading. Quarto adds the ability to run R or Python code inside the document and render it to HTML, PDF, or other formats.]

**How to read this guide:**

- When this guide says **"run"** a command, it means: open **Terminal** (`Cmd + Space`, type "Terminal") and paste the command exactly.^[On macOS, the default shell is **zsh** (since macOS Catalina). You may also see **bash** mentioned — both understand the same commands used in this guide. The `$` symbol at the start of a command just indicates "this is a terminal command"; do not type the `$`.]
- Commands appear in grey blocks like this: `git status`. Run them as written.
- Lines starting with `#` inside a command block are comments — they explain what the next line does. Do not worry about them; they do not do anything when run.

\newpage

# The contributor rule

**As a collaborator, you touch exactly two things:**

1. `.qmd` files — your chapter content
2. The `chapters:` list in `_quarto.yml` — to register a new chapter you wrote

Everything else — `styles.scss`, `_includes/`, `_filters/`, `.github/workflows/`, and all other settings in `_quarto.yml` — is maintained by the site editors (Parushya and Sara). **Do not edit those files.** If something looks wrong with layout, colors, or controls, report it rather than fixing it yourself.

This constraint exists because the shared design system is carefully tuned. A well-intentioned edit to `styles.scss` in one chapter can break the appearance across the entire site for all readers.

\newpage

# The big picture: how this system works

Before installing anything, it helps to understand the overall architecture. This section explains the key concepts: what Git is, how the repos are organized, and what happens when you push a change.

## What Git is (briefly)

**Git** is a version control system — a tool that tracks changes to files over time and lets multiple people work on the same project without overwriting each other's work.^[Think of Git as a very detailed "track changes" system, except instead of Word's inline markup it records snapshots of the entire project at each "save point" (called a **commit**). Every commit has a message describing what changed and who changed it, and you can always go back to any previous snapshot.]

**GitHub** is a website that hosts Git repositories online, adds collaboration features (Pull Requests, Issues, Actions), and in our case serves as the deployment pipeline for the public website.

## The two kinds of repositories

A **repository** (or "repo") is a folder tracked by Git. We have two kinds:

### Source repos — where you write content

These contain the raw Quarto source files: `.qmd` chapters, images, configuration.

| Repo name | What it is | Public URL |
|---|---|---|
| `govtmethods.github.io` | Landing page | `https://govtmethods.github.io/` |
| `mathcamp` | Math Camp book | `.../mathcamp/` |
| `methodsbrownbag` | Methods Brown Bag book | `.../methodsbrownbag/` |

### Deploy repo — CI-managed output only

The repo `govtmethods.github.io` serves a second role: it also holds the **rendered HTML output** that GitHub Pages serves to the public. You never edit this repo directly.^[This dual-role naming is a GitHub Pages convention: the repo named `<org>.github.io` is automatically served as the root of the public site. Our CI pipeline renders each source repo and pushes the output into the correct subfolder of this deploy repo. Contributors never clone or edit the deploy repo.]

**Rule:** almost all editing happens in the **source repos**, never in the deploy output.

## What happens when you push a change

When a Pull Request is merged into `main` in any source repo:

1. **GitHub Actions** detects the merge and starts an automated build job.^[GitHub Actions is GitHub's built-in CI/CD (Continuous Integration / Continuous Deployment) system. It runs code on GitHub's servers in response to events like pushes and pull requests. You can think of it as a robot that automatically builds and publishes your changes.]
2. The build machine installs Quarto, R, and all required R packages.
3. It runs `quarto render`, which compiles every `.qmd` file into HTML and PDF.
4. It copies the rendered output into the correct subfolder of the deploy repo.
5. GitHub Pages detects the new output and updates the live website.

**You never render for deployment yourself.** Your job ends at `git push`. The robot takes it from there.

\newpage

# One-time onboarding

Do these steps once when you first join the project.

## Step 0 — Accept the collaborator invitation

Parushya will send you a collaborator invitation from the `govtmethods` GitHub organization to your personal GitHub account. You must accept it before you can push to any `govtmethods` repo.

Check your GitHub notifications or the email GitHub sends to your registered address. Click "Accept invitation."

If you skip this step, pushes will fail with errors like:

```
remote: Repository not found.
fatal: Could not read from remote repository.
```

or

```
ERROR: Permission to govtmethods/methodsbrownbag.git denied to yourusername.
```

Both mean the same thing: GitHub does not recognize your account as having write access.

## Step 1 — Install the tools

### Git

Check whether Git is already installed:

```bash
git --version
```

A successful response looks like: `git version 2.45.0`. If you get "command not found":

```bash
# macOS: install Xcode Command Line Tools (includes Git)
xcode-select --install
```

Alternatively, install Git via [Homebrew](https://brew.sh):^[Homebrew is a package manager for macOS — a tool that installs and manages command-line software. If you do not have Homebrew, install it first by following the one-line installer at `brew.sh`. Many developers use it to manage Git, Python, R, and other tools.]

```bash
brew install git
```

### Quarto

Quarto is the publishing system that converts `.qmd` files to HTML and PDF. Check:

```bash
quarto --version
```

If missing, download the installer for your operating system from [quarto.org/docs/get-started](https://quarto.org/docs/get-started/), or on macOS:

```bash
brew install --cask quarto
```

### R

R is required for the book projects (`mathcamp`, `methodsbrownbag`), which contain R code chunks.^[The landing site (`govtmethods.github.io`) does not use R, so contributors who only work on the landing page do not need R installed.] Check:

```bash
R --version
```

If missing, download from [cran.r-project.org](https://cran.r-project.org) for your OS. The projects require **R 4.5.2** — install that version or newer.

### RStudio (optional)

RStudio is an editor tailored for R and Quarto. It is not required — any text editor (VS Code, Cursor, even TextEdit) can open `.qmd` files — but RStudio provides a built-in Quarto preview panel that many contributors find helpful.

## Step 2 — Set up SSH

### Why SSH, and what it is

**SSH** (Secure Shell) is a protocol for authenticating with remote servers. For our purposes, it is the method by which your laptop proves its identity to GitHub when you push changes.^[The alternative is HTTPS with a Personal Access Token (PAT). SSH is preferred here because: (1) once configured it requires no password at each push, (2) it cannot accidentally expire the way tokens do, and (3) GitHub restricts HTTPS tokens from modifying workflow files (`.github/workflows/`), which maintainers need to update occasionally.]

SSH uses a **key pair**: a private key (stays on your laptop, never shared) and a public key (uploaded to GitHub). When you push, your laptop signs the request with the private key; GitHub verifies it with the public key. If they match, you're in.

### How access to `govtmethods` repos works

This is a common point of confusion: **there is no separate "govtmethods SSH key."** GitHub authenticates you as your personal account. What gives you write access to `govtmethods` repos is the collaborator invitation (Step 0) linking your personal account to those repos.

The flow is:

```
Your SSH private key
      ↓  (proves identity to GitHub)
Your personal GitHub account (e.g., github.com/yourname)
      ↓  (has collaborator access via invitation)
govtmethods/methodsbrownbag  ←  push allowed
```

You add the public key to **your personal** GitHub account settings, not to the `govtmethods` organization. The org doesn't need its own key.

### Step 2A — Check for an existing SSH key

```bash
ls -al ~/.ssh
```

Look for a pair of files:

- `id_ed25519` — your private key (never share this)
- `id_ed25519.pub` — your public key (safe to share; this goes to GitHub)

If both exist, skip to **Step 2D**.

You may also see `id_rsa` / `id_rsa.pub` — an older key type. These still work but Ed25519 keys (`ed25519`) are smaller and more secure. If you only have RSA keys, you can either use them or generate a fresh Ed25519 key alongside them.^[Having multiple key pairs is fine. You can tell SSH which key to use for which host via `~/.ssh/config`. See Step 2F for the multi-account case.]

### Step 2B — Generate a new SSH key

```bash
ssh-keygen -t ed25519 -C "your.github.email@example.com"
```

The `-C` flag adds a comment label to the key — use the email address associated with your GitHub account so you can identify the key later.

You will be prompted twice:

**"Enter file in which to save the key"** → Press **Enter** to accept the default location (`~/.ssh/id_ed25519`). Only change this if you are managing multiple keys.

**"Enter passphrase"** → A passphrase encrypts the private key file, so even if someone gets access to your laptop they cannot use your key without knowing the passphrase. We recommend setting one. You will not need to type it at every push — the SSH agent (next step) handles that.

### Step 2C — Load the key into the SSH agent

The **SSH agent** is a background process that holds your decrypted key in memory so you do not need to re-enter your passphrase at every push.

#### macOS

macOS can persist the passphrase in the system Keychain:

```bash
eval "$(ssh-agent -s)"
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
```

To make this permanent (so it survives reboots), add the following to `~/.ssh/config` (create the file if it doesn't exist):

```
Host github.com
    HostName github.com
    User git
    AddKeysToAgent yes
    UseKeychain yes
    IdentityFile ~/.ssh/id_ed25519
```

After this, macOS will automatically load the key on login and store the passphrase in Keychain.

#### Linux

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

To make it permanent, add `ssh-add ~/.ssh/id_ed25519` to your `~/.bashrc` or `~/.zshrc`.

#### Windows

Use Git Bash or Windows Terminal with OpenSSH enabled. Enable the OpenSSH Authentication Agent service in Windows Services, then:

```bash
ssh-add ~/.ssh/id_ed25519
```

### Step 2D — Add your public key to GitHub

First, copy the public key to your clipboard:

```bash
# macOS
pbcopy < ~/.ssh/id_ed25519.pub

# Linux (prints to terminal — copy manually)
cat ~/.ssh/id_ed25519.pub
```

Then on **your personal GitHub account**:

1. Click your profile photo (top-right corner) → **Settings**
2. Left sidebar → **SSH and GPG keys**
3. Click **New SSH key**
4. In the **Title** field, give the key a descriptive name so you know which machine it belongs to — e.g., `MacBook Pro M3 2024` or `Georgetown laptop`
5. Paste the public key into the **Key** field
6. Click **Add SSH key**

GitHub will confirm with an email notification.

### Step 2E — Test the connection

```bash
ssh -T git@github.com
```

A successful response:

```
Hi yourusername! You've successfully authenticated,
but GitHub does not provide shell access.
```

This message will show **your personal username**, not `govtmethods`. That is correct — you are authenticating as yourself. The "no shell access" part is also expected; GitHub SSH is only for Git operations.

If you see `Permission denied (publickey)`, the key is not loaded or not on GitHub. Re-run Step 2C, then retry.

### Step 2F — If you have two GitHub accounts on the same machine

Many researchers have both a personal GitHub account and a university or lab account. If the two accounts use different SSH keys, you need a config file to tell Git which key to use for which context.

Create or edit `~/.ssh/config`:

```
# Personal account — used for govtmethods repos
Host github.com
    HostName github.com
    User git
    AddKeysToAgent yes
    UseKeychain yes
    IdentityFile ~/.ssh/id_ed25519

# University/lab account — uses a separate key
Host github-university
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_university
```

With this setup:

- `git@github.com:govtmethods/methodsbrownbag.git` → uses your personal key automatically
- For university repos, change the remote URL to use the alias: `git@github-university:lab-org/repo.git`^[You can update an existing repo's remote with: `git remote set-url origin git@github-university:lab-org/repo.git`]

Test each account separately:

```bash
ssh -T git@github.com           # should say: Hi personal-username!
ssh -T git@github-university    # should say: Hi university-username!
```

If you only have one GitHub account, you do not need a config file — the defaults work.

## Step 3 — Create a workspace folder

This is just an ordinary folder on your computer to hold multiple repos side by side. It is not itself a Git repo.

```bash
mkdir -p ~/Documents/GitHub/govtmethods-workspace
```

After cloning (next step), the folder will contain:

```
govtmethods-workspace/
├── govtmethods.github.io/    ← landing site source
├── mathcamp/                 ← Math Camp source
└── methodsbrownbag/          ← Brown Bag source
```

## Step 4 — Clone the repos

Navigate into the workspace folder and clone each source repo:

```bash
cd ~/Documents/GitHub/govtmethods-workspace

git clone git@github.com:govtmethods/govtmethods.github.io.git
git clone git@github.com:govtmethods/mathcamp.git
git clone git@github.com:govtmethods/methodsbrownbag.git
```

**What `git clone` does:** it downloads the entire repo history to your machine and sets up `origin` as the name for the remote (the version on GitHub). You can now make changes locally and push them back.

You do **not** clone the deploy repo. CI manages it and contributors never edit it.

### Restore R packages after cloning

For `mathcamp` and `methodsbrownbag`, after cloning, open R inside the project directory and run:

```r
renv::restore()
```

This downloads and installs the exact package versions recorded in `renv.lock`, so your local environment matches what CI uses. (See Section 7 for a fuller explanation of renv.)

## Step 5 — Verify SSH push works

Pick one repo and do a harmless test:

```bash
cd ~/Documents/GitHub/govtmethods-workspace/govtmethods.github.io

git status         # should say: "On branch main, nothing to commit"
git pull           # sync any changes from GitHub

# Make a trivial change (add a blank line somewhere), then:
git checkout -b test/ssh-works
git add -A
git commit -m "Test: verify SSH push works"
git push -u origin HEAD
```

If the push succeeds, go to GitHub and delete the test branch (`govtmethods.github.io` → Branches → delete `test/ssh-works`). Your setup is complete.

If you see `Permission denied (publickey)`, your SSH key is not loaded. Run:

```bash
eval "$(ssh-agent -s)"
ssh-add --apple-use-keychain ~/.ssh/id_ed25519   # macOS
ssh -T git@github.com                             # confirm it works
```

Then retry the push.

\newpage

# Day-to-day editing workflow

Once onboarding is complete, every contribution follows the same loop.

## Option A — PR-based workflow (recommended)

This is the standard academic collaboration model: work on a branch, open a Pull Request for review, merge to deploy.

### Why branches?

A **branch** is an independent copy of the codebase where you can make changes without affecting the live site.^[Git branches are extremely lightweight — creating one takes milliseconds and uses almost no disk space. The `main` branch is the "production" version: whatever is on `main` gets deployed. Any branch other than `main` can be freely experimented with, broken, and discarded without consequence.] `main` is the branch that deploys to the live website. Your work lives on a separate branch until it has been reviewed and approved.

**Example:** You are writing a Brown Bag session on regression discontinuity. You create a branch called `content/regression-discontinuity`. You write your chapter there over several days, committing as you go. When finished, you open a Pull Request. Parushya reviews it, suggests a change, you update it, and she approves. Merging the PR deploys your chapter to the website.

### Step A1 — Sync your local `main`

Before starting anything new, get the latest version of `main`:

```bash
git checkout main
git pull
```

`git checkout main` switches you to the main branch. `git pull` downloads any changes others have pushed since you last synced.^[If you skip this step and someone else merged a change while you were working, you may end up with a **merge conflict** — two people edited the same part of a file and Git does not know which version to keep. Pulling before branching minimizes this risk.]

### Step A2 — Create a branch

Name the branch according to what you are doing:

| Prefix | Use for |
|---|---|
| `content/` | Writing or updating a chapter |
| `fix/` | Correcting a broken link, typo, or error |
| `feature/` | Adding a new structural element |
| `chore/` | Housekeeping (renaming files, updating dates) |

```bash
git checkout -b content/regression-discontinuity
```

This creates the branch and switches to it in one step.

### Step A3 — Write your content

Open the repo folder in your text editor or RStudio and write your `.qmd` file. See Section 5 for the full chapter creation workflow.

**Files you may edit:**

- `chapters/NN-name.qmd` (your chapter)
- `_quarto.yml` — but only to add your chapter to the `chapters:` list

**Files you must not edit:**

- `styles.scss` — the design system
- `_includes/` — shared HTML components (controls, toggles)
- `_filters/` — Lua filters for PDF output
- `.github/workflows/` — CI configuration
- Any other `_quarto.yml` settings beyond the `chapters:` list

### Step A4 — Preview your chapter locally

```bash
# Run from inside the repo directory
quarto preview
```

This starts a local web server and opens your browser to a live-reloading preview. As you save changes to `.qmd` files, the browser updates automatically.

> **Important limitation:** `quarto preview` does not run the post-render hook. This means the sidebar author label and the index session card for your chapter will not appear during preview — they are populated by a script (`_scripts/build-chapter-meta.R`) that only runs during a full `quarto render`. For writing and checking your prose, preview is sufficient. Before pushing, do one full render.

```bash
quarto render
```

This compiles everything, runs the post-render hook, and verifies there are no errors.

### Step A5 — What not to commit

**Never commit build output.** The following are generated files that Git should not track:

| Path | What it is |
|---|---|
| `_site/` | Rendered HTML (website output) |
| `_book/` | Rendered HTML + PDF (book output) |
| `.quarto/` | Quarto cache |
| `.Rproj.user/` | RStudio session state |
| `.DS_Store` | macOS folder metadata |
| `*.html.md`, `*.pdf.md` | Intermediate Pandoc files |

These should already be listed in `.gitignore`, meaning Git will ignore them automatically. If `git status` shows any of these paths as "untracked" or "modified", something is wrong with your `.gitignore`. Stop and check before committing.

### Step A6 — Stage and commit

```bash
git status   # review what changed
```

Stage only the files you intentionally changed:

```bash
git add chapters/03-regression-discontinuity.qmd
git add _quarto.yml    # only if you added your chapter to the list
```

Avoid `git add -A` or `git add .` — these stage everything, including files you didn't intend to include.^[`git add -A` has burned many contributors: it picks up unintended files like `.DS_Store`, generated outputs, or accidentally edited files. Staging files one by one or by explicit path is more deliberate and safer.]

Write a clear, specific commit message:

```bash
git commit -m "Add session: regression discontinuity design"
```

A good commit message answers "what did this change and why?" in one line. Bad: `"update"`. Good: `"Add session: regression discontinuity design"`.

### Step A7 — Push your branch

```bash
git push -u origin HEAD
```

`-u origin HEAD` sets the upstream so future pushes from this branch only need `git push`. After the first push, subsequent pushes are just:

```bash
git push
```

### Step A8 — Open a Pull Request

Go to `github.com/govtmethods/methodsbrownbag` (or whichever repo). GitHub usually shows a banner: "Your branch was recently pushed — compare and open a pull request." Click it.

Fill in:

- **Title:** a clear one-line description (`Add session: regression discontinuity`)
- **Description:** what you added, any notes for the reviewer, anything you want flagged
- **Reviewers:** assign Parushya (right sidebar)

Click **Create pull request**.

### Step A9 — Respond to review and merge

The reviewer may leave comments requesting changes. Make the edits locally, commit, and push — the PR updates automatically. Once approved:

- Click **Merge pull request**
- This triggers GitHub Actions
- The site updates within approximately 2–5 minutes

## Option B — Direct push to `main` (use sparingly)

Only when branch protections are off and the team has explicitly agreed:

```bash
git checkout main
git pull

# make your edits

git add chapters/NN-name.qmd _quarto.yml
git commit -m "Add session: title"
git push origin main
```

Deployment is immediate. Use this for small fixes (typo corrections, broken links) where review overhead outweighs the risk.

\newpage

# Adding a new chapter

This is the most common contributor task. Follow these steps in order.

## Step 1 — Create the chapter file

Navigate to the `chapters/` directory in the repo. Create a new file named with the next available number:

```
chapters/
├── 01-intro.qmd
├── 02-github.qmd
└── 03-regression-discontinuity.qmd   ← your new file
```

Every chapter file **must** begin with a YAML front matter block containing exactly these three fields:

```yaml
---
title: "Regression Discontinuity Design"
author: "Your Full Name"
date: "2026-04-27"
---
```

**Why these fields are required — not optional:**

These three fields power four separate features of the site automatically:

| Feature | Where it appears | Powered by |
|---|---|---|
| Sidebar author label | Below chapter title in left nav (HTML) | JS reading `chapters-meta.json` |
| Index session card | Chapter listing on the home page (HTML) | JS reading `chapters-meta.json` |
| PDF chapter listing | Table of chapters in book PDF | R code in `index.qmd` |
| PDF chapter byline | Author + date below chapter heading in PDF | Lua filter |

Missing any of these fields causes silent gaps: your chapter appears but without attribution, or the index card shows no author.

**Date format:** use `YYYY-MM-DD` (e.g., `2026-04-27`). This is the session date, not today's date.

**Multi-author chapters:** if the session had two presenters, separate names with a comma:

```yaml
author: "First Author, Second Author"
```

## Step 2 — Write your content

After the YAML block, write your chapter in Quarto Markdown. A minimal example:

```markdown
---
title: "Regression Discontinuity Design"
author: "Your Name"
date: "2026-04-15"
---

## Introduction

Regression discontinuity (RD) designs exploit a threshold rule...

## The basic setup

Let $Y_i$ be the outcome for unit $i$, and $X_i$ be the running variable...
```

Math equations use standard LaTeX notation: `$inline$` or `$$display$$`.

Code blocks are fenced with triple backticks and a language label:

````markdown
```{r}
library(tidyverse)
df |> ggplot(aes(x = score, y = outcome)) + geom_point()
```
````

## Step 3 — Register your chapter in `_quarto.yml`

Open `_quarto.yml`. Find the `chapters:` list under `book:` and add your file:

```yaml
book:
  chapters:
    - index.qmd
    - chapters/01-intro.qmd
    - chapters/02-github.qmd
    - chapters/03-regression-discontinuity.qmd   # add this line
```

**This is the only change you should make to `_quarto.yml`.** Do not touch any other settings in this file.

## Step 4 — Run a full render

```bash
quarto render
```

This does two things you need:

1. Compiles your chapter into HTML and PDF — confirms there are no syntax errors
2. Runs `_scripts/build-chapter-meta.R`, which reads the `chapters:` list, parses each `.qmd` YAML block, and writes `_book/chapters-meta.json`

The JSON file is what the sidebar and index page read at runtime. Without it, `quarto preview` works but the deployed site would show outdated sidebar labels.

If the render succeeds with no errors, you are ready to commit.

## Step 5 — Commit and push

```bash
git add chapters/03-regression-discontinuity.qmd
git add _quarto.yml

git commit -m "Add session: regression discontinuity design"
git push -u origin HEAD
```

Open a PR and request review as described in Section 4.

\newpage

# R package reproducibility with `renv`

## What the problem is

Imagine two contributors: one installed the `tidyverse` package six months ago, the other just installed it yesterday. The package authors released a new version in between that changed how a function works. Now the same code produces different output on the two machines — and on the CI server, which has yet another version.

`renv` solves this by locking every R package to a specific version, recorded in a file called `renv.lock`.^[`renv.lock` is a plain-text JSON file listing every package, its version, and its source (CRAN, GitHub, etc.). When you run `renv::restore()`, renv reads this file and installs exactly those versions, ignoring anything newer in CRAN. This means a chapter rendered in April 2026 produces the same output in April 2028, even if package APIs have changed.]

## What `renv` means for contributors

When you clone `mathcamp` or `methodsbrownbag` for the first time, open R in that directory and run:

```r
renv::restore()
```

This installs exactly the packages listed in `renv.lock`. You do not need to install packages yourself. Do not run `install.packages()` directly in these projects — that bypasses renv and may create version mismatches.^[If you accidentally install a package with `install.packages()`, run `renv::status()` to see what is out of sync, and `renv::restore()` to return to the locked state.]

## If you need to add a new package

Work inside the relevant repo directory in R:

```r
renv::install("packagename")   # installs and records in renv
renv::snapshot()                # updates renv.lock with the new package
```

Then commit the updated lockfile:

```bash
git add renv.lock
git commit -m "Add R package: packagename"
git push
```

CI will automatically use the updated lockfile on the next build.

\newpage

# Deployment: what happens and when

## The deploy token (you do not handle this)

CI deployments use a secret called `PAGES_DEPLOY_PAT` stored in each source repo's settings. This is a GitHub Personal Access Token owned by the `govtmethods` admin account that authorizes CI to push rendered output into the deploy repo. As a contributor, you never see or need this token — it is an infrastructure concern for maintainers only.

## Where your content ends up

| Source repo | Renders to | Public URL |
|---|---|---|
| `govtmethods.github.io` | `/` in deploy repo | `https://govtmethods.github.io/` |
| `mathcamp` | `/mathcamp/` | `.../mathcamp/` |
| `methodsbrownbag` | `/methodsbrownbag/` | `.../methodsbrownbag/` |

## How long deployment takes

Deployment time depends on whether R packages are cached from a previous successful run:

| Situation | Expected time |
|---|---|
| First deploy, or after `renv.lock` changed | 5–10 minutes (packages build from source) |
| Subsequent deploys, same `renv.lock` | 1–3 minutes (packages restored from cache) |

You can watch the live build log at: `github.com/govtmethods/<repo>/actions`.

\newpage

# Troubleshooting

## "Repository not found" or "Permission denied"

**Most likely cause:** SSH key not loaded, or not added to GitHub.

Diagnosis:

```bash
ssh -T git@github.com
```

If this returns `Permission denied (publickey)`, your key is not loaded. Fix:

```bash
eval "$(ssh-agent -s)"
ssh-add --apple-use-keychain ~/.ssh/id_ed25519   # macOS
ssh -T git@github.com                             # retry
```

If `ssh -T` works but `git push` still fails with "Repository not found", check that you accepted the collaborator invitation (Step 0) and that the remote URL uses SSH not HTTPS:

```bash
git remote -v
# should show:  git@github.com:govtmethods/...
# not:          https://github.com/govtmethods/...
```

To fix an HTTPS remote:

```bash
git remote set-url origin git@github.com:govtmethods/methodsbrownbag.git
```

## SSH agent loses key after reboot (macOS)

If you need to run `ssh-add` every time you restart your Mac, the Keychain configuration is missing. Add this to `~/.ssh/config`:

```
Host github.com
    HostName github.com
    User git
    AddKeysToAgent yes
    UseKeychain yes
    IdentityFile ~/.ssh/id_ed25519
```

## Sidebar author labels or index cards not showing

You ran `quarto preview` but not `quarto render`. The post-render script only runs on a full render.

```bash
quarto render
```

If the labels still do not appear locally, check that your chapter file has `author:` in the YAML front matter and that the chapter path is listed in `_quarto.yml` under `chapters:`.

## Your chapter renders locally but fails in CI

**Most likely causes:**

1. **Missing package in `renv.lock`:** You used a package locally that is not in the lockfile. Run `renv::snapshot()` locally and commit the updated `renv.lock`.

2. **Code that works locally but not in a clean environment:** Your chapter may rely on a local file, an environment variable, or a package loaded in `.Rprofile` that CI does not have. Make sure all data and packages are loaded explicitly inside the chapter.

3. **R version mismatch:** CI uses R 4.5.2. If you are running an older version locally, some code may behave differently. Check: `R --version`.

## CI fails with "Write access not granted" (403)

This is a deploy token issue — the `PAGES_DEPLOY_PAT` secret is missing or expired. You cannot fix this yourself. Contact Parushya with the Actions run URL.

\newpage

# Quick reference card

## Daily commands

```bash
# Sync main before starting work
git checkout main && git pull

# Create a branch
git checkout -b content/your-topic

# Preview (live reload, no post-render hook)
quarto preview

# Full render (required before pushing)
quarto render

# Stage specific files
git add chapters/NN-name.qmd
git add _quarto.yml          # only if you updated the chapters list

# Commit
git commit -m "Add session: your chapter title"

# Push branch (first push)
git push -u origin HEAD

# Push branch (subsequent pushes)
git push
```

## SSH commands

```bash
# Check if key is loaded
ssh -T git@github.com

# Load key (macOS)
eval "$(ssh-agent -s)"
ssh-add --apple-use-keychain ~/.ssh/id_ed25519

# Check what remote URL a repo is using
git remote -v

# Fix remote from HTTPS to SSH
git remote set-url origin git@github.com:govtmethods/REPONAME.git
```

## R (inside a book project)

```r
renv::restore()          # install locked packages after cloning
renv::install("pkg")    # install a new package
renv::snapshot()         # update renv.lock after installing
renv::status()           # check if environment matches lockfile
```

## Chapter YAML template

```yaml
---
title: "Your Session Title"
author: "Your Full Name"
date: "YYYY-MM-DD"
---
```

\newpage

# Contact

If something breaks, send the following to a maintainer:

1. Which repo you were in (`methodsbrownbag`, `mathcamp`, or `govtmethods.github.io`)
2. The exact command you ran
3. The full error output (copy-paste from Terminal or the CI log)
4. If a CI failure: the link to the failing Actions run (e.g., `github.com/govtmethods/methodsbrownbag/actions/runs/12345`)

**Maintainers:**

- **Parushya** — infrastructure, CI, theme, SSH access
- **Sara** — content review, editorial standards
