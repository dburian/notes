---
tags: [cmdline]
---
# Git Worktrees

Git worktrees allow you to have more working directories and thereby having more
branches checked-out at one time. However, often it comes with some hassle, some
of which can be avoided.

Source: [Using git worktrees in a clean
way](https://morgan.cugerone.com/blog/how-to-use-git-worktree-and-in-a-clean-way/).


## Bare clone

Instead of using the typical clone and having one *main* worktree (also known as
[working directory](./git_three_trees.md)) and other *linked* worktree it is
recommended to checkout bare repo. The reasons are that it is more clearer what
each directory holds. Linked worktrees can be deleted, without any fuss. But
deleting your main worktree which holds the `.git` directory will purge entire
code base (together with unpushed history).

Clone a repo like so:

```bash
# Inside your project folder
git clone --bare git@.... .bare
```

To tell git to use that repo in your project folder, add a link to it:

```bash
echo "gitdir: ./.bare" > .git
```

While bare repositories make the project folder a lot cleaner, they have one
major disadvantage. By adding `--bare` to the clone command, no remote-tracking
branches or related configuration is created. This means that you won't be able
to checkout remote branch. The solution is to add this configuration yourself:

```bash
git config remote.origin.fetch "+refs/heads/*:refs/remotes/origin/*"
```

## Add worktrees

All worktree-related operations are done with `git worktree`

## Creating bare repo

Creating bare repo is not really something I'd advise. You basically create just
the scaffolding, but have no way to "populate it" with branches. In order to do
that you have to clone it somewhere else in your system to a non-bare repo and
basically do a loop:

1. Init bare repo
```bash
mkdir -p CoolRepo/.bare
cd CoolRepo/.bare
git init --bare
```

2. Clone it in a newly initialized non-bare repo on your system
```bash
cd ../../
mkdir CoolRepoClone
cd CoolRepoClone
git init .

git clone ../CoolRepo/.bare
```
3. Do a commit in non-bare repo
```bash
touch README.md
git add README.md
git c -m "Initial commit"
```
4. Push the commit into the bare repo
```bash
git push ../CoolRepo/.bare main
```

5. Now you can create a worktree in your bare repo
```bash
cd ../CoolRepo
git worktree add main
```




