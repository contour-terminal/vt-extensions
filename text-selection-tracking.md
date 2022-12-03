# Text Selection Tracking

Status: RFC

## Motivation

The idea behind such an extension goes nicely along with DEC mode ?2029 (Passive mouse tracking mode).

We explicitly address the issue of main screen applications (especially your shell)
having trouble adopting mouse protocol along with the idea to retain the ability
to select text outside of the main screen application's context
as well as giving the user a unified experience in how selected text looks like.

We want the terminal to select the text, as it already does when mouse event tracking
is disabled, but these selected ranges must now be forwarded to the application, if requested,
such that the application can track selections within the cursor adressable area and act accordingly,
e.g. delete the selected text if the user pressed DELETE, or do other actions with it.

Note, lines that are outside of the cursor addressable area (i.g. history lines) will be excluded
in a report.

## Feature and mode detection

Use `DECRQM` to detect if the mode is supported and if so, its current state.
This feature, text selection tracking, is considered supported if and only if
the reply contains state value `1` (set) or value `2` (reset).

## Enabling Text Selection Tracking

Sending `CSI ? 2030 h`  to the terminal will enable text selection tracking.

## Tracking event syntax.

The syntax used to send the reply is intentionally chosen close to be similar to how SGR mouse encoding works.


```
CSI > {SelectionMode} ; {StartLine} ; {StartColumn} ; {EndLine} ; {EndColumn} M
```

SelectionMode | Name                  | Notes 
--------------|-----------------------|----------------------------------
`1`           | linear selection      | With a single word being selected, `StartLine` and `EndLine` will be equal. With three or more lines being selected, all inner lines will be a full line selection. On the first and last line and the other cases, StartLine's selection starts at StartColumn and EndLine's selection ends at EndColumn.
`2`           | full line selection   | Every line between StartLine up to and including EndLine will be fully selected. StartColumn must be always set to `1` and EndColumn must be always set to the total number of columns the screen currently has.
`3`           | rectangular selection | StartLine marks the top of the rectangle, StartColumn the left edge, EndLine the bottom and EndColumn its right edge. All values are inclusive.

## Disabling Passive Mouse Tracking

Sending `CSI ? 2030 l` will disable text selection tracking.


## Affect on soft reset and hard reset

Soft reset and hard reset both also reset the DEC mode `2030` to its default state (reset).

## Additional Notes

There have been ideas for extending the idea of selection tracking to certain areas of the screen,
such that this extension could be made more useful for alt-screen apps that also do not always cover 
the full screen, for example due to vertical split screen or tooltips / floating windows.
Such possibilities are left for future extensions to this spec.

We do not think that this would break with this extension but rather extend on top of it.


## Adoption state

| Support | Terminal/Tookit/App                                        | Notes |
|---------|------------------------------------------------------------|-------|
| n/a     | [xterm.js](https://github.com/xtermjs/xterm.js/)           | |
| n/a     | [Windows Terminal](https://github.com/microsoft/terminal/) | |
| n/a     | [Contour](https://github.com/contour-terminal/contour/)    | see: https://github.com/contour-terminal/contour/pull/882 |
| n/a     | [mintty](https://github.com/mintty/mintty)                 | |
| n/a     | [notcurses](https://github.com/dankamongmen/notcurses)     | |
| n/a     | [foot](https://codeberg.org/dnkl/foot)                     | |
| n/a     | [Wezterm](https://github.com/wez/wezterm)                  | |
| n/a     | [VTE](https://gitlab.gnome.org/GNOME/vte) / gnome-terminal | |
| n/a     | [iTerm2](https://github.com/gnachman/iTerm2)               | |
| n/a     | [Kitty](https://github.com/kovidgoyal/kitty)               | |
| n/a     | [Alacritty](https://github.com/alacritty/alacritty)        | |
| n/a     | [Konsole](https://konsole.kde.org/)                        | |
| n/a     | [st](https://st.suckless.org/)                             | |

