[s/progit]: sources/progit.md

# Git internals

Git comes with two sets of commands:

- porcelain cmds - high-level user friendly commands centered around version
  control
- plumbing cmds - low-level commands responsible for managing the filesystem
  as content database.

These two sets of commands create two layers of git. On the top we have
user-friendly Version Control System (VCS) layer which is built on the bottom
layer, which stores, looks-up content in the filesystem.

Plumbing commands are the interesting ones. They need to manage (mainly):

- objects (the "content")
- references (the named pointers to some version of the "content")

## Objects

There are several types of objects and all are stored under their hash in
`.git/objects/<first two chars from hash>/<rest of the hash>`.

You can examine objects by using git plumbing command `cat-file`:

```bash
$ git cat-file -p a906cb2a4a904a152e80877d4088654daad0c859

# This is a readme

...
```

The `-p` stands on pretty print based on the type of object.

### Blobs and tree objects

Each file stored in git is saved as a blob. The blobs do not include filenames
or rights of the stored files. That information is stored in tree objects - list
of files or tree objects each with:

1. mode - can specify normal file, executable, symbolic link, ...
2. type - 'blob' or 'tree'
3. hash - hash of the object
4. name - filename or directory name

Example:
```
100644 blob a906cb2a4a904a152e80877d4088654daad0c859 README
100644 blob 8f94139338f9404f26296befa88755fc2598c289 Rakefile
040000 tree 99f1a6d12cb4b6f19c8655fca46c3ecf317074e0 lib
```

So to store a directory 'A' with subdirectory 'B' you'd have to store 'B' as
a tree and then store that tree as a record of the 'A' tree.

Tree objects always contain the whole snapshot of the stored content. If the
content hasn't changed from the previous commit, the tree object stores the old
hash. Which brings us to...

### Commit objects

When you commit you create another type of object - commit object. Commit
objects store

1. the author of the commit,
2. the committer,
3. hash of the tree committed,
4. hashes of parent commits and
5. the commit message.

### Tag objects

As you've probably already guessed annotated tags are another type of object.
Lightweight tags are simply references (described below).

### How it fits together

The resulting structure is:

- files stored as blobs whose hashes are stored as
- entries in the tree objects whose hashes are stored in
- git commits whose hashes are stored in children commits.

## References

Objects can store a snapshot of the content. You could easily retrieve it (with
the help of plumbing commands), but you'd have to remember the hash of the
commit you want to retrieve. That is the reason references exist - they are
simply named hashes (and yes you can have references to blobs or tree-objects).
References are stored in `.git/refs`.

`HEAD` is simply reference to a reference - a file with name of the reference
which stores the hash.

Remotes are also references but they're read-only. They store the hash of the
last pushed commit to the given remote and branch.


## Packfiles

As you probably noticed two largely similar files would be stored as two blobs
taking up unnecessary space (since the contents of the files is largely the
same). This is solved by garbage-collect command `git gc`. `git gc` goes through
the objects and packs the 'unused' ones into a packfile in `.git/objects/pack`.
Next to the packfile there will be an index listing the contents of the
packfile. When packing the files git can store a diff instead of the whole file
to save up space.

`git gc` command is run automatically when pushing to a server so you don't have
to run it manually.

Source: Chapter "Git Internals" of [Pro Git][s/progit].

