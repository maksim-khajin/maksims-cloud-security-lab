# Git and GitHub setup

Ubuntu 26.04 under WSL2 on Windows. Set up 2026-09-05.

## SSH key

    ssh-keygen -t ed25519 -C "label@machine"   # creates ~/.ssh/id_ed25519(.pub)
    cat ~/.ssh/id_ed25519.pub                  # public part -> GitHub Settings > SSH keys
    ssh -T git@github.com                      # test; expect "Hi <user>!"

Change the passphrase without regenerating the key:

    ssh-keygen -p -f ~/.ssh/id_ed25519

Reload the agent after changing it, or it keeps offering the old one:

    ssh-add -d ~/.ssh/id_ed25519 && ssh-add ~/.ssh/id_ed25519
    ssh-add -l

## Global config

    git config --global user.name "Full Name"
    git config --global user.email "<id>+<user>@users.noreply.github.com"
    git config --global init.defaultBranch main
    git config --global pull.rebase false
    git config --global core.editor nano
    git config --global --list

The noreply address comes from GitHub Settings > Emails once
"Keep my email addresses private" is on. It keeps the personal
address out of commit history, which is public and permanent.

## Daily cycle

    git status                 # what changed
    git add .                  # stage everything
    git commit -m "message"    # record a snapshot
    git push                   # send it to GitHub

## Things that tripped me up

- Two different shells. A `PS C:\...>` prompt is PowerShell on Windows:
  `wsl`, `docker`, `minikube`. A `user@host:~$` prompt is Ubuntu:
  `git`, `apt`, `ssh-keygen`. A Windows command run inside Ubuntu
  answers "command not found".
- Work in `~`, never in `/mnt/c/`. The Windows filesystem seen from WSL
  is much slower and breaks file permissions, which surfaces later in Docker.
- Verify the GitHub host fingerprint on first connect instead of typing
  "yes" blind. GitHub publishes the current fingerprints in their docs.
- A heredoc looks frozen at a `>` prompt until the closing delimiter line
  is entered. Nested code fences inside one will break the paste.
- Passwords and passphrases in Linux echo nothing at all while typing.
  That is normal, not a stuck terminal.

## Branches

    git branch                       # list local branches, * marks the current one
    git branch -a                    # include remote-tracking branches
    git switch -c <name>             # create a branch and switch to it
    git switch main                  # switch back
    git merge <name>                 # merge <name> into the current branch
    git branch -d <name>             # delete a merged branch (refuses if unmerged)
    git branch -D <name>             # delete regardless, losing unmerged commits

A branch is a pointer to a commit, not a copy of the project. Deleting a
branch removes the label; the commits stay in history.

## Merge conflicts

Git stops when both branches changed the same lines, and writes both
versions into the file with markers:

    <<<<<<< HEAD
    version in the current branch
    =======
    version in the branch being merged
    >>>>>>> other-branch

Resolution is manual: edit the file into its final form and delete all
three marker lines. Then:

    git add <file>     # means "conflict resolved, this is the result"
    git commit         # records the merge commit

Escape hatch, returns everything to the pre-merge state:

    git merge --abort

## Local vs remote

The local repository is complete on its own. Nothing reaches GitHub until
push is run explicitly.

    git push                         # send commits to the tracked remote branch
    git push origin main             # full form: where, and what
    git push -u origin <new-branch>  # first push of a new branch; -u sets tracking
    git pull                         # fetch remote commits and merge them in
    git push origin --delete <name>  # delete a branch on the remote

"origin" is just an alias for the remote URL, set automatically by clone.

A rejected push usually means the remote has commits that are missing
locally. Run pull first, resolve anything that conflicts, then push.

## Reading the push output

    c290506..cca0698  main -> main

Old commit, new commit, which local branch went to which remote branch.
