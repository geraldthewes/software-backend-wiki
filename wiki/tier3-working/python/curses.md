# Python curses — Terminal UI (Tier 3)

> **Tier 3** | Source: Python Curses HOWTO, docs.python.org/3/howto/curses.html | Enforces/Derives From: wiki/tier3-working/python/overview.md, wiki/tier1-sources/python-peps/pep-020-zen.md

## Summary

Python's `curses` module wraps the ncurses C library to provide a terminal-independent API for full-screen, character-cell terminal interfaces. It manages screen clearing, cursor positioning, window management, text attributes, color pairs, and keyboard input. `curses` is a Linux/macOS-only module — it is not available on Windows (use `UniCurses` or `windows-curses` for cross-platform needs). For richer TUI applications consider `urwid`, `textual`, or `rich` before reaching for raw curses.

## Key Concepts

### Initialization and Cleanup

The safest way to initialize and clean up curses is through `curses.wrapper()`:

```python
import curses

def main(stdscr: curses.window) -> None:
    # stdscr is the main window, already initialized
    stdscr.clear()
    stdscr.addstr(0, 0, "Hello, curses!")
    stdscr.refresh()
    stdscr.getch()   # wait for keypress

curses.wrapper(main)
# wrapper() handles initscr(), noecho(), cbreak(), keypad(), and endwin()
# It also catches exceptions and calls endwin() before re-raising
```

Manual initialization (when wrapper is insufficient):

```python
stdscr = curses.initscr()
curses.noecho()          # don't echo keypresses to screen
curses.cbreak()          # react immediately, no Enter needed
stdscr.keypad(True)      # enable special keys (arrows, F-keys, etc.)
curses.start_color()     # enable color support

try:
    application(stdscr)
finally:
    curses.nocbreak()
    stdscr.keypad(False)
    curses.echo()
    curses.endwin()      # restore terminal to original state
```

### Coordinate System

curses uses **(y, x)** order, not (x, y). The origin `(0, 0)` is the **top-left** corner:

```python
rows, cols = curses.LINES, curses.COLS    # terminal dimensions
max_y, max_x = stdscr.getmaxyx()         # window dimensions

stdscr.addstr(0, 0, "Top-left")
stdscr.addstr(rows - 1, 0, "Bottom-left")
```

### Windows

```python
# Create a subwindow
height, width = 10, 40
begin_y, begin_x = 5, 10
win = curses.newwin(height, width, begin_y, begin_x)
win.box()                          # draw border
win.addstr(1, 1, "Window content")
win.refresh()

# Pads — larger than screen, scrollable
pad = curses.newpad(200, 80)
pad.addstr(0, 0, "Lots of content...")
# Refresh a portion of the pad onto the screen
pad.refresh(pad_row, pad_col, screen_y1, screen_x1, screen_y2, screen_x2)
```

### Displaying Text with Attributes

```python
# Text attributes
stdscr.addstr(0, 0, "Bold text",      curses.A_BOLD)
stdscr.addstr(1, 0, "Underlined",     curses.A_UNDERLINE)
stdscr.addstr(2, 0, "Reversed",       curses.A_REVERSE)
stdscr.addstr(3, 0, "Dim text",       curses.A_DIM)

# Colors — initialize pairs (pair 0 is always white-on-black)
curses.start_color()
curses.init_pair(1, curses.COLOR_RED,   curses.COLOR_BLACK)
curses.init_pair(2, curses.COLOR_GREEN, curses.COLOR_BLACK)
curses.init_pair(3, curses.COLOR_CYAN,  curses.COLOR_BLUE)

stdscr.addstr(4, 0, "Red on black", curses.color_pair(1))
stdscr.addstr(5, 0, "Bold red",     curses.color_pair(1) | curses.A_BOLD)
```

### Keyboard Input

```python
# Basic key reading
c = stdscr.getch()                  # blocks, returns int
if c == ord('q'):
    break
elif c == curses.KEY_UP:
    cursor_y = max(0, cursor_y - 1)
elif c == curses.KEY_DOWN:
    cursor_y = min(max_y - 1, cursor_y + 1)

# Non-blocking
stdscr.nodelay(True)
c = stdscr.getch()
if c == curses.ERR:
    pass   # no input available

# String input (Emacs-style editor)
from curses.textpad import Textbox, rectangle

editwin = curses.newwin(3, 30, 2, 1)
rectangle(stdscr, 1, 0, 4, 31)
stdscr.refresh()
box = Textbox(editwin)
box.edit()
text = box.gather().strip()
```

### Refresh Strategy

Calling `refresh()` on each window separately causes visible flicker. For smooth updates:

```python
# Mark windows as needing refresh (no actual display update)
win1.noutrefresh()
win2.noutrefresh()
status_bar.noutrefresh()

# Single atomic display update
curses.doupdate()
```

## Complete Pattern: Event Loop

```python
import curses

def run(stdscr: curses.window) -> None:
    curses.curs_set(0)     # hide cursor
    curses.start_color()
    curses.init_pair(1, curses.COLOR_GREEN, curses.COLOR_BLACK)

    stdscr.addstr(0, 0, "Press 'q' to quit", curses.color_pair(1))
    stdscr.refresh()

    while True:
        key = stdscr.getch()
        if key == ord('q'):
            break
        elif key == curses.KEY_RESIZE:
            rows, cols = stdscr.getmaxyx()
            stdscr.clear()
            stdscr.addstr(0, 0, f"Resized: {rows}x{cols}")
            stdscr.refresh()

curses.wrapper(run)
```

## Agent Guidance

### Do

- Always use `curses.wrapper()` — it handles initialization, cleanup, and terminal restoration on exceptions.
- Remember coordinates are `(y, x)`, not `(x, y)`.
- Call `curses.start_color()` before using `init_pair()` or `color_pair()`.
- Use `noutrefresh()` on multiple windows followed by `doupdate()` to prevent flicker.
- Handle `curses.KEY_RESIZE` in the event loop — terminal resize is a common user action.
- Hide the cursor with `curses.curs_set(0)` for full-screen applications.
- Check `curses.has_colors()` before using color features on unknown terminals.

### Do Not

- Do not use curses on Windows without `windows-curses` — the module will `ImportError`.
- Do not call `curses.endwin()` without `initscr()` first — it will crash.
- Do not assume terminal dimensions are fixed — read `stdscr.getmaxyx()` on each `KEY_RESIZE`.
- Do not call `refresh()` on every window separately in animation loops — use `noutrefresh()` + `doupdate()`.
- Do not use raw curses for applications that need rich widgets — prefer `urwid` or `textual`.

## Checklist

- [ ] `curses.wrapper()` used as entry point
- [ ] Color initialized with `curses.start_color()` before `init_pair()`
- [ ] Coordinates passed in `(y, x)` order
- [ ] `noutrefresh()` + `doupdate()` used for multi-window updates
- [ ] `KEY_RESIZE` handled in event loop
- [ ] Platform availability checked or documented (Linux/macOS only)

## See Also

- wiki/tier3-working/python/overview.md
- wiki/tier1-sources/python-peps/pep-020-zen.md
- wiki/tier3-working/python/argparse.md
- wiki/tier3-working/python/async-patterns.md

## Source

Python Curses HOWTO, docs.python.org/3/howto/curses.html. Python `curses`, `curses.textpad`, `curses.ascii` module documentation.
