---
layout: article
category: articles
title: wmctrl – window controller
author: mattf
---

The wmctrl command can be used to control windows that have been opened in X Windows from the command line. With this you can easily close windows based on the title of them. Not amazing, but nice anyway.

With Ubuntu’s new search occupying the alt key by default, selecting the menus for a window through the keyboard only has become tricky. This can present problems if you are [going commando](http://www.codinghorror.com/blog/2007/03/going-commando---put-down-the-mouse.html), and as such I wanted an easy way to close the gitk windows that I was frequently opening.

After a short search I found [this post](http://superuser.com/questions/190067/ubuntu-how-to-send-a-close-command-to-a-window-with-a-title-matching-a-regex) on superuser which discussed, very briefly, the wmctrl command. The essence of the command, as far as I was concerned, was that it was able to take a partial title name or an exact match, and then close the window:

Partial Match:

    wmctrl -c 'substring'

Exact Match:

    wmctrl -F -c 'exact match'

Each invocation of the command will close at most one window.

After looking at the command more, I find that there are a few other beneficial commands that can be used. First, to list all of the windows:

    wmctrl -l
    0x01a00004  0 mattf-pc Desktop
    0x02200004  0      N/A DNDCollectionWindow
    0x02200005  0      N/A launcher
    0x02200006  0      N/A DNDCollectionWindow
    0x02200007  0      N/A launcher
    ...
    0x03200047  0      N/A gitk: admin_base

Then, to close a window based on the id of the window (first column of the list) you can issue the command:

    wmctrl -i -c 0x03200047

The `-i` switch makes it interpret the window name as an ID. It is important to put that before the command, otherwise it will be interpreted as the window name.

Finally, you can always bring a window into focus (moving it to the current desktop if required) by issuing the command:

    wmctrl -R 'window'

Not too shabby. Might want to set up some more compact aliases to make the command easier to handle though. Unfortunately the `w` command is already taken.
