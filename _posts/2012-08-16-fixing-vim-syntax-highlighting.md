---
layout: article
category: articles
title: Fixing Vim Syntax Highlighting
author: mattf
---

Vim is a great text editor. However sometimes when editing large files, the syntax coloring can become broken. It is possible to fix this in a few different ways.

The underlying problem comes from the default behaviour of going backwards in the file from the current position and calculating the syntax highlighting from there. If the syntax check starts at the wrong place then it can incorrectly highlight sections.

An easy way to fix this is to explicitly tell vim to always start from the beginning of the file. While this will always produce correct syntax, it can be slow on very large files. If you wish to do this, then add the following line to your `.vimrc` file:

    autocmd BufEnter * :syntax sync fromstart

This can be a bit overkill though. Most of the time the syntax highlighting is correct, and you just need to fix it the few times that it breaks. If you want to be able to quickly fix it, then you can issue the following command in vim:

    :syntax sync fromstart

You can also bind this to a key so that you can quickly correct the syntax highlighting when it becomes an issue:

    noremap <F12> <Esc>:syntax sync fromstart<CR>
    inoremap <F12> <C-o>:syntax sync fromstart<CR>

This would bind the F12 key to fixing the syntax highlighting.

You can find more tips and advice regarding fixing syntax highlighting in [the vim documentation](http://vim.wikia.com/wiki/Fix_syntax_highlighting).