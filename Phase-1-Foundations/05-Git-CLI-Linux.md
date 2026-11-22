---
tags: [phase-1, git, cli]
---
# 05 — Git, the Command Line & Linux

> On Windows: install **WSL2** (Ubuntu) or use Git Bash. Most servers, containers and CI run Linux, so get fluent.

## 1. Shell fundamentals
```bash
pwd; ls -la; cd dir; mkdir -p a/b; cp -r src dst; mv a b; rm -rf dir   # rm -rf is irreversible!
cat f; less f; head -n 20 f; tail -f log.txt        # tail -f follows a log
grep -rn "TODO" src/                                  # search; also `rg` (ripgrep) is faster
find . -name "*.py" -mtime -1
wc -l f; sort | uniq -c | sort -rn | head            # classic pipeline: top frequencies
curl -s https://httpbin.org/get | jq '.headers'       # HTTP + JSON from CLI
```
- **Pipes** `|` connect stdout → stdin. **Redirects**: `>` overwrite, `>>` append, `2>&1` merge stderr, `< file`.
- **Env vars:** `export KEY=value`, `echo $KEY`, `.env` files (never commit them!).
- **Permissions:** `chmod +x script.sh`, `ls -l` (rwx for user/group/other), `chown`, `sudo`.
- **Processes:** `ps aux`, `top`/`htop`, `kill -9 PID`, `&` background, `nohup`, `systemctl`.
- **Networking:** `curl`, `wget`, `ping`, `ss -tulpn` (what's on which port), `dig`, `ssh`, `scp`.
- **Scripting:** variables, `if`, `for`, functions, `set -euo pipefail` (fail fast). Know when to switch to Python instead (anything beyond ~30 lines).
- **Tools to learn:** `jq`, `rg`, `fd`, `tmux`, `make`, `xargs`, `sed`/`awk` basics.
- **SSH keys**: `ssh-keygen -t ed25519`, public vs private key. Never share the private key.

## 2. Git mental model
Git is a **content-addressed, distributed database of snapshots** (commits) forming a DAG.
- **Working tree** → (`git add`) → **staging area/index** → (`git commit`) → **local repo** → (`git push`) → **remote**.
- A **commit** = snapshot + parent pointer(s) + message + author. A **branch** = movable pointer to a commit. `HEAD` = where you are.

### Daily commands
```bash
git init / git clone URL
git status ; git diff ; git diff --staged
git add -p                         # stage hunks interactively (good habit)
git commit -m "feat: add retry decorator"
git log --oneline --graph --all
git switch -c feature/x            # new branch
git switch main ; git merge feature/x
git fetch ; git pull --rebase ; git push -u origin feature/x
git stash ; git stash pop
git restore file ; git restore --staged file
git revert <sha>                   # safe undo (new commit)
git reset --hard <sha>             # destructive; rewrites local state
git rebase main                    # replay your commits on top of main
git rebase -i HEAD~3               # squash/reword (local branches only)
git cherry-pick <sha>
git bisect start/good/bad          # binary-search for the commit that broke something
git reflog                         # your safety net: recover "lost" commits
```
### Workflow (GitHub flow)
1. Branch from `main` → 2. small commits → 3. push → 4. open **Pull Request** → 5. CI runs + review → 6. squash-merge → 7. delete branch.
- **Conventional Commits**: `feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `chore:`.
- **Never** force-push shared branches. **Never** commit secrets (API keys). If you did: rotate the key *immediately*; removing from history isn't enough.
- `.gitignore`: `.venv/`, `__pycache__/`, `.env`, `*.pyc`, `node_modules/`, data/model files.
- **Merge conflicts**: edit the `<<<<<<<` markers, `git add`, `git rebase --continue`/`git commit`.
- Merge vs rebase: merge preserves history; rebase gives linear history. Rule: *don't rebase what others have pulled.*

### GitHub features to use
Issues, PRs, Actions (CI), Projects, Releases, Codespaces, Dependabot, branch protection, `gh` CLI (`gh pr create`). A **clean GitHub profile with READMEs is your portfolio.**

## 3. Linux/OS advanced
- Filesystem hierarchy (`/etc`, `/var/log`, `/usr/bin`, `/home`, `/proc`), inodes, symlinks.
- Process signals (SIGTERM vs SIGKILL — graceful shutdown in services), `cron`/systemd timers, `journalctl`.
- Resource limits, `ulimit`, OOM killer, `free -h`, `df -h`, `du -sh`, `nvidia-smi` (GPU).
- Networking: iptables/firewalls, ports, reverse proxies (nginx/Caddy), TLS certs (Let's Encrypt).
- Containers use Linux namespaces + cgroups (→ [[08-Testing-Tooling-Docker]]).

## Practice
- Learn Git Branching (interactive), OverTheWire Bandit (shell wargame), MIT Missing Semester — links in [[12-Resources-and-Practice-Links]].

Next → [[06-HTTP-APIs-JSON]]
