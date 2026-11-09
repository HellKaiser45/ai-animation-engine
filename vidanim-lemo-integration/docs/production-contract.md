# Agent-facing animation page contract

This is a lightweight convention for pages that need deterministic frame capture, review or export. It is not a claim that the repository already contains a universal renderer, audio mixer or headless export CLI. Implement only the fields the project needs.

## Recommended globals

| Global | Required for capture? | Contract |
|---|---:|---|
| `window.DUR` | Yes | Duration in seconds, finite and greater than zero. |
| `window.FPS` | Recommended | Intended output frame rate. |
| `window.READY` | Yes | Set to `true` only after fonts, images, geometry and other required resources are ready. |
| `window.render(t)` | Yes | Draw the requested time `t` in seconds, independent of previous calls. |
| `window.EV` | Optional | Declarative event markers sharing the visual timeline. |
| `window.TEXTS(t)` | Optional | Readable text visible at `t`, with display/raster-space bounding boxes. |
| `window.PROJECT` | Optional | Project metadata such as width, height, fps, duration, seed and coordinate-space notes. |

## Minimal implementation pattern

~~~js
const DURATION = 12;
const FPS = 30;

window.DUR = DURATION;
window.FPS = FPS;
window.READY = false;
window.EV = [];

// The scene evaluator is pure: same t + same inputs = same state.
function evaluate(t) {
  return {
    time: Math.max(0, Math.min(DURATION, t)),
    // derive camera, object transforms, lights and effects here
  };
}

function draw(state) {
  // Clear and redraw the complete frame from state.
}

window.render = function renderAt(t) {
  const numericTime = Number(t);
  const safeTime = Number.isFinite(numericTime)
    ? Math.max(0, Math.min(DURATION, numericTime))
    : 0;
  draw(evaluate(safeTime));
};

window.TEXTS = function visibleTextsAt(t) {
  // Return [] when no text needs a read-time check.
  return [];
};

// Set only after required resources and scene setup have completed.
window.READY = true;
~~~

The names are conventions, not renderer APIs. A project may keep different internal function names and adapt them through a small wrapper.

## Determinism requirements

- `render(5)`, `render(2)`, `render(5)` must produce identical pixels for both calls at 5 seconds.
- Clear or fully overwrite the output buffer on every call; do not rely on what the previous frame drew.
- Derive all animation values from `t`, not from accumulated delta time, wall-clock time or the number of preview ticks.
- Seed procedural randomness once from project metadata. Do not call unseeded random functions from frame evaluation.
- Preview playback may use `requestAnimationFrame` to choose a time, but it should call the same deterministic `render(t)` used for seeking and capture.
- Any asynchronous resources must finish before `READY` becomes true. If resources can load later, update readiness deliberately and expose an error state rather than rendering silently incomplete frames.

## Event convention

`EV` is an optional list of stable, time-based cues. Use seconds from the start of the film and stable IDs. A recommended shape is:

~~~js
window.EV = [
  { t: 1.25, type: "visual", id: "title-lockup" },
  { t: 4.00, type: "sfx", id: "transition-whoosh" },
  { t: 8.50, type: "voice", id: "line-02" }
];
~~~

The event list is descriptive data. It does not itself synthesize sound or create subtitles. A separate production tool may consume it, or an animation may use it for its own event-driven effects. Keep the schema stable within a project and do not invent audio outputs that have not been rendered.

## Text timing convention

`TEXTS(t)` should return every text item that is intended to be read and is visible at time `t`:

~~~js
window.TEXTS = (t) => [
  {
    id: "chapter-title",
    text: "A new beginning",
    x0: 420,
    y0: 120,
    x1: 1500,
    y1: 230
  }
];
~~~

- Use stable IDs. A genuinely new text item gets a new ID.
- Coordinates are pixels in the animation's declared output/display coordinate space, not CSS percentages or world units.
- Return the full intended text even during a typewriter reveal; do not report only the currently typed prefix.
- Mark passing signage or deliberately unreadable background text with `pass: true` only when a downstream checker supports that convention.
- The text checker / export utility is optional tooling; implementing `TEXTS` does not mean such a checker already exists.

## Project metadata

A useful optional metadata object:

~~~js
window.PROJECT = {
  title: "Example film",
  width: 1920,
  height: 1080,
  fps: 30,
  duration: 12,
  seed: 42,
  coordinateSpace: "output-pixels"
};
~~~

Declare actual values. If the internal drawing buffer is lower-resolution than the output, record that separately (for example `renderWidth` and `renderHeight`) and document the upscale method. Do not imply that a 512×288 buffer has native 1920×1080 detail simply because its CSS display size is larger.

## Validation recipe

1. Confirm `DUR`, `READY` and `render` exist before capture.
2. Call `render(0)`, `render(DUR / 2)`, `render(DUR)` and inspect the output.
3. Call `render(3)`, `render(1)`, `render(3)` and compare the two 3-second frames for determinism.
4. Seek to beats around camera cuts, reveals, collisions and text changes.
5. Check that every `TEXTS(t)` box fits the output frame and that text remains long enough to read.
6. Check all `EV` timestamps fall within `[0, DUR]` and their IDs are unique.
7. Check preview and export use the same evaluation function and coordinate-space assumptions.

No test should be reported as passed unless it was actually run.
