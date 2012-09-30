git - find all branches where file was changed
===========================================

[Solution from stackoverflow](http://stackoverflow.com/questions/6258440/find-a-git-branch-containing-changes-to-a-given-file).

Find all branches which contain a change to FILENAME (even if before the (non-recorded) branch point):

    git log --all --format=%H FILENAME | while read f; do git branch --contains $f; done | sort -u

Manually inspect:

    gitk --all --date-order -- FILENAME

Find all changes to FILENAME not merged to master:

    git for-each-ref --format="%(refname:short)" refs/heads | grep -v master | while read br; do git cherry master $br | while read x h; do if [ "`git log -n 1 --format=%H $h -- FILENAME`" = "$h" ]; then echo $br; fi; done; done | sort -u