# Git & GitHub - Day 2 Notes

Topics: Git config, `git show`, branching strategy, push/pull, and professional best practices.

## 1. Git configuration (user identity)

Git configuration tracks authorship and can be set at three levels (priority: local > global > system):

- system: `/etc/gitconfig` — applies to all users on the machine
- global: `~/.gitconfig` — applies to all repos for the current user (recommended for daily use)
- local: `.git/config` — applies only to the current repository

Essential commands

```bash
# Set config globally (recommended)
git config --global user.name "Your Name"
git config --global user.email "your.email@company.com"

# Set config locally for the current repo
git config --local user.name "Your Name"
git config --local user.email "your.email@company.com"

# View configuration
git config --list
git config --list --show-origin
cat ~/.gitconfig
```

Key points
- Git may block your first commit if user.name and user.email are not set.
- The GitHub contribution graph (green squares) is tied to the email in your commits; ensure `user.email` matches your GitHub account email (or a verified email on GitHub).
- Changing config does not rewrite past commits. To update the author of the last commit:

```bash
git commit --amend --author="Name <email@example.com>"
```


## 2. Inspecting commits with `git show`

`git show` shows details for a single commit (author, date, message, and diff).

```bash
git show <commit-id>              # full commit details and diff
git show --name-only <commit-id>  # list filenames changed by the commit
git show --stat <commit-id>       # summary stat of insertions/deletions
git show HEAD                     # the most recent commit
git show HEAD~1                   # the commit before last
git show <commit-id>:index.html   # show the file content at that commit
```

Concept clarity
- `git log` lists commits; `git show` inspects one commit; `git diff` shows unstaged (or inter-commit) differences.
- Short commit IDs (first 7 chars) are commonly used but are only unique in the repo context.

Practical tip: when a production build broke after a push, run `git show HEAD` (or inspect the specific commit) to quickly see what changed.


## 3. Branching (core concept)

A branch is an independent line of development. Branches are lightweight: creating a branch just creates a pointer to a commit — files are not copied.

Real-world example: a service-based app (Movies, Train, DTH, Recharge) where each developer works on a dedicated feature branch.

Branching strategies
- GitFlow: `main` (production), `develop`, `feature/*`, `release/*`, `hotfix/*` — good for larger teams with formal releases
- GitHub Flow: `main` + short-lived `feature/*` branches and pull requests — simple and common for many teams
- Trunk-Based Development: short-lived branches or direct commits to `main` with feature toggles — used in high-velocity teams

Default branch note
- The default branch (e.g., `master` or `main`) only appears after the initial commit; running `git init` alone does not create a named branch that shows in `git branch` until a commit exists.

Commands (modern and classic)

```bash
# Listing
git branch            # list local branches
git branch -a         # list local and remote branches
git branch -r         # list remote branches

# Create and switch (recommended modern commands)
git switch -c <branch-name>    # create and switch to a new branch
git switch <branch-name>       # switch to an existing branch

# Classic approach
git checkout -b <branch-name>  # create + switch (equivalent to switch -c)

# Delete and rename
git branch -d <branch-name>    # safe delete (only if merged)
git branch -D <branch-name>    # force delete
git push origin --delete <branch-name>  # delete remote branch
git branch -m <old> <new>      # rename local branch
```

Important: uncommitted changes remain in your working directory and will be visible on the branch you switch to. Commit or stash before switching.


## 4. Pushing and pulling (local <-> remote)

Definitions:
- Push: upload local commits to a remote (GitHub)
- Pull: download commits from remote and merge into current branch (`git pull = git fetch + git merge`)

Connect local repo to remote

```bash
git remote add origin https://github.com/username/repo.git
git remote -v
git remote set-url origin <new-url>
git remote remove origin
```

HTTPS vs SSH
- HTTPS: `https://github.com/user/repo.git` — uses Personal Access Token (PAT) for authentication (recommended for users who prefer HTTPS)
- SSH: `git@github.com:user/repo.git` — uses SSH keys; one-time setup and then passwordless pushes (preferred in many orgs)

SSH key example (recommended modern key type):

```bash
ssh-keygen -t ed25519 -C "your.email@company.com"
cat ~/.ssh/id_ed25519.pub   # add this to GitHub -> Settings -> SSH and GPG keys
```

Personal Access Tokens (PAT)
- GitHub removed password authentication for Git operations on Aug 13, 2021.
- Generate a token at: GitHub -> Settings -> Developer settings -> Personal access tokens (classic) or Fine-grained tokens
- Scopes: for repo work, enable `repo` (or use fine-grained repo access for least privilege)
- Copy the token immediately — it is shown only once. Store it securely and never commit it.

Credential helper (optional):

```bash
git config --global credential.helper store   # stores credentials unencrypted on disk (use with caution)
git config --global credential.helper cache   # cache in memory (Linux)
```

Push commands

```bash
git push origin <branch-name>        # push a branch
git push -u origin <branch-name>     # set upstream (use once)
git push                             # after upstream set, this pushes current branch
git push origin master:main          # push local master to remote main
```

Cloning vs initializing
- `git init` creates a new empty repo locally
- `git clone <url>` copies an existing remote repo locally (it sets up origin and fetches history)


## 5. `git pull` — details and best practices

`git pull` = `git fetch` + `git merge` by default. This can produce merge commits.

Recommended safe flow used in many companies:

```bash
# safe approach
git fetch origin
git log HEAD..origin/main   # inspect incoming commits
git merge origin/main       # merge when ready

# quick (one-step)
git pull origin main

# rebase instead of merge to keep linear history
git pull --rebase origin main
```

Common commands

```bash
git pull                # pull from default remote/branch
git pull origin main    # pull specific remote/branch
git fetch --all         # fetch all remotes
git pull --all          # pull from all remotes (less common)

# Force local to match remote (dangerous, will discard local changes)
git fetch origin
git reset --hard origin/main
```

Advanced options
- `git pull --rebase` keeps history linear by replaying local commits on top of fetched commits.
- `git pull --no-commit` will perform the merge but stop before creating a merge commit (you can review and edit before committing).


## 6. Quick Q&A / Interview checklist

- What is Git and why use it? (distributed VCS, rollback, collaboration)
- Stages of Git: working directory, staging area (index), repository (commits)
- CVCS vs DVCS: centralized (SVN) vs distributed (Git)
- How to view history: `git log`, `git log --oneline`, `git log -p`
- Branch operations: create, switch, delete, rename
- `git pull` vs `git fetch`; `git pull` merges, `git fetch` does not
- `git merge` vs `git cherry-pick` — merge brings all changes; cherry-pick applies a specific commit
- `git clone` vs `fork` — clone copies to local; fork copies to another GitHub account
- What is `.gitignore`? Avoid tracking files you don't want in history
- What is `git stash`? Temporary store uncommitted changes
- `git revert` vs `git restore` — revert makes a new commit that undoes a previous commit; restore restores files in working directory (or unstages/stages depending on flags)


## 7. Pull request / workflow notes (high level)
- Create a feature branch, commit locally, push to remote
- Create a Pull Request (PR) on GitHub to merge into target branch
- Use PRs for code review, CI checks, and merge policies
- Protect `main` or release branches with branch protection rules (require PR review, status checks, disallow force-push/deletes)


---

I will add this as `Git-Day2-Professional.md` in your repository `SrinivasGit55/GITHUB-Learning` with the commit message: "Add Git Day2 professional notes (proofread & corrected)". Proceeding to create the file now.
