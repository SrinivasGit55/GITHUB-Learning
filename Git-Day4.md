# Git Day 4 Notes

## 1. Git merge - definition

Merge is used to bring changes from one branch into another branch.

Use case:
- Developer worked on `movies` branch.
- Now we want the code from `movies` to be available in `master`.

Command:

```bash
git merge <source-branch-name>
```

Important:
- You must be on the destination branch before merging.
- Example: merge `movies` into `master`.

```bash
git checkout master
# or: git switch master

git merge movies
```

### Example workflow

```bash
git init
touch index.html
git add .
git commit -m "Initial commit - master has 1 file"

git checkout -b movies
touch movies1.html movies2.html movies3.html movies4.html movies5.html
git add .
git commit -m "Developer 1 - movies"

git checkout master
git merge movies
```

After merge, the files from `movies` are present in `master`.

Example note:
- If the branch is already up to date, Git will say: `Already up to date.`

---

## 2. Git rebase - definition

Rebase is also used to apply changes from one branch onto another branch.

Functionally, both merge and rebase can bring the same code into the destination branch, but their history behavior is different.

Command:

```bash
git rebase <source-branch-name>
```

Example:

```bash
git checkout master
git rebase dth
```

This moves the current branch so that the branch history is replayed on top of the target branch.

### Important note
- Rebase can rewrite history.
- It is best used on local feature branches that have not been shared with others.
- Avoid rebasing public/shared branches without coordination.

---

## 3. Merge vs rebase - interview answer

This is a very common Git interview question.

### Merge
- Brings changes from one branch into another
- Keeps original history intact
- Creates a merge commit
- History tends to become non-linear
- Safer for shared/public branches such as `main` or `develop`

Example:

```bash
git checkout master
git merge dth
```

### Rebase
- Also brings changes from one branch into another
- Replays commit history on top of the destination branch
- Produces a cleaner, linear history
- Rewrites commit history
- Best for local feature branches before opening a PR

Example:

```bash
git checkout master
git rebase dth
```

### Practical difference

If you run both commands on the same branch:

```bash
git checkout master
git merge dth
```

and then:

```bash
git checkout master
git rebase dth
```

both can result in the same files being present in `master`, but the commit history will look different.

### HEAD
`HEAD` points to the latest commit on the current branch.

After merge or rebase, `HEAD` moves to the newest commit in the branch.

### Graph view
To see a clearer history:

```bash
git log --oneline --graph --decorate --all
```

This helps you visually inspect whether history is merged or rebased.

---

## 4. Merge on GitHub - Pull Request

Local Git commands are not the only way to merge branches. In real projects, GitHub uses Pull Requests (PRs).

### Example scenario
- `train` branch has train-specific files
- `recharge` branch has all recharge-related files
- You want to merge `recharge` files into `train`

### Steps on GitHub
1. Open the repository
2. Go to `Pull Requests`
3. Click `New Pull Request`
4. Set:
   - Base branch: `train`
   - Compare branch: `recharge`
5. GitHub shows commit difference and file differences
6. Click `Create Pull Request`
7. Review the PR
8. Click `Merge Pull Request`
9. Confirm merge

After merge:
- `train` branch will include the changes from `recharge`

Important note:
- If you want to merge on GitHub, you normally use a Pull Request.

---

## 5. Branch delete and restore - CLI and GUI

### A. Delete local branch

```bash
git branch -D <branch-name>    # force delete
git branch -d <branch-name>     # safe delete
```

Important:
- You cannot delete the branch currently checked out.

Example:

```bash
git branch
# * recharge
#   master
#   movies

git checkout master
git branch -D recharge
```

This removes the `recharge` branch from the local repo.

### B. Recover deleted local branch from GitHub

If the branch existed remotely and was pushed earlier, it can often be recovered.

```bash
git pull origin <branch-name>
git checkout <branch-name>
```

This fetches the branch from GitHub and creates a local pointer to it.

### C. Delete remote branch on GitHub

From the web UI:
- open repo
- open `Branches`
- click trash/delete icon for the branch

From CLI:

```bash
git push origin --delete <branch-name>
```

### D. Restore deleted remote branch

GitHub sometimes provides a restore option after a branch is deleted, depending on settings.

If not visible, you can restore it by pushing it again from local:

```bash
git push origin <branch-name>
```

### Real-world permission note
In companies, developers often do not have permission to delete branches on GitHub.
- Only leads or managers may have the right to delete protected branches.
- PAT scopes are usually limited to push/pull access rather than full admin rights.

---

## 6. Git stash - hide uncommitted work temporarily

Git stash is used when you want to temporarily save work that is tracked but not committed, and then switch branches without committing half-finished work.

### Why use stash?
A developer may be in the middle of editing a file, but must switch to another task urgently.

If the work is not ready to commit, stash is useful.

### Important difference: stash vs .gitignore

#### `.gitignore`
- tells Git to ignore files
- files still exist in the folder
- files do not appear in `git status`

#### `git stash`
- stores tracked, modified/uncommitted work temporarily
- files are hidden from working directory
- they are moved into a stash area managed by Git

### Note
A file must be tracked and modified (or staged) before it can be stashed.

### Example

```bash
mkdir ABCD && cd ABCD
git init

touch Java1.java
git add Java1.java
git commit -m "Java1 committed"

touch Java2.java
git add Java2.java
git status
# Shows Java2.java as staged but not committed

git stash
# or: git stash push -m "half work of Java2"

ls -l
# Java2.java is hidden from working directory

git status
# Clean working tree

git stash list
# Lists saved stashes

git stash apply
# or: git stash pop
```

### Common stash commands

```bash
git stash
# hide tracked but uncommitted changes
git stash list
# list all stashes
git stash apply
# restore latest stash without deleting it
git stash pop
# restore latest stash and remove it from stash list
git stash apply stash@{0}
# restore specific stash by index
git stash clear
# delete all stashes
git stash push -m "message"
# stash with custom message
```

### Interview statement
Question: What is `git stash`?

Answer: It is used to temporarily store tracked but uncommitted changes so that the developer can switch branches or work on something else without committing incomplete work.

---

## 7. Final summary

This class covers the most important Git workflow concepts used in professional projects:
- `git merge`
- `git rebase`
- merge vs rebase interview differences
- GitHub pull request workflow
- local and remote branch deletion
- branch restore/recovery
- `git stash`

These are core concepts for DevOps and software engineering roles.
