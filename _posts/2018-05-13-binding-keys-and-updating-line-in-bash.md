---
layout: post
title: Keybinding and updating terminal text line in bash
description: "See some cool Keybindings and how to change terminal text from a script"
modified: 2018-05-13
tags: [bash, shell, keybinding]
image:
    feature:
    credit:
    creditlink:
feature:
credit:
creditlink:
---

tl;dr
1) `bind -x '"<some character>":some-function'`; 2) change the `READLINE_LINE`/`READLINE_POINT` variables in the script


## Keybinding

This is a really cool feature that you can use to create/change the behaviour of keys in your terminal.

### Before we proceed

Keep in mind that you can set the keybindings in your `~/.inputrc` or `~/.bashrc` (or any other file that is sourced on start).

Here we have two examples of setting the `arrou up` to get the previous command that matches the initial text written so far (kind of `Ctrl+r`). Just  note that `history-search-backward` is a builtin function.

{% highlight bash %}
# ~/.inputrc
"\e[A":history-search-backward
{% endhighlight %}

{% highlight bash %}
# ~/.bashrc
bind -x '"\e[A":history-search-backward'
{% endhighlight %}

Here is a simple example using a custom function

{% highlight bash %}
# ~/.bashrc
__clear_screen__() {
  # clear all the content of current screen (same as reopen termial)
  clear && printf '\e[3J'
}

# "C-l" means `Ctrl+l`
# you can see more options/the syntax at https://www.computerhope.com/unix/bash/bind.htm
bind -x '"\C-l":__clear_screen__'
{% endhighlight %}


## The `READLINE_LINE` and `READLINE_POINT` variables

Those variables just store the current text and the cursor position. Here is a simple example of dealing with them:

{% highlight bash %}
__print_letter_and_write_it__() {
  # just prints a
  echo $1

  # changes the content of input text
  READLINE_LINE="$READLINE_LINE$1"

  # puts the cursor after the letter `a`
  # without this, the cusrsor would end after the letter `a`
  # if you are not familiar with `${#READLINE_LINE}`, it just returns the length of the `READLINE_LINE` variable
  READLINE_POINT=${#READLINE_LINE}
}

# call `__print_letter_and_write_it__` with `a` as argument
bind -x '"a":__print_letter_and_write_it__ a'
{% endhighlight %}


[Here](https://github.com/paulojean/suggestions.bash) you can see another example of dealing with binding keys and updating the input text, where I start [fzf](https://github.com/junegunn/fzf) and update the text based on the user's action.
