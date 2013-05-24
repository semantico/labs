---
layout: article
category: articles
title: How to :quit Vim
author: mattf
abstract: Everyone loves Vim, but the biggest problem any new user has is everything, including exiting the program. I hope to help newbies out with this tricky problem by showing a way to quit Vim using ZSH Expansion.
---

## TL;DR
This is stupid. This is a really bad idea. This will forcibly kill Vim and you may lose any unsaved changes. All that said, it’s still pretty cool.

ZSH pattern expansion allows you to define a custom matcher. This custom matcher can test potential matches and indicate if they should be returned. It is also possible for the custom matcher to alter the match before returning it, including returning more than one match from a single potential match. This can be used to do useful things like traversing zip files or returning files based on mime type, but here we will be returning a single value which will represent the Vim process.

ZSH pattern expansion is best done on files and folders, and it happens that there is a filesystem that represents processes. Using this filesystem it is possible to locate a folder which clearly indicates the Vim process id. In this way ZSH pattern expansion can identify the process that needs to be killed to exit Vim.

## The /proc filesystem

This will use the `/proc` virtual filesystem to search from the current process up the process tree until it finds the Vim process. The `/proc` folder contains a folder for each running process on the system. The content of the folder contains several different files which provide information about the process in question. Here are some of the more interesting ones:

* `exe`  
  This is a symlink to the actual file that is being executed. This would be a great way to determine the command that a process folder represents, except for the irritating fact that following the symlink requires permission.

* `cmdline`  
  This is a plain text file of the command that was initially executed to start this process. Unlike the exe symlink, this is world readable, so it is a more consistent way to get the underlying process. Since you can execute the same command in many different ways, detecting the command in this way takes a bit more effort.

* `stat`  
  This is a somewhat opaque file that contains many interesting bits of information. The important bit for us is the parent pid, which is field 4. You can read about the entire content of this file by finding the `/proc/[pid]/stat` section of `man 5 proc`.
There are many other files, but the ones above are the important ones for this.

That is not all the `/proc` filesystem provides! One of the best things in there is the `/proc/self` symlink. This symlink is unlike others, in that it always points to the folder representing the process that looks at it. A little example might help:

    % cat /proc/self/cmdline
    cat/proc/self/cmdline

*(the cmdline file is delimited by the null character, hence the output looks like it misses the space)*

It really is quite a neat symlink, all things considered.

## The VIM :shell

Vim has a full shell available to it. As it happens, it will be the same shell that is used to start the Vim command. This means you really have to start Vim using ZSH:

    % zsh -c 'vim'
*(just in case you arn’t using zsh already)*

You can then confirm you are running inside zsh by executing the following ex command:

    :!echo $SHELL
It should produce something like `/bin/zsh`.

## Approach

The approach can be summed up using this little flowchart:

![flowchart](/articles/img/quit-vim-flowchart.png)

It is quite a basic approach, searching all parents till the Vim or Init processes are found. Finding Vim represents success, as it is that process that we want to kill. If we hit Init then we have failed – the Init process is the parent of all processes, so reaching that means we have missed our target.

## ZSH Expansion and Custom Matchers

In my [presentation on ZSH Expansion](http://labs.semantico.com/talks/2013/01/14/zsh-expansion.html), near the end I cover custom matchers which allow arbitrary shell code to be executed to determine if the current pattern is appropriate. This takes it a bit further, writing out a whole file to define a single custom matcher.

Before we start using that, lets go over a few ZSH pattern expansions:

* `/a/symlink(:A)`  
  The :A switch here will expand any relative path to an absolute path, as well as following a symlink to the final file. This will follow any (reasonable) number of symlinks to the final destination. This is used to expand the `/proc/self` symlink to the process folder.

* `/some/file/path(:t)`  
  The `:t` switch here will return the final file name in the path (think of it as the tail of the path). In this example the result would be `path`.

* `=executable`  
  The `=` prefix here will expand the executable name provided to the path for it, using `$PATH` to locate the executable. You can combine this with `:A` to return the destination for any symlinks. For example, `${${:-=vim}:A}` will return the underlying executable for vim.

### Applying flags to plain strings
This is the complicated example given directly above. Fundamentally, the `${VAR:-SUB}` pattern is used – the SUB string is substituted if the VAR is empty. The SUB text can have flags applied to it, such as using `=` to find the path. To force it to always use the SUB string, just use nothing for the VAR, hence `${:-=vim}`.

* `${(s/ /)VAR}`  
  This will split the variable on spaces. The array can be indexed to get a specific value. This can split on many things, but in this code it is useful to split on null characters (for cmdline) and spaces (for stat). Like most of these zsh flags, the delimiter can vary (`#` `:` `‘` etc).

* `/*/*/*(+matcher)`  
This is the core expansion we will use. This passes each potential match as $REPLY to the function called matcher. If the matcher function returns a true value then the value of `$REPLY` is used as a match for the expansion.

It is possible to alter the value of `$REPLY` to adjust the match, even if the result could not match the original pattern. Furthermore a `$reply` variable also exists, which overrides `$REPLY` if it has been set. The lowercase version is able to hold an array, thus allowing multiple matching values to be returned. Complicated!

These are just a small selection of the ways that ZSH can expand strings. As you can see, almost all functions that would require external commands in bash can be done using cryptic ZSH expansion modifiers. This means my little script only needs to use the cat command, to turn a filename into the content. There may even be a ZSH expansion which does that...

## Putting it all together

Lets start by opening the file containing the lovely shell code:

    vim =(curl https://raw.github.com/matthewfranglen/quit/master/quit.txt)

What this does it is takes the output of the curl command and creates a temporary file which Vim then opens. This file actually exists on disk and you can play with it using other commands, however when the original command that uses the file exits, the file will be deleted. It is basically an automatic temporary file.

Now we have vim running we can find the proc folder that relates to the vim process by executing the following vim command:

    :! source %; echo /proc/self(+get_vim_process)

This will produce output something like:

    /proc/7549
    Press ENTER or type command to continue

So here you can see that the Vim process has a pid of `7549`.

The `kill` command will not be happy if we pass a `/proc/ID` path to it, so what we need is to reduce this down to just the pid. Luckily, the `:t` expansion will allow you to return only the last element of the path, which in this case is the pid.

So, finally, the way to quit Vim is:

    :! source %; kill -9 /proc/self(+get_vim_process:t)

CONGRATULATIONS!