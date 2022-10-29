[s/progit]: sources/progit.md
[git_three_trees]: git_three_trees.md

# Git reset, checkout, revert and restore

All commands sound similarly and I constantly googled how they are different. So
this is a little summarization starting with `git reset`.

To understand `git reset` you should get the hang of the [three git
trees][git_three_trees].

## Git reset

You can run `git reset` in three levels of 'seriousness', first level being the
default.

1. `--soft` - update branch under `HEAD`
2. `--mixed` - update branch under `HEAD` and index
3. `--hard` - update branch under `HEAD`, index and working directory

`git reset` is always run with a commit hash (`HEAD` being the implicit). So
running `git reset --hard <sha>` while on branch `dev` will:

1. move branch dev to `<sha>`,
2. rewrite index with the contents of `<sha>`,
3. rewrite working directory with the contents of `<sha>`.

`git reset` with a specified path limits the behaviour to the given path and
skips the first step (updating branch under `HEAD`).

## Git checkout

`git checkout` is similar to `git reset --hard` but differs at two main points:

1. Without given path `git checkout` would check for overwrites, if there are
   any, it prompts you and fails. Meaning you cannot accidentally delete
   something (*without* `path` *given*).
2. `git checkout` only moves the `HEAD` itself, not the branch it is pointing
   to.

Be wary, however, of running `git checkout <path>` -- it is not working
directory safe and git checkout will not ask you for permission to overwrite any
file.

## Git revert

`git revert` may sound that it also discards the current state in favor of a
state in the past. In reality `git revert <sha>` introduces new commit so that
`HEAD`, index and working directory will be the same as in `<sha>`. It does not
delete the previous commits, it only creates a new one.

With flags it can only modify the index and working tree (i.e. does not create a
commit).

## Git restore

`git restore` restores files from a *source* in either:

- in the working directory (`--worktree`) -- the default -- or
- in the index (`--staged`).

A source can be either a commit or index (only when restoring files in the
working directory). `git restore` must be called with a path specified.

So `git restore` somewhat overlaps with `git reset` however there are notable
differences:

- `git revert` does not update references,
- `git revert` works only with paths,
- `git revert` can update only the working directory without influencing index.


---

Sources:
- Chapter "Reset demystified" of [Pro Git][s/progit].
- `man git revert`
- See "Reset, restore and revert" in `man 1 git`.
