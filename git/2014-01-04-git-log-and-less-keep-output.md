How to keep git log and less output on the screen
============================================

When `git` uses `less` as pager the output of commands like `git log` disappears from the console
screen when you exit from less.
This is not convenient in many cases so here is how to fix this.

Just for git commands:

    git config --global --replace-all core.pager "less -iXFR"

For less globally (including git) - add to .bashrc / .zshrc / etc:

    export LESS=-iXFR

The options we set for less are:

    * -i - ignore case when searching (but respect case if search term contains uppercase letters)
    * -X - do not clear screen on exit
    * -F - exit if text is less then one screen long
    * -R - was on by default on my system, something related to colors

Links
============================================
[Linux Questions: more or less - but less does not keep its output on the screen](http://www.linuxquestions.org/questions/linux-software-2/more-or-less-but-less-does-not-keep-its-output-on-the-screen-938187/)
