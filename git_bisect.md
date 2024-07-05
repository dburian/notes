# Git bisect


Git bisect is useful to find a commit that introduced a specific change. You
supply a 'good' commit where the change is alredy present, and a 'bad' commit
where the change is not yet implemented. `git bisect` then does **binary
search**, each time asking you, if the change is present or not (if the commit
is 'good' or 'bad').

I haven't used it yet, but should try to. More in
[man-pages](https://git-scm.com/docs/git-bisect).
