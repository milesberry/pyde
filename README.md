# pyde

A self-contained, single-file Python IDE that runs entirely in the browser — no server, no install, no build step. Double-click `index.html`, or use it live at **[milesberry.net/pyde](https://milesberry.net/pyde/)**.

Built for teaching (GCSE/A Level CS style exercises), but useful for any quick "write and run some Python" need.

## Features

- **Run Python** — full CPython 3.12, compiled to WebAssembly (via [Pyodide](https://pyodide.org)), executing directly in your tab. stdout/stderr are captured and shown in the console, with errors colour-coded.
- **New** — a clearly-labelled way to start a blank program, distinct from just deleting everything in the editor (which reads as normal practice to a pupil watching, when the point is to save the old one and start a deliberate new one).
- **Syntax highlighting** — [CodeMirror 5](https://codemirror.net/5/) with a Python mode and a dark (Dracula) / light (Eclipse) theme toggle. Copying a selection also writes a colour-highlighted `text/html` version to the clipboard alongside the plain text, matching the current theme, so pasting into Word/OneNote/Google Docs keeps the syntax colours instead of landing as flat text.
- **Multi-cursor editing** — select a word and press `Cmd-D` / `Ctrl-D` to select the next occurrence, VS Code / Sublime style.
- **Block comment / indent** — select some lines, press `#` to toggle a comment prefix on all of them, or `Tab` / `Shift-Tab` to indent/dedent the block.
- **Download / Upload** — save your code as a `.py` file, or load one from disk.
- **Virtual input file** — paste text into the "Program input file" box and it's readable from Python via `open("filename")`. Any files your program writes are picked up automatically — text files show their content right on the page (handy on iPad, where downloading a blob is clunky), with a download link alongside; binary files are download-only.
- **Auto-recovery** — the editor and the input file both save to `localStorage` as you type and restore automatically next visit, so closing the tab, a crash, or a hung program doesn't lose your work. A page refresh after editing further than a share link always reflects your latest changes, not the stale shared snapshot.
- **Turtle graphics** — `import turtle` (or `from turtle import *`) renders to an on-page canvas.
- **p5.js sketches** — write `setup()` / `draw()` functions and call `p5.background(...)`, `p5.ellipse(...)`, etc. p5.js is loaded from CDN on demand.
- **Parsons puzzles** — turn the current code into a drag-and-drop line-reordering puzzle, with a Check button that highlights right/wrong placements, and a Copy button for the completed arrangement. Shareable as a standalone puzzle link — the answer is never loaded into the editor until solved correctly, so closing the puzzle before solving it doesn't reveal the original code.
- **Share via URL** — the whole program (or just a Parsons puzzle) is LZ-compressed into the page's URL hash, so a link fully reproduces it with no backend. If the program reads a file via the input box above, that content travels in the link too, so a shared example that depends on it actually works for whoever opens it.
- **Tidy** — auto-formats the code to PEP8 style with [autopep8](https://github.com/hhatto/autopep8), installed on demand via `micropip` the first time it's used.

## Supported libraries

Because Python is run via Pyodide, most of the **standard library** works out of the box, including:

`math`, `random`, `string`, `re`, `datetime`, `itertools`, `collections`, `json`, `statistics`, `decimal`, `fractions`, `functools`, `dataclasses`, `enum`, `typing`, `textwrap`, `copy`, `heapq`, `bisect`, and more.

A couple of modules are **shimmed** rather than the real thing, because the browser sandbox can't support them as-is:

- **`turtle`** — the real stdlib `turtle` needs Tkinter, which isn't available in a WebAssembly browser build. Instead, `import turtle` / `from turtle import *` loads a custom canvas-backed replacement covering the common API: `forward`, `backward`, `left`, `right`, `goto`, `circle`, `penup`/`pendown`, `color`/`pencolor`/`fillcolor`, `begin_fill`/`end_fill`, `write`, `Screen()`, and the usual short-form aliases (`fd`, `bk`, `lt`, `rt`, ...).
- **`p5`** — not a real PyPI package. Detected when your code contains `import p5` / `from p5 import` or a `def draw():` function, this loads [p5.js](https://p5js.org) from CDN and exposes it as a Python module. `import p5; p5.whatever(...)` forwards any p5.js call, camelCase and all. `from p5 import *` also works, for a curated set of the common drawing/canvas/colour functions — including the snake_case and Processing-style names (`no_fill`, `no_stroke`, `size` for canvas sizing, ...) that convention expects, alongside the plain camelCase ones. Anything that would shadow a Python builtin (`map`, `min`, `max`, `print`, `pow`, ...) is deliberately left out of that starred set — `remap` is provided in place of `map`, and the rest stay reachable via `p5.min(...)` etc. Live-valued properties like `width`, `mouseX`, or `frameCount` aren't star-importable either (there's no way to make a plain Python name "live"), so use `p5.width` and friends for those.

**Not supported:** anything needing real OS features — `tkinter` (GUI), `socket` (raw networking), `subprocess`, `multiprocessing`, real threads, or the real filesystem (there's an in-memory virtual filesystem instead, used for the input/output file feature). Third-party PyPI packages (numpy, pandas, etc.) aren't loaded — there's no `pip`/`micropip` wiring, so only what ships with Pyodide's base distribution is available.

`input()` works. For plain (non-turtle, non-p5) programs, it shows an inline text box right in the console, keeping the full scrollback visible while you answer — this covers both `input()` at top-level code (a guess-the-number loop) and `input()` called from inside a helper function (very common — e.g. a `get_guess()` function, or a whole game wrapped in its own `def`), including chains of functions calling each other. If a program uses turtle/p5, or does something with `input()` too dynamic to safely rewrite (aliasing a function, passing it around as a value, recursion, decorators, a name shadowed elsewhere in the file), it falls back to a native `prompt()` dialog instead. See [colophon.md](colophon.md) for how (and why it can't always avoid the popup).

`time.process_time()` doesn't work — it always reads `0.0`, even across genuine CPU-bound work, because it relies on OS-level CPU accounting that doesn't exist inside a WebAssembly sandbox. Use `time.time()` or `time.perf_counter()` for timing code instead; both work correctly.

There's currently no way to stop a running program from the page — an infinite loop or a long `time.sleep()` will freeze the tab, since Python runs on the same thread as the page itself. Reload the tab to recover.

## Running it locally

There's nothing to install. Open `index.html` directly in a browser, or serve the folder with any static file server if you prefer.

## License

[MIT](LICENSE) — use it, modify it, teach with it, no strings attached.
