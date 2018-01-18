---
layout: post
title: Writing Vim plugin in VimL
description: "Learn the first steps to write Vim plugins using VimL"
modified: 2018-01-18
tags: [vim, viml, vimscript]
image:
    feature:
    credit:
    creditlink:
feature:
credit:
creditlink:
---

tl;dr
In this post I'll explain the basics of writing a vim plugin, talking about structure and some useful commands.


## Getting started

The firs thing to know is that to write a plugin you only need a project with a folder called `plugin` and some files with the `.vim` extension inside it (on the startup, vim will load those files).

Another, important, thing to know is that a vim plugin is just a bunch of vim commands (this may sound obvious but, 'cause of ~reasons~, I allways thought that a plugin was some kind of magic spells).
For example, if you want to write a plugin that strip trailing whitespaces, you need the following structure

{% highlight bash %}
.
+── plugin
    +── awesome.vim
{% endhighlight %}

and, inside `awesome.vim`

{% highlight vim %}
function! StripWhiteSpaces()
  %s/\s\+$//g
endfunction
{% endhighlight %}

and just run `:call StripWhiteSpaces()` (or map to something else) and you're done! (ps: this simple function has some problems, such as: 1) after you execute it will move your cursor to the last line that was modified; 2) if there is no trailing whitespace it will throw an error, because we'are trying to replace a pattern that does not exist. But it is enought to get the idea).

You also can add a folde called `doc` (in the same level as `plugin`) and add a `awesome.txt` there, so people can read the help from vim.

## Reading a more complete example

Now lets look at another function that do more stuffs (this example is part of the [sort-quire.vim](https://github.com/paulojean/sort-quire.vim)'s plugin):

{% highlight vim %}
function! s:SortQuire_sort_clojure()
  " Store the default registry's value
  let l:current_register = @"

  " Execute the command in normal mode, same thing as you would do if you wanted to go the first
  " occurence of the word `require`.
  " The bang in `normal!` is important: http://learnvimscriptthehardway.stevelosh.com/chapters/29.html
  execute "normal! gg/require\<cr>"

  " again, just evaluating vim commands (in this case to copy the section of the current line)
  normal! 0wy%

  " Getting the value from the default registry.
  " In this case the value is the block yanked in the previous commmand
  let l:require_block = @0

  " getting the substring, here the idea is to get only the requires (in the form of `[lib :as a]`)
  let reqs = strpart(l:require_block, 10, strlen(l:require_block) - 10)

  " just a split, as in many languages
  let arr_requires = split(reqs, "  ")

  " From inside:
  " Copy the previous array, given that filter/map changes the array, in this case it is not
  " really required to copy.
  " Inside filter/map use `v:val` to refer to each element value
  let requires = sort(map(filter(copy(arr_requires), 'v:val != ""'), 'strpart(v:val, 0, strlen(v:val) - 1)'))

  " Use `call` when you need to invoke another function
  call Replace_Requires(requires)

  execute "normal! gg/require\<cr>"
  normal! 0wy%
  execute "normal! \<c-v>\<s-%>="

  normal! gg

  " Check the existence of the pattern before trying to remove it, so no error is shown to the user
  if search(')\s\+)')
    %s/)\s\+)/))/g
  endif

  " Restore the content of the default registry to what it was before running the script
  call setreg('"', l:current_register)
endfunction

function! Replace_Requires(requires)
  " Call `execute` and interpolate command with variable contetnt
  execute "normal! d%i(:require " . get(a:requires, 0) . "\r"
  call SortQuire_sort_clojure_fill(a:requires)
endfunction

function! SortQuire_sort_clojure_fill(items)
  normal! k
  let index = 1
  let items_length = len(a:items) - 1
  while index < items_length
    " Write var content on the line bellow cursor
    put = get(a:items, index)
    let index += 1
  endwhile
  normal! j
  execute "normal! i" . get(a:items, index) . ")"
endfunction
{% endhighlight %}

The `s:` and `l:` refers to the scope of the function and variables, I wont talk about them here (I
put them there just to tell about their existence), but you can read about scope [here](https://www.gilesorr.com/blog/vim-variable-scope.html).

You need to define the function with a bang in the end (`function!`) to replace if it alreaday exists. This may sound strange but, otherwise, everythime (but the first) that this file gets sourced you will get an error, because vim will try to create the function (so it's a good idea to avoid creating plugin with a name that already exists).


This is a simple example because it deals only with the current file content, to write something that handle external content (eg: write/read from/to another files) you probably won't want to do that in VimL, you will be better using something else (python, rust, go etc) and interacting with vim through their client api (just google that:), maybe in the future I write a walkthrough about that.
