git - submodule helpers
=========================
Below are some git commands which can be useful to resolve problems with submodules.

Get a list of commits inside a submodule
-------------------------------------------
Submodules are identified by SHA-1 hashes so you may need to get a list of them. Do the following inside the submodule folder (or inside the separate submodule repository):

    $ git log --oneline

    5bd722f commit 5
    b5c1524 commit 4
    2444bfa commit 3
    e0eadd5 commit 2
    c180c5a commit 1
    702fc8a commit 0

View current submodule commit
-------------------------------------------

    $ git submodule status

    5bd722fa26dcdd64128392aa28e08849fe37f111 sub (heads/master)

Compare a submodule state with another branch
-------------------------------------------
Assume we are on the "branch1" and want to compare 'sub' submodule state with master:

    $ git diff master -- sub

    diff --git a/sub b/sub
    index 702fc8a..5bd722f 160000
    --- a/sub
    +++ b/sub
    @@ -1 +1 @@
    -Subproject commit 702fc8a7edcdf5ceba9929958bd6cd7f000eb369
    +Subproject commit 5bd722fa26dcdd64128392aa28e08849fe37f111

Where "-Subproject commit" is another (master) branch state and "+Subproject commit" is a current branch state.
VIEW CHANGES TO SUBMODULE STATE ON THE CURRENT BRANCH
What we do is show a git log with diffs for submodule (named 'sub' in this case):

    $ git log -p sub

    commit 261aa4bdb6944c77fc98e52748d656e56969ba6a
    Author: name
    Date:   Thu Sep 13 15:20:48 2012 +0300
    accidental sub switch to commit 5
    diff --git a/sub b/sub
    index b5c1524..5bd722f 160000
    --- a/sub
    +++ b/sub
    @@ -1 +1 @@
    -Subproject commit b5c15241fe40f122d5225e5c76457802de3ad605
    +Subproject commit 5bd722fa26dcdd64128392aa28e08849fe37f111

    commit 073aaf160706d705c6c95ab21327388cd6683486
    Author: name
    Date:   Thu Sep 13 14:58:41 2012 +0300
    sub commit 4
    diff --git a/sub b/sub
    index 2444bfa..b5c1524 160000
    --- a/sub
    +++ b/sub
    @@ -1 +1 @@
    -Subproject commit 2444bfa760b5d3cc974658bd18482f9679f52b69
    +Subproject commit b5c15241fe40f122d5225e5c76457802de3ad605

    ...

Where "-Subproject commit" is a previous submodule state and "+Subproject commit" is a new submodule state.

Links
-------------------------------------------
[GitSubmoduleTutorial](https://git.wiki.kernel.org/index.php/GitSubmoduleTutorial)

[Pro Git: 6.6 Git Tools - Submodules](http://git-scm.com/book/en/Git-Tools-Submodules)

[Stackoverflow: git submodule update](http://stackoverflow.com/questions/1979167/git-submodule-update)

[Understanding Git Submodules](http://speirs.org/blog/2009/5/11/understanding-git-submodules.html)

[git Submodules Explained](http://longair.net/blog/2010/06/02/git-submodules-explained/)
