# tmux-url-select

`tmux-url-select` is a perl script that lets you select URLs with the keyboard.

It integrates with tmux to capture the current pane buffer, switch to a window
with highlighted links, and let you select the link you want to open/yank.

It's like the urxvtperls [url-select][1] script or [urlview][2], except that
it's inspired by the interface of the former and the capture-pane method
usually used with the latter.

## Features

 * Tightly integrated to tmux to prevent flickering when switching to url
   selection
 * No dependencies (Not even ncurses, so no portability either!)
 * Uses colors! Configurable colors!
 * Configurable external commands too!
 * vi-like keybindings (actually just `j` and `k`)

[1]: https://github.com/muennich/urxvt-perls/blob/master/url-select
[2]: http://packages.qa.debian.org/u/urlview.html

## Is it any good?

[Yes][3]

[3]: https://news.ycombinator.com/item?id=3067434

## Huge animated GIF

![](http://dump.dequis.org/e1f1c.gif)

## Requirements

Depends on `perl`, `tmux` and `stty`.

Optional and configurable: `xdg-open` (can be any url opener or browser) and
`xclip` (for yank)

## Installation

Place it somewhere in your path with name `tmux-url-select` and `chmod +x` it.

Add this to your `.tmux.conf`:

    bind some-key-here run tmux-url-select

Where some-key-here is any key you want to use to start url selection.
Personally I use "z" which is an unused keybinding that is really close to my
tmux prefix key (`` ` ``)

    bind z run tmux-url-select

## Usage

Once you're inside tmux-url-select, keybindings:

 * `j`: down
 * `k`: up
 * `0`-`9`: select link by number
 * `y`: yank (copy to clipboard)
 * Enter or `o`: open link
 * `Y` / `O`: yank or open link without closing
 * `q`: quit

You can't use arrow keys because those are more complex than a single ascii
character.

## Configuration

There's a bunch of constants near the top of the file, you can modify them to
your liking.

    use constant COMMAND => 'xdg-open %s';
    use constant YANK_COMMAND => 'echo %s | xclip -i';

    use constant SHOW_STATUS_BAR => 1;
    use constant VERBOSE_MESSAGES => 0;
    use constant TMUX_WINDOW_TITLE => 'Select URL';

    use constant PROMPT_COLOR => "\033[42;30m";
    use constant ACTIVE_LINK_HIGHLIGHT => "\033[44;4m";
    use constant NORMAL_LINK_HIGHLIGHT => "\033[94;1;4m";

Probably should add some explanations. Maybe. For now just go ahead and
experiment with stuff.

## Known issues

If a line with a link has a background color, it will get reset after the link.
I have no idea how to do this without writing a parser of ansi escape codes or
using ncurses properly, so I'm leaving it unfixed as a reminder that as humans
we're all flawed in different ways.

Might flicker when selecting links because the whole screen is redrawn. It can't
be helped. Works fine for me most of the time.

Already workarounded, but: Some url openers don't like the fact that we kill the
terminal right after running the process. `gvfs-open` in particular opens my
browser asynchronously and doesn't wait for it, so there's a race condition in
which the terminal often get closed before the browser gets to do anything.
Workarounds for this:

 * `sleep 1` after running the command. The laziest and most generic workaround,
   currently applied in the code because it doesn't cause any other issues.
 * Prefix your url open command with `setsid`. This makes the child process a
   process group leader, so it won't care about getting the terminal closed.
 * Prefix with `nohup`. Does something similar but not quite the same as
   `setsid`, and didn't work for me, but worth a try!
 * Wrap the command with `$(...)`. Weirdest one, I found this trick accidentally
   last year and [had to ask stack overflow][4] about it. It makes bash wait for
   the whole process tree to finish executing, apparently.
 * Prefix with `strace -f -o /dev/null`. Another weird one! This one also seems
   to force waiting for all child processes to finish, by attaching to them with
   ptrace. It's overkill and that makes it fun.

[4]: http://stackoverflow.com/questions/16874043/bash-command-substitution-forcing-process-to-foreground

## FAQ

Q: Why perl? It's dead and it sucks, cool kids use node.js nowadays.

A: It's fun. Fun things are fun.
