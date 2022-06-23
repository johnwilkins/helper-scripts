# Helper Scripts
To make working with `git` a little less challenging, I use the following scripts in my day-to-day workflow. Generally, they live in `~/.local/bin` so that they are in my `$PATH` and I can execute them without `./`. 

## Clone OpenShift Docs
My `clone-openshift-docs` script is my most infrequently used script. It does what it says--it clones the OCP repo again. Every now and then, if the index gets corrupted and I don't want to spend the time figuring out how to fix it, I just save my work from within the repo to a directory outside the repo, delete the repo and run this script to reclone the repo. It uses a standard `git clone` command, but it also sets the upstream. If you have `gh`, that process will set the upstream for you. However, this script also sets the `git pull` command to use rebase rather than merge or fast forward. To use this script, make sure you change my user name in the script. Also, my OCP repo lives under a directory called `ocp-repos`, which is from the time I had multiple repos for both upstream and downstream documentation. You can also remove the lines pertaining to `cd ~/ocp-docs` and `cd -`. It's not that useful, since I use it so infrequently.

## Rebase the `main` branch to the upstream repo
My `rebase-ocp-main` script is probably my most frequently used script. To avoid subsequent merge conflicts when creating a working branch, I *ALWAYS* rebase the `main` branch first. Although I don't use it this way anymore, I wrote the script so that it could run on a `cron` job such that it will stash work, change from a working branch to the `main` branch, rebase the repository, change back to the working branch (if any) and pop any stashed changes. If your OpenShift docs repo doesn't live in `~/ocp-repos`, you must modify line 3 of the script to reflect the location of your `openshift-docs` repo. This is a script that I cannot live without, because I don't want to remember to execute all the steps in the process of rebasing, since I do it so frequently.

Within the `openshift-docs` repo, simply execute: 

```bash
$ rebase-ocp-main
```

## Create a working branch
My `create-branch` script is a moderately helpful script. Creating a `git` branch is fairly simple. However, you must be in the repository. Also, when creating a new working branch, you want to ensure that you are creating the branch off of the `main` branch. Lastly, when creating a branch with `git checkout -b <branch_name>`, I often forget to set the upstream origin so that it's tracking my remote.

To streamline my workflow, this script ensures I'm in a `git` repository, it will ensure I'm on the `main` branch, and then it will prompt me for the JIRA ticket that corresponds to the branch. By naming the branch after the JIRA ticket, it makes it easy for me to know which branch corresponds to which JIRA ticket (or BZ number). Then, the script sets the upstream origin so that I don't forget to do that, because when you rebase a working branch `git` wants to know the name of the remote origin branch. 

Within the `openshift-docs` repo, simply execute: 

```bash
$ create-branch
```

## Rebase the working branch to upstream `main`
Some people always rebase the working branch with upstream `main` before creating a pull request. I only rebase to upstream `main` if there is an apparent merge conflict, because it's just a series of unneeded steps if there is no merge conflict. This script will get the current branch and rebase it to upstream `main`. It's essentially the same as the `rebase-ocp-main` script, except that it assumes you are on a working branch. It is possible for this script to exit with an error on the rebase step if there is a merge conflict. In that case, you must resolve the merge conflict manually, push the changes manually, and pop any stashed changes manually. When that happens, I usually check the script to remember what I need to do after addressing a merge conflict. So the script also serves as my notes.

Within the `openshift-docs` repo, simply execute: 

```bash
$ rebase-wb-main
```

## Create an enterprise branch
From time to time, you may need to create a branch for a particular release of OpenShift. Since you cannot modify release branches such as `enterprise-4.x`, you typically must rebase the enterprise branch before creating a working branch off of it. The `create-enterprise-branch` prompts you for the release, such as `4.10`, prompts you for the JIRA ticket, and then it will stash any unstaged changes, checkout the enterprise branch, fetch the upstream version of the enterprise branch , rebase the enterprise branch, checkout a new working branch using the JIRA ticket and release number as the branch name, set the upstream origin branch name and pop any stashed changes. 

Within the `openshift-docs` repo, simply execute: 

```bash
$ create-enterprise-branch
```

## Create a pull request
I find creating pull requests manually to be rather tedious. The OCP team wants PRs in a particular format:
* They wayt the JIRA ticket, and it's title in the PR title. 
* They want a commit message.
* They want a link to the JIRA ticket or BZ in the PR. 
* They want the release version(s) that the PR corresponds to. 
* They want a Preview URL.
* They want you to sign-off the PR.  
Additionally, Sheldon wants to have the PR URL in the JIRA ticket. All of these practices make perfect sense, but the process is tedious. So my `create-ds-pr` script (the `ds` means 'downstream' and is an artifact when I did both upstream and downstream work simultaneously) automates most of the process for you. 

Since you won't be executing this on the `main` branch, the script ensures you are on a working branch. It prompts you for the JIRA ticket. It prompts you for a commit message. It prompts you for the release(s) number. Then, it creates the PR by contacting JIRA and retrieving the JIRA ticket title and retrieving any pre-existing PR URLs, formats the PR, creates the PR, and once created, it puts the PR URL back into the JIRA ticket automatically for you. To use this script, you must create a JIRA token (with a lenghty expiration, so you don't have to change it constantly). See lines 23-25 for instructions.

Within the `openshift-docs` repo, on a working branch, with all changes commited and pushed, simply execute: 

```bash
$ create-ds-pr
```

**NOTE:** I don't use this script for release notes, because at the time of writing the script, the OCP team had not made the `BASE_BRANCH` variable modifiable, so creating a PR that will merge to `main` ends up creating a PR with merge conflicts and a bazillion commits--which is anxiety inducing. Hopefully, that will change in the future. 

This is another script that I cannot live without. It says me lots of time and frustration.


## Squash Commits
The OCP team does not want your commit history so that they can keep the upstream `git` log clean and simple. It makes sense, but squashing commits interactively is tedious and for me a bit anxiety inducing. I have accidentally squashed more commits than I should have before, and this script automates most of the task, eliminates some anxiety and generally gets the job done quickly.

Squashing commits is playing with power tools. So the script ensures that you are not on the `main` branch and inadvertantly squashing commits on `main`. It also does a pull in case you accepted some commit suggestions on a PR. This script works with standard working branch that merge to the upstream `main` branch, and it works with enterprise branches for a particular release. So the script will prompt you for the name of the base branch, which is usually `main`. The script also ensures that you have two or more commits so that you don't try to squash when you have no commits to squash.

The script does have a known flaw, which I have not resolved yet: when you have to rebase a PR, it typically gets the commit count wrong--usually many more commits than you have to squash. So the script will ALWAYS ask you if you want to squash a certain number of commits before it actually does it. If the commit count is higher than the number of commits on the PR page, simply press *N* to abort the procedure. Alternatively, the script may prompt you how many commits you want to squash if it suspects the number is too high. The peer review squad likes a commit message on squashed commits, typically a message similar to the commit message in the PR (less the other things like sign-off, preview URLs, etc.).

Within the `openshift-docs` repo, on a working branch, with all changes commited and pushed, and more than 1 commit in the PR, simply execute: 

```bash
$ squash-commits
```
