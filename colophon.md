# Colophon

Notes on how `pyde` was built, for anyone curious about the mechanics.

## How it was made

This was built conversationally with [Claude Code](https://claude.com/claude-code) (Claude Sonnet 5) — a single back-and-forth session: describe a feature, get a working implementation, test it in an actual browser, fix what didn't work, move to the next feature. No hand-written scaffolding beforehand; the whole thing grew from one prompt describing the nine core features, then iterated from there (multi-cursor editing, comment/indent shortcuts, `input()` support, the Parsons puzzle share link, and finally deployment).

Everything lives in one HTML file on purpose — no build step, no `node_modules`, no bundler. It has to work by double-clicking it, so anything that couldn't be inlined or loaded from a CDN was out of scope.

## Stack

- **[Pyodide](https://pyodide.org)** — CPython 3.12 compiled to WebAssembly, loaded from jsDelivr. This is what actually runs your Python, in-tab, no server round-trip.
- **[CodeMirror 5](https://codemirror.net/5/)** — the editor, with the Python mode, Dracula theme, and the `matchbrackets`/`closebrackets`/`searchcursor` addons (the last one powers the multi-cursor "select next occurrence" command).
- **[lz-string](https://pieroxy.net/blog/pages/lz-string/index.html)** — compresses code into the URL hash for the Share features.
- **[p5.js](https://p5js.org)** — loaded lazily from CDN only when a sketch is detected, so it doesn't cost anything for programs that don't use it.
- Everything else is vanilla JS. No framework, no npm dependencies to manage.

## Things that weren't obvious going in

**Turtle graphics needed reinventing, not importing.** Pyodide ships the real CPython standard library, but `turtle` depends on Tkinter, which needs a real windowing system — not available in a WebAssembly sandbox. Rather than fight that, `pyde` registers its own `turtle` module (`sys.modules['turtle'] = ...`) that implements the common API on top of an HTML canvas. Same idea for `p5` — it was never a real importable package; it's a small Python class whose `__getattr__` forwards straight through to the live p5.js sketch instance running in JS.

**Python name-mangling almost broke the turtle shim.** Early versions referenced the canvas/context via double-underscore-prefixed globals (`__turtle_canvas`). Inside a class body, Python silently mangles any `__name` (two leading underscores, at most one trailing) into `_ClassName__name` — so code that worked at module scope broke the moment it moved into a method. Switching to single-underscore names (`_turtle_canvas`) fixed it; it's a sharp edge worth remembering any time generated Python code defines classes around externally-injected names.

**`input()` can't avoid a popup by pausing the interpreter — but it can avoid one by rewriting the code.** The first version wired `input()` straight to `window.prompt()`, since Python's `input()` blocks synchronously and Pyodide can't suspend an ordinary (non-`async`) call mid-execution without a Web Worker plus `Atomics.wait` (which needs `SharedArrayBuffer`, which needs cross-origin-isolation headers a `file://` page can never get). That held until a student's number-guessing program made the real cost obvious: a native dialog is *modal* — it hides the console entirely, so there's no way to see the previous guess's "too high"/"too low" feedback while typing the next one.

The fix that actually works without a server: don't try to pause the interpreter, rewrite the *code* before running it. Using Python's own `ast` module, every top-level `input(prompt)` call is rewritten into `(yield prompt)`, and the whole script gets wrapped in a generator function. JS then drives that generator — call `.next()`, get back the yielded prompt, show a plain inline `<input>` in the console (not a dialog), wait for Enter, resume with `.send(value)`. Because this is cooperative and JS-driven, the tab stays responsive and the full scrollback stays visible throughout, which is the actual thing that was broken.

This only works when every `input()` call sits at literal top-level code — inside loops and conditionals is fine (they don't introduce a new scope), but one nested inside a `def`/`lambda`/`class` would need each such function turned into a generator too, and every call site turned into `yield from`, transitively, for the whole call graph. That's a real compiler problem, not a quick extension, so the transform instead does a single safety check first: if any `input()` call is nested inside a function, it declines entirely and the code runs through the old `window.prompt()` path unchanged. Same story for turtle and p5 sketches, which are excluded outright — both rely on `setup()`/`draw()` (or user code) existing as real module-level globals afterward, which wrapping everything in a generator function would break, since those names would become local to the wrapper instead.

**Testing an artifact preview and testing a real browser aren't the same thing.** The preview environment used while building this loads local files as `data:` URLs, which have an opaque origin and disable `sessionStorage`/`localStorage` — and Pyodide's loader touches `sessionStorage` on startup. A small storage shim (falling back to an in-memory `Map` if the real thing throws) papers over that. It has zero effect on how the file behaves for anyone opening it normally as `file://`, but without it, testing inside that preview tool wasn't possible at all — worth remembering that a sandboxed preview's quirks aren't always the deployed page's quirks.

**Deployment turned out to need less than expected.** The plan was: new repo for the source, then copy the built file into the existing `milesberry.github.io` repo to get it under `milesberry.net/pyde`. Turns out GitHub Pages already handles this — once a user's root site has a custom domain attached, any other repo's project Pages site is automatically served under `<that domain>/<repo-name>/` too, no extra configuration. So `pyde` stayed its own clean repo, and `milesberry.net/pyde/` just worked.

## What's deliberately not here

No package manager, no transpilation, no telemetry, no dependency beyond what's fetched from a CDN at load time. If a browser can run it, it runs.
