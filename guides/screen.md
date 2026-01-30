# Screen

Is another terminal multiplexer, basically the father of tmux, much simpler but it comes preinstalled everywhere so that is pretty handy and it makes worth to know how to use it.
The main point is to have multiple sessions on a server, so basically ssh then screen to open up a session, start a program (eg an ML training run) and then detach, in this way the run keeps going in the background and we can run other commands in our session without the need of multiple ssh sessions to the same server.

# Commands

As usual C means Ctrl

- `screen -ls` list of the current session with details
- `screen -S <session-name>` creates new session and attaches to it
- `C-d` detaches from session and kills it
- `C-a-d` detaches from session without killing it
- `screen -r <session-name>` attaches to running session
