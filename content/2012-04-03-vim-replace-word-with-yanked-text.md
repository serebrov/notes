vim - replace a word with yanked text
======================================

Copy a word and paste it over other words:

    yiw     yank inner word (copy word under cursor, say "first").
    ...	 Move the cursor to another word (say "second").
    viwp	 select "second", then replace it with "first".
    ...	 Move the cursor to another word (say "third").
    viw"0p	 select "third", then replace it with "first".

Deleting, changing and yanking text copies the affected text to the unnamed register (""). 
Yanking text also copies the text to register 0 ("0). 
So the command yiw copies the current word to "" and to "0.
Typing viw selects the current word, then pressing p pastes the unnamed register over the visual selection. 
The visual selection is deleted to the unnamed register.

Links
------
[vim.wikia.com](http://vim.wikia.com/wiki/Replace_a_word_with_yanked_text)