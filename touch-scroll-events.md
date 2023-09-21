# Touch scroll events

## Motivation

Modern hardware has more inputs than a keyboard or mouse: touch input devices
are increasingly being used, whether on mobile or desktop. Touch devices offer
additional inputs that are not currently exposed to the terminal, such as
gestures (typically a three-finger motion). Touchscreen devices also offer a
different user experience, and may benefit from different handling of scroll
events.

A typical TUI application will scroll 3 lines per mouse-wheel click. When a user
scrolls with touch, this multiplier can make the TUI scroll significantly faster
than a GUI application would in response to touch. By exposing touch events,
such a TUI could use a 1x multiplier if the scroll was known to be from a touch
input.

## Semantics

Touch input events are sent as an SGR mouse sequence. A mouse SGR sequence looks
like

`CSI < Cb ; Cx ; Cy M`

Cb is an 8 bit number of the encoded mouse event.

| 0          | 0          | 0      | 0       | 0    | 0     | 0                 | 0           |
| ---------- | ---------- | ------ | ------- | ---- | ----- | ----------------- | ----------- |
| Wheel mask | Wheel mask | Motion | Control | Meta | Shift | Button 3, Release | Button 1, 2 |

The 8 bit value is needed for non-SGR reporting. In legacy modes, the encoded
button is sent as printable text (IE ' ' signifies button 1 click). In SGR
reporting, the numeric value is sent. This allows for an extended button
encoding.

This specification extends the reporting value by two bits. These bits are used
to encode two-finger and three-finger scroll events, similar to wheel events.

The encoded events thus will be:

| Encoded value | Meaning              |
| :-----------: | -------------------- |
|       0       | Button 1             |
|       1       | Button 2             |
|       2       | Button 3             |
|  0 + 32 = 32  | Button 1 + Motion    |
|  1 + 32 = 33  | Button 2 + Motion    |
|  2 + 32 = 34  | Button 3 + Motion    |
|  3 + 32 = 35  | Release + Motion     |
|  0 + 64 = 64  | Wheel up             |
|  1 + 64 = 65  | Wheel down           |
|  2 + 64 = 66  | Wheel left           |
|  3 + 64 = 67  | Wheel right          |
| 0 + 128 = 128 | Button 8             |
| 1 + 128 = 129 | Button 9             |
| 2 + 128 = 130 | Button 10            |
| 3 + 128 = 131 | Button 11            |
| 0 + 256 = 256 | Touch scroll up      |
| 1 + 256 = 257 | Touch scroll down    |
| 2 + 256 = 258 | Touch scroll left    |
| 3 + 256 = 259 | Touch scroll right   |
| 0 + 512 = 512 | Gesture scroll up    |
| 1 + 512 = 513 | Gesture scroll down  |
| 2 + 512 = 514 | Gesture scroll left  |
| 3 + 512 = 515 | Gesture scroll right |

## Feature detection

Touch events are queried/set/reset using DEC private mode `2030`.

Use `DECRQM` to detect if the mode is supported and if so, its current state.

## Using the feature

Enable the feature by setting mode `2030`. Setting this mode implicitly sets
button tracking mode (`1002`) and SGR reporting mode (`1006`).

`\x1B[?2006h`

The user scrolls up with their touchpad.

`\x1B[<256;37;42M`

The user does a three-finger swipe to the right.

`\x1B[<515;37;42M`

Note that this proposal does not disambiguate a touch-click from a mouse click.
One, two, and three finger clicks are canonically mapped to left, right, and
middle click already. Users know to expect these responses already and further
handling of these events is likely not needed.

## Adoption State

| Support | Terminal/Tookit/App                                        | Notes |
| ------- | ---------------------------------------------------------- | ----- |
| n/a     | [xterm.js](https://github.com/xtermjs/xterm.js/)           |       |
| n/a     | [Windows Terminal](https://github.com/microsoft/terminal/) |       |
| n/a     | [Contour](https://github.com/contour-terminal/contour/)    |       |
| n/a     | [mintty](https://github.com/mintty/mintty)                 |       |
| n/a     | [notcurses](https://github.com/dankamongmen/notcurses)     |       |
| n/a     | [foot](https://codeberg.org/dnkl/foot)                     |       |
| n/a     | [Wezterm](https://github.com/wez/wezterm)                  |       |
| n/a     | [VTE](https://gitlab.gnome.org/GNOME/vte) / gnome-terminal |       |
| n/a     | [iTerm2](https://github.com/gnachman/iTerm2)               |       |
| n/a     | [Kitty](https://github.com/kovidgoyal/kitty)               |       |
| n/a     | [Alacritty](https://github.com/alacritty/alacritty)        |       |
| n/a     | [Konsole](https://konsole.kde.org/)                        |       |
| n/a     | [st](https://st.suckless.org/)                             |       |
