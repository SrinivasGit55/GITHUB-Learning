# Git & GitHub - Advanced Branch Operations, .gitignore & Restore - Professional Notes

## 1. Optimized branch creation - single command

Instead of two commands to create a branch and switch to it, use a single command.

```bash
git branch movies
git checkout movies
```

The above two steps can be done in one command:

```bash
git checkout -b <branch-name>   # old but still widely used
git switch -c <branch-name>      # modern Git (2.23+)
```

Example:

```bash
git checkout -b dth
```

Output:

```bash
Switched to a new branch 'dth'
```

This performs two operations:
1. Creates the `dth` branch
2. Switches to it

Verification:

```bash
git branch
```

The current branch is marked with `*`.

Error case:

```bash
git checkout dth
```

If the branch does not exist, it will fail with an error similar to:

```bash
error: pathspec 'dth' did not match any file(s) known to git
```

Note: When you create a new branch from an existing branch, all committed files from the current branch are carried forward. A branch is a pointer to a commit, not a file copy.

In real-world examples, branch names often reflect workflows rather than service names, for example:
- `main`
- `master`
- `dev`
- `qa`
- `release/v1.0`
- `hotfix`

For learning purposes, we often use examples such as:
- `movies`
- `train`
- `dth`
- `recharge`

---

## 2. Pattern to follow for each developer

This is the standard workflow used in real projects:

```bash
# 1. Create branch
# 2. Switch to branch
# 3. Create files
# 4. Add files
# 5. Commit
# 6. Push to GitHub
```

Example:

```bash
git checkout -b movies
touch movies1.html movies2.html
git add movies*
git commit -m "Developer 1 - movies service"
git push origin movies
```

The same pattern repeats for `train`, `dth`, and `recharge` branches. Each developer pushes to the same repository but to a different branch.

---

## 3. `.gitignore` - ignore files you do not want to track

Purpose:
- Keep files in the local folder
- Do not track them
- Do not commit them
- Do not push them to GitHub
- Do not carry them into new branches

Common use cases:
- Java `.class` files
- Python `__pycache__`
- `node_modules`
- log files
- `.env` secrets

### Implementation

The file name must be exactly `.gitignore`.

```bash
# Create some sample files
touch Java1.java Java2.java python1.py python2.py raham.txt

git status
# Shows all files as untracked

# Create .gitignore
vi .gitignore
```

Inside `.gitignore`, add patterns like:

```gitignore
Java1.java
*.java
*.class
python*
p*
```

Save with:

```bash
Esc
:wq
Enter
```

Then check:

```bash
git status
```

Ignored files will not appear in the untracked list, but they still exist in the folder.

```bash
ls -l
```

Important behavior:

```bash
git add *
```

If files are in `.gitignore`, Git will block them with a message similar to:

```bash
The following paths are ignored by one of your .gitignore files:
Use -f if you really want to add them.
```

Force add if necessary:

```bash
git add -f Java1.java
```

Ignored files remain only on local Git. They never reach GitHub because they are not tracked or committed.

### Common `.gitignore` template

```gitignore
# Java
*.class
*.jar

# Python
__pycache__/
*.pyc

# Node
node_modules/

# Logs
*.log

# OS
.DS_Store
Thumbs.db
```

---

## 4. Branch rename operations

### A. Rename local branch (CLI)

```bash
git branch -m <old-name> <new-name>
```

`-m` means modify or move.

Examples:

```bash
git branch -m master raham
git branch -m raham main
git branch -m main master
git branch
```

Only the branch name changes; the files remain the same.

Important: command syntax is `git branch -m old-name new-name`. Replace `old-name` and `new-name` with actual names.

### B. Rename remote branch (GitHub web UI)

1. Go to GitHub repo
2. Open the Branches section
3. Click the edit/pencil icon next to the branch name
4. Enter the new name and press Enter

Example:
- `recharge` -> `raham`

### Sync behavior
Renaming on GitHub does not automatically rename the local branch.
Renaming locally does not rename the remote branch.
They must be managed separately.

- Local to remote: use CLI or push commands
- Remote UI: use GitHub web interface

CLI mode is used heavily by DevOps engineers.
GUI mode is used by developers working via GitHub UI.

---

## 5. Branch delete and recovery

### A. Delete local branch

```bash
git branch -D <branch-name>     # force delete
git branch -d <branch-name>      # safe delete (only if merged)
```

Important rule: you cannot delete the currently checked-out branch.

Example:

```bash
git branch
# * recharge
#   master
#   movies

git branch -D recharge
```

If you are on `recharge`, Git will block the deletion.

Correct process:

```bash
git checkout movies
git branch -D recharge
git branch
```

After deletion, the local branch pointer and branch files are removed from the local repo.

### B. Recover deleted branch

If the branch was pushed to GitHub before deletion, it can often be recovered from the remote.

```bash
git pull origin recharge
```

This fetches and merges the deleted branch from remote into local, if it still exists on GitHub.

If it does not appear immediately:

```bash
git fetch --all
git pull origin <branch-name>
git checkout <branch-name>
```

Definition:
- `git push` = send local changes to GitHub
- `git pull` = get remote branches/files into local Git

### C. Delete remote branch on GitHub

GitHub UI:
- open branch list
- click delete/trash icon

CLI:

```bash
git push origin --delete <branch-name>
```

### D. Restore deleted remote branch

If a remote branch was deleted recently, GitHub may show a restore option if branch protection allows it.

Otherwise, if the branch still exists locally, push it again:

```bash
git push origin train
```

---

## 6. `git restore` command - common use cases

### Use case 1: unstage a tracked file

When you mistakenly staged a file using `git add`:

```bash
touch raham.txt
git status
# untracked file
git add raham.txt
git status
# file is now staged

git restore --staged raham.txt
# or: git reset HEAD raham.txt (older style)

git status
# file is back to untracked
```

Purpose: untrack or unstage a file.

### Use case 2: recover a deleted tracked file

You can restore only tracked files.

Example:

```bash
mkdir ABCD && cd ABCD
touch file1.txt
git init
git add file1.txt
git commit -m "add file1"
rm -f file1.txt
ls
# file1.txt is gone
git restore file1.txt
ls
# file is back
```

### Example of untracked file (cannot be restored)

```bash
touch file2.txt
rm -f file2.txt
git restore file2.txt
```

This usually fails because the file was never added to Git.

### Bulk restore

```bash
touch file{1..100}.txt
git add *
rm -f *
git restore *
ls
```

All tracked files can be restored at once.

Key interview point:
- We can restore only tracked files.
- Untracked files cannot be restored by `git restore`.
- There is no strict limit on the number of files that can be restored in one command.

Where restored files come from:
- Git stores file snapshots in the `.git` directory when you `git add` and commit.
- Deleting the file from working directory does not erase the tracked version from Git history.

---

## 7. Push limitations and branch behavior

### Can we push multiple branches in one command?

In practice, many users push one branch at a time for clarity. However, Git does support multiple references in one push if you specify several refspecs.

Example:

```bash
git push origin master movies
```

This is valid Git syntax, but in real projects it is usually safer and clearer to push one branch at a time.

#### Safer pattern:

```bash
git push origin master
git push origin movies
```

### Can we ignore a branch from pushing?

Yes. `.gitignore` is only for files, not branches.

- To ignore a branch, do not push it.
- It remains local only.

---

## 8. Pull request concept (preview)

If you are on the `recharge` branch and want to merge its contents into the `movies` branch, a direct `git push` from `recharge` to `movies` is not the correct workflow.

The correct method is a Pull Request (PR):

1. Go to GitHub
2. Open Pull Requests
3. Click `New Pull Request`
4. Base branch: `movies`
5. Compare branch: `recharge`
6. Review differences
7. Create and merge the PR

After merging, the `movies` branch on GitHub contains the changes from `recharge`.

This will be covered in detail in the next class.

---

## Final summary

Key topics covered in this class:
- efficient branch creation
- developer branch workflow
- `.gitignore` usage for ignored files
- rename and delete branches
- recovering deleted branches
- `git restore` for staged and deleted tracked files
- push behavior and GitHub pull-request basics

This is a core Git workflow used in real DevOps and software engineering teams.
