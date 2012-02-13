Git - Your branch and 'origin/xxx' have diverged
============================================

Git error:

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

Links
-----------------
[Stackoverflow](http://stackoverflow.com/questions/2452226/master-branch-and-origin-master-have-diverged-how-to-undiverge-branches)



