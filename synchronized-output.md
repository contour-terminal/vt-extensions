## Synchronized Output

Original article was a gist, found [here](https://gist.github.com/christianparpart/d8a62cc1ab659194337d73e399004036).

Alternate words to this feature has been:

- Batched Rendering
- Synchronized Updates
- Synchronized Output

But they all mean the same.

Synchronized output is merely implementing the feature as inspired by iTerm2 synchronized output,
except that it's not using the rare DCS but rather the well known `SM ?` and `RM ?`. iTerm2 has now also adopted to use the new syntax instead of using DCS.

### Semantics

When rendering the screen of the terminal, the Emulator usually iterates through each visible grid cell and renders its current state. With applications updating the screen a at higher frequency this can cause tearing.

This mode attempts to mitigate that.

When the synchronization mode is enabled following render calls will keep rendering the last rendered state.
The terminal Emulator keeps processing incoming text and sequences. When the synchronized update mode is disabled again the renderer may  fetch the latest screen buffer state again, effectively avoiding the tearing effect by unintentionally rendering in the middle a of an application screen update.

### Feature detection

Use `CSI ? 2026 $ p` to query the state of the (DEC) mode `2026`. This works for any private mode number.
If you get nothing back (DECRQM not implemented at all) or you get back a `CSI ? 2026 ; 0 $ y`
then synchronized output is not supported.
See [DECRQM](https://vt100.net/docs/vt510-rm/DECRQM.html) (request)
and [DECRPM](https://vt100.net/docs/vt510-rm/DECRPM.html) (response) for more details.

[DECRPM](https://vt100.net/docs/vt510-rm/DECRPM.html) can respond with different values.

Value | Documentation | Relevance for synchronized output mode ?2026
------|---------------|--------------------------
`0`   | Mode is not recognized | not supported
`1`   | Set   | supported and screen updates are not shown to the user until mode is disabled
`2`   | Reset | supported and screen updates are shown as usual (e.g. as soon as they arrive)
`3`   | Permanently set | undefined
`4`   | Permanently reset | not supported

### Using the feature

Use `CSI ? 2026 h` to enable batching output commands into a command queue.

Use `CSI ? 2026 l` when done with your current frame rendering, implicitly updating the render state by reading out the latest grid buffer state.


### Notation

Some developers name the beginning and end of such a synchronized frame (and therefore the instance)

* `BSU` (begin synchronized update, `CSI ? 2026 h`), and
* `ESU` (end synchronized update, `CSI ? 2026 l`).

### Timeout

So far there is no real concensus if and if so how long a timeout should be.
The toolkit/application implementer should keep this in mind.
However, a too short timeout (maybe due to a very slow connection) won't be worse
than having no synchronized output at all.

### Adoption State

| Support | Terminal/Tookit/App                                        | Notes |
|---------|------------------------------------------------------------|-------|
| ✅      | [Alacritty](https://github.com/alacritty/alacritty)        | [alacritty#96](https://github.com/alacritty/vte/pull/96) |
| ✅      | [Contour](https://github.com/contour-terminal/contour/)    | |
| ✅      | bubbletea | https://github.com/charmbracelet/bubbletea/discussions/1320 |
| ✅      | [foot](https://codeberg.org/dnkl/foot)                     | |
| ✅      | [ghostty](https://github.com/ghostty-org/ghostty)          | [commit](https://github.com/ghostty-org/ghostty/commit/2cc1e4371651ccd692f3e8e8ba5a5cf731b2e21f) |
| ✅      | [iTerm2](https://github.com/gnachman/iTerm2)               | |
| ✅      | [Jexer](https://gitlab.com/AutumnMeowMeow/jexer)           | |
| ✅      | [kava](https://github.com/karlstav/cava)                   | https://github.com/karlstav/cava/pull/576 |
| ✅      | [Kitty](https://github.com/kovidgoyal/kitty)               | [commit](https://github.com/kovidgoyal/kitty/commit/5768c54c5b5763e4bbb300726b8ff71b40c128f8) |
| unknown | [Konsole](https://konsole.kde.org/)                        | |
| ✅      | [mintty](https://github.com/mintty/mintty)                 | |
| ✅      | [notcurses](https://github.com/dankamongmen/notcurses)     | [notcurses#1582](https://github.com/dankamongmen/notcurses/issues/1582) |
| unknown | [st](https://st.suckless.org/)                             | |
| unknown | [urxvt](https://software.schmorp.de/pkg/rxvt-unicode.html) | |
| not yet | [VTE](https://gitlab.gnome.org/GNOME/vte) / gnome-terminal | [gitlab/vte#15](https://gitlab.gnome.org/GNOME/vte/-/issues/15) |
| ✅      | [Warp](https://github.com/warpdotdev/Warp)                 | [warp#2185](https://github.com/warpdotdev/Warp/issues/2185) |
| ✅      | [Wezterm](https://github.com/wez/wezterm)                  | [wezterm#882](https://github.com/wez/wezterm/issues/882) |
| ✅      | [Windows Terminal](https://github.com/microsoft/terminal/) | [wt#18826](https://github.com/microsoft/terminal/pull/18826) |
| not yet | [xterm.js](https://github.com/xtermjs/xterm.js/)            | [xterm.js#3375](https://github.com/xtermjs/xterm.js/issues/3375) |
| ✅ (see notes) | [st](https://st.suckless.org/)                      | via patch: https://st.suckless.org/patches/sync/ |
| ✅      | [Zellij](https://github.com/zellij-org/zellij)             | https://github.com/zellij-org/zellij/pull/2977 |


In case some project is adding support for this feature, please leave a comment or contact me, so we can keep the spec and implementation state table up to date.
