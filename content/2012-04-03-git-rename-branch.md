 git - rename branch (local and remote)
=======================================

    #rename local branch
    git branch -m old-branch-name new-branch-name

    #delete remote branch with old name
    git push origin :old-branch-name

    # create remote renamed branch
    git push origin new-branch-name

Links
--------
[stackoverflow](http://stackoverflow.com/questions/1526794/git-rename-remote-branch)