git - Your branch is ahead of 'origin/master' by 1 commit after pull
===========================================

Your branch is ahead of 'origin/master' by 1 commit (or X commits) after `git pull origin master`.

The sequence:
* Have up-to-date repository
* There is a change in the origin/master
* Do `git pull origin master`
* Change is received and merged
* git status shows “Your branch is ahead of 'origin/master' by 1 commit.”

The reason is because during “pull origin master” reference to the remote 
origin/master is not changed (still points to older version).

To check this:

    git diff origin/master

It will show the difference - it should be the last master change.

To fix this exacute:

    git fetch origin

This will update git links to remotes.

If you have this situation then you probably wanted to do just `git pull` instead of `git pull origin master`.

Links
-------------------------------------------

[Stackoverflow: 'git pull origin mybranch' leaves local mybranch N commits ahead of origin. Why?](http://stackoverflow.com/questions/1741143/git-git-pull-origin-mybranch-leaves-local-mybranch-n-commits-ahead-of-origin)

[Git repo on a machine with zero commits is ahead of remote by 103 commits.. !](http://git.661346.n2.nabble.com/Git-repo-on-a-machine-with-zero-commits-is-ahead-of-remote-by-103-commits-td5957671.html)

[Quora: Git (revision control): What are the differences between "git pull", "git pull origin master", and "git pull origin/master"?](http://www.quora.com/What-are-the-differences-between-git-pull-git-pull-origin-master-and-git-pull-origin-master)

[Stackoverflow: Differences between git pull origin master & git pull origin/master](http://stackoverflow.com/questions/2883840/differences-between-git-pull-origin-master-git-pull-origin-master)

[Stackoverflow: Why do I have to push the changes I just pulled from origin in Git?](http://stackoverflow.com/questions/5283829/why-do-i-have-to-push-the-changes-i-just-pulled-from-origin-in-git)