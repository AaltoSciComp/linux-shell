Instructor's guide
==================

Setting up instructions for the lecturer
----------------------------------------

Main terminal white&black with the enlarged font size. One small terminal at the top that shows
commands to the learners.

 - ``export PROMPT_COMMAND='history -a'``   # .bashrc or all the terminals one launches commands
 - ``tail -n 0 -F .bash_history``

Alternatively, ``script`` allows to follow the session even after sshing to a remote host
plus command appear as soon as they are run. The
regular expression can be adapted to the lecturer's PS1, this one assumes *]$ command*.

 - ``script -f demos.out``   # action window
 - ``tail -n 1 -f demos.out | while read line; do [[ "$line" =~ \]\$\ ([^ ].+)$ ]] && echo ${BASH_REMATCH[1]}; done``
