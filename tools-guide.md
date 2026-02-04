---
layout: default
title: Guide of things that i intend to use 
nav_order: 1
has_children: true
---

# Guide of things that i intend to use

## Tmux

- `C` means Ctrl
- `C-b` is the prefix for everything
- `C-b ?` is the manual
- `C-b %` splits vertically
- `C-b arrows` to navigate the panes

There are probably much smarter things that one can do

Okay but what are this panes windows session, thats a bit hard to understand

### Sessions, windows and panes

A Session is the big container, with inside windows, and inside those panes,
when you split the screen you are creating new panes.

- `tmux a ` to attach to a session from outside, the only session or the last one
- `tmux a -t <session_name>` to a ttach to a specific session
- `exit` to exit and kill the session
- `C-b d` to detach from the session and let it run
