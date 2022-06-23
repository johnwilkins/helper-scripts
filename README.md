# Helper Scripts
To make working with `git` a little less challenging, I use the following scripts in my day-to-day workflow. Generally, they live in `~/.local/bin` so that they are in my `$PATH` and I can execute them without `./`. 

## Clone OpenShift Docs
My `clone-openshift-docs` script is my most infrequently used script. It does what it says--it clones the OCP repo again. Every now and then, if the index gets corrupted and I don't want to spend the time figuring out how to fix it, I just save my work from within the repo to a directory outside the repo, delete the repo and run this script to reclone the repo. It uses a standard `git clone` command, but it also sets the upstream. If you have `gh`, that process will set the upstream for you. However, this script also sets the `git pull` command to use rebase rather than merge or fast forward. To use this script, make sure you change my user name in the script. Also, my OCP repo lives under a directory called `ocp-repos`, which is from the time I had multiple repos for both upstream and downstream documentation. You can also remove the lines pertaining to `cd ~/ocp-docs` and `cd -`. It's not that useful, since I use it so infrequently.

## Rebase OCP Main
My `rebase-ocp-main` script is probably my most frequently used script. To avoid merge conflicts when creating a working branch, I ALWAYS rebase the main branch first. Although I don't use it this way anymore, I wrote the script so that it could run on a `cron` job such that it will stash work, change from a working branch to the `main` branch, rebase the repository, change back to the working branch (if any) and pop any stashed changes. If your OpenShift docs repo doesn't live in `~/ocp-repos`, you must modify line 3 of the script to reflect the location of your `openshift-docs` repo. This is a script that I cannot live without, because I don't want to remember to execute all the steps in the process of rebasing, since I do it so frequently.

Within the `openshift-docs` repo, simpley execute: 

---
$ rebase-ocp-main
---
