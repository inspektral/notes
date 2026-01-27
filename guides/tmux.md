# Tmux

tmux is a terminal multiplexer, that allows for having multiple terminal sessions, with different, sessions, windows and panes. It makes quite a bit of sense to use it especially if one has different sessions via ssh, and also wants to split into panes to have more flexibilty when using CLI stuff. There are no many reasons to not have it and it is so customizable and usable anywhere that it is kinda a super terminal learn it once use it everywhere.

- `C` means Ctrl
- `C-b` is the prefix for everything
- `C-b ?` is the manual
- `C-b %` splits vertically
- `C-b arrows` to navigate the panes


There are commands and keybindings, that makes sense, the command prompt is activated with `C-b :` and then
one can type any command. some commands already have keybindings, other keybindings can be assigned I guess.

## Sessions, windows and panes

A Session is the big container, with inside windows, and inside those panes,
when you split the screen you are creating new panes.

- `tmux a ` to attach to a session from outside, the only session or the last one
- `tmux a -t <session_name>` to a ttach to a specific session
- `exit` to exit and kill the session
- `C-b d` to detach from the session and let it run

## Pane commands

- `split-window` is the main command to make new panes, main options are `-v` and `-h`
- `C-b %` splits vertically
- `C-b "` splits horizontally
- `C-b arrows` to navigate the panes
- `C-b x` closes the current pane, I guess killing anything inside it


## Windows commands

- `:select-window` is the basic command to change window, then there are mulitple premade keybindings
- `C-b c` new window, default name
- `C-b ,` rename window
- `C-b n` go to next window
- `C-b p` go to previous window
- `C-b 0-9` go to window n
- `C-b w` list windows

# Mouse mode

I activated the mous mode, it lets you switch panes by clicking and select things in the terminal but it is
 a bit clunky. It needs some practice. Mouse mode can be activated adding `set -g mouse on` to `.tmux.conf`

# Customizations

Some stuff can be customized in the `.tmux.conf` file, from apperance to keybindings. that's quite fun and you can make
tmux quite personalized to your needs.
