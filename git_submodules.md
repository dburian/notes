[s/progit]: sources/progit.md

# Git submodules

Git submodule is a convenient and de-fact *the* way of storing a git repo inside
another git repo.

#### Without submodules

If you stored a repository the straightforward way, you'd have to git-ignore the
`.git` directory (otherwise the commit diff would be huge and unreadable) and
even so there is no metadata as to which repository it actually is. This is
desired only in minority of cases.

## Git submodules

With a repo 'S' as a submodule of repo 'R' the current commit of 'S' is kept in
the history of repo 'R' -- not the whole content of 'R'. So if you'd look up
what object git uses for storing the submodule (subdirectory) using [plumbing
command][git_internals] you'd find a `commit` object instead of the usual `tree`
object.

The submodule's `.git` folder is stored inside the containing repo's `.git`
folder under `.git/modules/<path-to-module>`.

Each of submodule's meta-information is stored in `.gitmodules` in the root of
containing repo. For each submodule we must have:

- `path` - relative path to submodule from root of the containing repo,
- `url` - url to fetch the submodule's repo from
- `name` - identifier; if not specified `path` is used

`.gitmodules` is kept in history same as `.gitignore`. For public repositories
the fetch url should be publicly available -- so no ssh. You can override the
url for your setup using `git config`:

```bash
git config submodule.<path to submodule>.url <private url>
```

### Creating

You can create a submodule by executing:

```bash
git submodule add <url to repo> <path to submodule>
```

As with `git clone` the path is purely optional if you want the submodule's root
directory named differently or be somewhere else.

Keep in mind the path to the submodule should not be already in the index,
otherwise git is going to complain.

### Cloning repo with submodule

In case you clone a repo with submodules you are in for a surprise because there
will be nothing there. To get the submodules, there are two things you have to
do:

1. initialize the submodules - somewhat prepare git that the submodule is going
   to be used with `git submodule init` (optionally appending path to limit
   which submodules get initialized).

2. update the submodules - update the code in *initialized* submodule to match
   the current commit with `git submodule update` (again can be given a path to
   filter).

You can do both steps with `git submodule update --init` or even clone the
containing repo with `git clone --recurse-submodules`.


### Deleting a submodule

To remove a submodule use the old `git rm`.

Do not mistake `git submodule deinit` for a 'remove this submodule' command. It
simply deletes the submodule's content from your working tree.


Source: Chapter "Git submodules" of [Pro Git][s/progit].
