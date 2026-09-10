# pyde

A self-contained, single-file Python IDE that runs entirely in the browser — no server, no install, no build step. Double-click `index.html`, or use it live at **[milesberry.net/pyde](https://milesberry.net/pyde/)**.

Built for teaching (GCSE/A Level CS style exercises), but useful for any quick "write and run some Python" need.

## Features

- **Run Python** — full CPython 3.12, compiled to WebAssembly (via [Pyodide](https://pyodide.org)), executing directly in your tab. stdout/stderr are captured and shown in the console, with errors colour-coded.
- **Syntax highlighting** — [CodeMirror 5](https://codemirror.net/5/) with a Python mode and the Dracula theme.
- **Multi-cursor editing** — select a word and press `Cmd-D` / `Ctrl-D` to select the next occurrence, VS Code / Sublime style.
- **Block comment / indent** — select some lines, press `#` to toggle a comment prefix on all of them, or `Tab` / `Shift-Tab` to indent/dedent the block.
- **Download / Upload** — save your code as a `.py` file, or load one from disk.
- **Virtual input file** — paste text into the "Program input file" box and it's readable from Python via `open("filename")`. Any files your program writes are picked up automatically and offered as downloads.
- **Turtle graphics** — `import turtle` (or `from turtle import *`) renders to an on-page canvas.
- **p5.js sketches** — write `setup()` / `draw()` functions and call `p5.background(...)`, `p5.ellipse(...)`, etc. p5.js is loaded from CDN on demand.
- **Parsons puzzles** — turn the current code into a drag-and-drop line-reordering puzzle, with a Check button that highlights right/wrong placements. Shareable as a standalone puzzle link.
- **Share via URL** — the whole program (or just a Parsons puzzle) is LZ-compressed into the page's URL hash, so a link fully reproduces it with no backend.
- **Tidy** — auto-formats the code to PEP8 style with [autopep8](https://github.com/hhatto/autopep8), installed on demand via `micropip` the first time it's used.

## Supported libraries

Because Python is run via Pyodide, most of the **standard library** works out of the box, including:

`math`, `random`, `string`, `re`, `datetime`, `itertools`, `collections`, `json`, `statistics`, `decimal`, `fractions`, `functools`, `dataclasses`, `enum`, `typing`, `textwrap`, `copy`, `heapq`, `bisect`, and more.

A couple of modules are **shimmed** rather than the real thing, because the browser sandbox can't support them as-is:

- **`turtle`** — the real stdlib `turtle` needs Tkinter, which isn't available in a WebAssembly browser build. Instead, `import turtle` / `from turtle import *` loads a custom canvas-backed replacement covering the common API: `forward`, `backward`, `left`, `right`, `goto`, `circle`, `penup`/`pendown`, `color`/`pencolor`/`fillcolor`, `begin_fill`/`end_fill`, `write`, `Screen()`, and the usual short-form aliases (`fd`, `bk`, `lt`, `rt`, ...).
- **`p5`** — not a real PyPI package. Detected when your code contains `import p5` / `from p5 import` or a `def draw():` function, this loads [p5.js](https://p5js.org) from CDN and exposes it as a Python module, so `p5.background()`, `p5.ellipse()`, `p5.fill()`, etc. forward straight through to the JS library.

**Not supported:** anything needing real OS features — `tkinter` (GUI), `socket` (raw networking), `subprocess`, `multiprocessing`, real threads, or the real filesystem (there's an in-memory virtual filesystem instead, used for the input/output file feature). Third-party PyPI packages (numpy, pandas, etc.) aren't loaded — there's no `pip`/`micropip` wiring, so only what ships with Pyodide's base distribution is available.

`input()` works, but since a browser tab can't truly pause a running script, it's implemented via a native `prompt()` dialog — see [colophon.md](colophon.md) for why.

`time.process_time()` doesn't work — it always reads `0.0`, even across genuine CPU-bound work, because it relies on OS-level CPU accounting that doesn't exist inside a WebAssembly sandbox. Use `time.time()` or `time.perf_counter()` for timing code instead; both work correctly.

There's currently no way to stop a running program from the page — an infinite loop or a long `time.sleep()` will freeze the tab, since Python runs on the same thread as the page itself. Reload the tab to recover.

## Running it locally

There's nothing to install. Open `index.html` directly in a browser, or serve the folder with any static file server if you prefer.

## License

[MIT](LICENSE) — use it, modify it, teach with it, no strings attached.
