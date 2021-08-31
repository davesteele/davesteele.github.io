---
layout: post
title:  "Using Curses with Python Asyncio"
date:   2021-8-29 18:31:00
categories: development
excerpt_separator: <!--more-->
---

## Notes on Using Curses with Python Asyncio

[Curses](https://en.wikipedia.org/wiki/Curses_(programming_library)) has been
around for a long time.
[Asyncio](https://docs.python.org/3/library/asyncio.html) is more than a decade
newer. Curses coding practices were well established by that time.

It came time for me to write my first curses application (needed a GUI with
minimal infrastructure requirements), which needed asyncio capabilities. I
wouldn't have minded finding more information on the webs on how the two
standard libraries interact.

Here's what I eventually discovered after coding the app (with a bonus template
to start your own python curses app).

<!--more-->

## It's about Polling vs. Blocking I/O

The curses
[getch()](https://docs.python.org/3/library/curses.html#curses.window.getch)
call can operate in both a blocking and a
[non-blocking](https://docs.python.org/3/library/curses.html#curses.window.nodelay)
mode. Either will work in asyncio, with at least one caveat - if the blocking
getch() is called in a separate thread via an asyncio executor, it will not
return some events, notably KEY\_RESIZE. A polling mechanism is therefore
recommented, e.g.:


```
    self.stdscr.nodelay(True)

    while not self.done:
        char = self.stdscr.getch()
        if char == ERR:
            await asyncio.sleep(0.1)
        elif char == KEY_RESIZE:
            self.make_display()
        else:
            self.handle_char(char)
```

See a working demo template at
[curses\_demo.py](https://gist.github.com/davesteele/8838f03e0594ef11c89f77a7bca91206)
