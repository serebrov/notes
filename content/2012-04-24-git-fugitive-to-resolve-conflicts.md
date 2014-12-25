git - use vim with fugitive to resolve merge conflicts
=======================================

To use fugitive as you mergetool you can use the following.

    git config --global mergetool.fugitive.cmd 'vim -f -c "Gdiff" "$MERGED"'
    git config --global merge.tool fugitive

Links
--------
[stackoverflow](http://stackoverflow.com/questions/7309707/my-git-mergetool-open-4not-3-windows-in-vimdiff)
[fugitive screencast](http://vimcasts.org/episodes/fugitive-vim-resolving-merge-conflicts-with-vimdiff/)