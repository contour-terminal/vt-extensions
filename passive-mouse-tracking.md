# Passive Mouse Tracking

## Motivation

While mouse event tracking is already possible in various modes and encodings,
this feature seems most used by alt-screen applications however.

The reason why primary screen applications (such as your shell) usually don't make use of the mouse,
is the design problem, that whenever the application is requesting mouse tracking events,
the user cannot use the mouse to select text on the screen anymore.

With this VT extension, a new mode is introduced to inform the terminal that the user
may still use the mouse to select text, but still have mouse events being reported to the application
to allow passive tracking of mouse events.

This can then be used by shells to still act on mouse click events to reposition the cursor or
mouse-hover events over commands to enable the possibility for the shell to create tooltips.

## What this specification is Not

This spec is NOT trying to revolutionize mouse protocol support in the terminal
nor trying to handle every little corner case that may or may not arise in the usage scenarios.

We however attempt to address a current need in a way that this specification is easily understood,
easily implementable on both sides, and most importantly, helpful to users.

## Semantics

The semantics of mouse reporting are not altered.

The application only receives mouse tracking events for the main page area.

Passive mouse tracking mode will ONLY work in conjunction with SGR transport mode (`?1006`).
Using it with any other transport mode (e.g. X10, UTF-8, or URXVT) will be ignored.

The SGR mouse events received by the application will contain an additional parameter
to the end indicating if this event has also been handled by the user interface (terminal frontend) or not.

While mode ?2029 is active mode ?1007 has no effect (i.e. wheel events are not translated to arrow keys).

## Feature and mode detection.

Use `DECRQM` to detect if the mode is supported and if so, its current state.
Passive mouse tracking is considered supported if and only if
the reply contains state value `1` (set) or value `2` (reset).

## Enabling Passive Mouse Tracking

Sending `CSI ? 2029 h`  to the terminal will enable passive mouse tracking.
This will implicitly activate DEC mode `?1006` (SGR) as well as `?1002` (Button click event tracking).

In case mouse move events are desired as well, the application is free to send `CSI ? 2029 ; 1003 h` in one go.

## Syntax of the events being received

Since only SGR encoding is supported for passive mouse tracking, the syntax looks equivalent to this, except that an additional parameter is added to indicate whether or not the user interface (terminal) has handled this event as well.

Mouse Press (and move) Syntax: `CSI ? <ButtonAndModifierMask> ; <Column> ; <Line> ; <HandledFlag> M`

Mouse Release Syntax: `CSI ? <ButtonAndModifierMask> ; <Column> ; <Line> ; <HandledFlag> m`

Whereas `<ButtonAndModifierMask`, `Column`, `<Line>` are equivalent to how it is handled already.
`<HandledFlag>` is either `1` or greater to indicate that the user interface has handled this event as well,
or value `0` to indicate that the user interface has not handled this event.

## Disabling Passive Mouse Tracking

Sending `CSI ? 2029 l` will disable passive mouse tracking and also have any mouse event mode
as well as mouse event encoding mode disabled.
It is therefore sufficient to simply send this single VT sequence to have it all cleared.

## Disabling other mouse modes

When disabling other mouse modes (events as well as encoding) should alo disable passive mouse tracking implicitly
in order to best avoid leaking passive mouse tracking into another application by accident.

## Implementation Notes

### Mouse Coordinates

While alt-screen application do not have the problem, it must be ensured for primary screen applications
with scrollback lines available, that the mouse coordinates are mapped to the correct cell position.
With top-left being (1, 1), when the viewport of the user interface is scrolled up by 5 lines,
that grid cell must still be reported as (1, 1) as the connected terminal application does not
have the notion of viewport or would be even able to reference these.

### Fear of leaking mouse events into another application.

Applications like the shell (e.g. Bash, ZSH, Fish) are likely to frequently execute child processes,
with having the PTY handles inherited.
In case of fear that some mouse events might leak into the next application,
the application may do the following:

1. Disable mouse protocol (`CSI ?2029 l`)
2. Discard any pending input from stdin that may have be read without waiting.
3. Execute the child process.

Alternatively a very conservative way:

1. Disable mouse protocol (`CSI ?2929 l`)
2. Request `DA1`
3. Consume and discard any input until `DA1`'s response has been received.
4. Execute child process.

## Adoption state

| Support | Terminal/Tookit/App   | Notes |
|---------|------------|--------|
| n/a     | [xterm.js](https://github.com/xtermjs/xterm.js/)   | |
| n/a     | [Windows Terminal](https://github.com/microsoft/terminal/) | |
| ✅      | [Contour](https://github.com/contour-terminal/contour/)    | see: https://github.com/contour-terminal/contour/pull/882 |
| n/a     | [mintty](https://github.com/mintty/mintty)     | |
| n/a     | [notcurses](https://github.com/dankamongmen/notcurses)  | |
| n/a     | [foot](https://codeberg.org/dnkl/foot)       | |
| n/a     | [Wezterm](https://github.com/wez/wezterm)    | |
| n/a     | [VTE](https://gitlab.gnome.org/GNOME/vte) / gnome-terminal |  |
| n/a     | [iTerm2](https://github.com/gnachman/iTerm2)     | |
| n/a     | [Kitty](https://github.com/kovidgoyal/kitty)      | |
| n/a     | [Alacritty](https://github.com/alacritty/alacritty)  | |
| n/a     | [Konsole](https://konsole.kde.org/)    | |
| n/a     | [st](https://st.suckless.org/) | |

