---
tags: [cmdline]
---
[s/progit]: sources/progit.md
[git_re_cmds]: git_re_cmds.md
[git_internals]: git_internals.md

# Git's three trees

Working with git revolves around modifying three 'trees'(directory-like, not the
data structure):

1. the `HEAD`,
2. the index,
3. the working directory.

Keep in mind this a *simplification*. You can reed more about how `HEAD` and
index look like in [git internals][git_internals].

`HEAD` points to the current branch which in turn points to last commit. You can
think of `HEAD` as a storage of your most up-to-date committed content.

Index, on the other way, stores the content which is about to be committed. The
proposal for the next `HEAD` -- the next committed content. Index is also known
as the *staging area*.

Working tree is the contents of your filesystem. The proposal for the proposal.

## Workflow

You move content from `HEAD` to working tree with `git checkout`. By `git add`
you add that content to index. And finally by `git commit` you create a commit
to reflect the state of your index.

You can also move in the other direction -- from `HEAD` to index and from index
to working tree -- with [git reset][git_re_cmds].

Source: [Pro Git][s/progit].

