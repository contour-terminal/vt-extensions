# Clipboard Extension

The Clipboard Extension supports copying to the clipboard using a subset of the XTerm `OSC 52` escape sequence.

## Feature detection

Support for this functionality in a terminal is indicated by the extension parameter `52` in the Device Attributes (`DA`) report.

## Syntax
The escape sequence has the following syntax:

```
OSC 52 ; Pc ; Pd ST
```

Conforming terminals MUST at least support an empty _Pc_ value, and a _Pc_ value of `c`. Other values are optional.

Conforming terminals MUST support a _Pd_ value that is a base64 encoded representation of the content that will be copied to the clipboard. They SHOULD also support an empty _Pd_ value to clear the clipboard. They MAY support a _Pd_ value of `?` to query the clipboard content (as specified in the XTerm documentation).

If a terminal has an option to disable clipboard access, it MUST only indicate support for this extension when writing to the clipboard is actually allowed.

### Terminal Adoption State

| Support | Terminal                                                   | Notes |
|---------|------------------------------------------------------------|-------|
| not yet | [Alacritty](https://github.com/alacritty/alacritty)        | |
| ✅      | [Bobcat](https://github.com/ismail-yilmaz/Bobcat)          | [commit](https://github.com/ismail-yilmaz/Terminal/commit/4f62f0fd688d91e802779c30342bc5faceda50ef)|
| ✅      | [Contour](https://github.com/contour-terminal/contour/)    | [contour#1769](https://github.com/contour-terminal/contour/pull/1769) |
| not yet | [DomTerm](https://github.com/PerBothner/DomTerm)           | [domterm#124](https://github.com/PerBothner/DomTerm/issues/124) |
| ✅      | [foot](https://codeberg.org/dnkl/foot)                     | [foot#2130](https://codeberg.org/dnkl/foot/pulls/2130) |
| ✅      | [ghostty](https://github.com/ghostty-org/ghostty)          | [commit](https://github.com/ghostty-org/ghostty/commit/259228698873c0c934741445ec6790cfafb64502) |
| not yet | [iTerm2](https://github.com/gnachman/iTerm2)               | |
| ✅      | [Kitty](https://github.com/kovidgoyal/kitty)               | [commit](https://github.com/kovidgoyal/kitty/commit/eabddc287043083e25f57b236df7f0c9883760a5) |
| not yet | [Konsole](https://konsole.kde.org/)                        | |
| not yet | [mintty](https://github.com/mintty/mintty)                 | |
| not yet | [mlterm](https://github.com/arakiken/mlterm)               | [mlterm#144](https://github.com/arakiken/mlterm/issues/144) |
| not yet | [st](https://st.suckless.org/)                             | |
| not yet | [VTE](https://gitlab.gnome.org/GNOME/vte) / gnome-terminal | |
| ✅      | [Wezterm](https://github.com/wez/wezterm)                  | [wezterm#7046](https://github.com/wezterm/wezterm/pull/7046) |
| ✅      | [Windows Terminal](https://github.com/microsoft/terminal/) | [wt#19034](https://github.com/microsoft/terminal/pull/19034) |
| not yet | [xterm.js](https://github.com/xtermjs/xterm.js/)           | [xterm#5351](https://github.com/xtermjs/xterm.js/issues/5351) |

### Toolkit/App Adoption State

| Support | Tookit/App                                                 | Notes |
|---------|------------------------------------------------------------|-------|
| ✅      | [dte](https://github.com/craigbarnes/dte)                  | [commit](https://github.com/craigbarnes/dte/commit/bed8412692d2363e1198e838393b0f0cb1c197d8) |
| not yet | [neovim](https://github.com/neovim/neovim)                 | [neovim#34472](https://github.com/neovim/neovim/issues/34472) |
| not yet | [notcurses](https://github.com/dankamongmen/notcurses)     | |
| ✅      | [tmux](https://github.com/tmux/tmux)                       | [commit](https://github.com/tmux/tmux/commit/d858ad1179d98fb5ab31a4e077e789200ef7e411) |

In case some project is adding support for this feature, please leave a comment or contact me, so we can keep the spec and implementation state table up to date.
