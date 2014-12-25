Git - Your branch and 'origin/xxx' have diverged
============================================

Git error (not after rebase, see below):
--------------------------------------------

    Your branch and 'origin/xxx' have diverged,
    and have 1 and 1 different commit(s) each, respectively.

Error is caused by two independent commits - one (or more) on the local branch copy and other - on the remote branch copy (for example, commit by another person to the same branch)

History looks like:

    ... o ---- o ---- A ---- B  origin/branch_xxx (upstream work)
                    \
                        C  branch_xxx (your work)

Most easy way to solve - is to rebase commit C to the remote state:

$ git rebase origin/branch_xxx
The history will look like this:

    ... o ---- o ---- A ---- B  origin/branch_xxx (upstream work)
                            \
                            C'  branch_xxx (your work)

The same git error after rebase:
--------------------------------------------
Your branch and 'origin/xxx' have diverged,
and have 1 and 1 different commit(s) each, respectively.
In this case this is normal and expected, your local branch is different then origin version because of rebase.

For example, we have a history like this:

    ... o ---- o ---- A ---- B  master, origin/master
                    \
                        C  branch_xxx, origin/branch_xxx

Now we want to rebase branch_xxx against the master branch:

    $ git checkout branch_xxx
    $ git rebase master

And we get "Your branch and 'origin/branch_xxx' have diverged" because the history now is this:

    ... o ---- o ---- A ---------------------- B  master, origin/master
                    \                        \
                        C  origin/branch_xxx     C' branch_xxx

If you absolutely sure this is your case then you can force Git to push your changes:

    $(think twice before this) git push origin branch_xxx -f

Links
--------------------------------------------

[Stackoverflow - master branch and 'origin/master' have diverged, how to 'undiverge' branches'?](http://stackoverflow.com/questions/2452226/master-branch-and-origin-master-have-diverged-how-to-undiverge-branches)

[Stackoverflow - git rebase basics](http://stackoverflow.com/questions/11563319/git-rebase-basics)
