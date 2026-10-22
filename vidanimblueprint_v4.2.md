# VidAnimEngine — Blueprint v4.2 (AI-first, lightweight, deterministic rendering)

## Purpose

VidAnimEngine is a small, asset-free animation engine that an AI agent can understand and author without scaffolding a framework. It creates bespoke 2D, 2.5D and procedural 3D animations using vanilla HTML, CSS and JavaScript.

**Default deliverable: one well-written HTML file.** It should contain the scene, CSS, JavaScript, timeline and rendering logic needed for that animation. Split code only when doing so makes the animation easier to understand, edit or reuse.

No external libraries, model downloads, texture downloads, WebGL or WebGPU are required. Built-in browser APIs such as SVG, Canvas 2D, typed arrays, `ImageData` and audio APIs are allowed.

The goal is not to build a general-purpose framework for every possible future use. The goal is to make the requested animation look right, run smoothly, remain easy to edit, and be reliable when an AI agent creates or revises it.

---

## 1. Non-negotiable principles

1. **Keep projects small.** Prefer one HTML file. Do not create a module, class, folder or abstraction unless it solves a real problem in the current project or is genuinely reusable.
2. **One coherent world.** All objects—including text, diagrams, terrain and 3D shapes—belong to the same world and use the same active camera/projection rules. Use object depth and camera-facing orientation when a graphic needs to read clearly; do not create a separate overlay world.
3. **One source of time.** An animation state must be computable from the requested frame/time, not depend on how long the page has been running.
4. **Procedural by default.** Create shapes, geometry, materials and stylized textures from code or SVG descriptions. Do not require external assets.
5. **Bespoke style.** Design each animation for the user's request. Reusable operations are encouraged; forced visual templates are not.
6. **Optimize by avoiding work.** Reuse unchanged data, share vertices, cull invisible objects, batch compatible drawing and reduce detail when it cannot affect the image.
7. **Choose the simplest correct renderer.** Use SVG when it is a good fit. Use Canvas 2D/raster output when dense geometry makes SVG expensive or painter ordering visibly fails. ASCII is an optional style/output mode, not a mandatory pipeline.
8. **Do not claim unmeasured speedups.** Benchmark comparable scenes before declaring one method faster.
9. **Fail clearly.** If a shape cannot be triangulated or visibility cannot be resolved reliably, report the problem or choose an explicit fallback; do not silently draw a plausible but wrong result.
10. **Budget by measured time.** Performance gates are based on measured rendering time, not on screen area alone. Screen-space size may guide visual LOD, but is not a substitute for profiling.

## 2. The simple mental model

The engine has three jobs:

1. **Director:** given a frame, computes animation values and the camera.
2. **World:** stores objects, procedural geometry, transforms and materials.
3. **Renderer:** draws the world using the most suitable output method for the scene.

These are conceptual responsibilities, not a requirement to create three files. In a small animation, all three can live in `index.html`.

Every visible object follows the same sequence:

`local geometry → world transform → camera transform → clipping/projection → visibility → shading → output`

The `director` owns time. The renderer must not use independent timers to decide what belongs in a frame.

## 3. One world, one camera

- Use one world coordinate system and one active camera per composition.
- Perspective and orthographic are modes of the same camera system, not separate animation engines.
- Flat graphics and text can be modeled as flat world geometry or camera-facing world objects. They still have a world position and obey the shared camera and depth rules.
- If a graphic must sit in front, set its depth and orientation intentionally. A small depth bias is allowed only to resolve intentional near-coplanar ties.
- CSS `z-index` orders DOM layers; it does not resolve individual 3D surfaces. Do not use it as the general depth algorithm.
- Screen-pinned patterns, such as the ASCII lattice, may be anchored in viewport coordinates after the underlying world geometry is projected. This is a texture-coordinate rule, not a second world.

### Coordinate-space contract

Use these five named spaces explicitly. Do not mix values from different spaces just because they are all represented by JavaScript numbers.

1. **Local space** — coordinates relative to an object's own origin and orientation.
2. **World space** — shared scene coordinates after the object's model transform. All objects that interact spatially use this same world.
3. **Camera space** — coordinates after the world-to-camera transform and before perspective division. Camera-space depth determines projection and visibility calculations.
4. **Raster space** — coordinates in the discrete working render buffer. Raster coordinates are measured in raster pixels/samples; a supersampled buffer is larger than the resolved display buffer.
5. **Display space** — logical coordinates of the final composition. In an interactive preview, the resolved display buffer tracks the surface's CSS content-box dimensions in CSS pixels (round fractional dimensions consistently to integer buffer dimensions). In video export, it tracks the explicitly selected output frame dimensions.

The normal conversion chain is `local → world → camera → raster → display`. A vector/SVG renderer may project directly from camera space into display-space coordinates when it does not use a raster working buffer; the coordinate conversion must still be explicit.

Rules for projection constants and audits:

- Every projection-related constant must have a nearby comment declaring its **space and units**, for example `focalRasterPx` (raster pixels), `centerDisplayCssPx` (display CSS pixels), `cameraZ` (camera-space world units), or `rasterPixelsPerCssPixel` (raster pixels per display CSS pixel). Prefer names that encode space/units when they prevent ambiguity.
- Every use of a focal-length, viewport-size, raster-size, display-size, center-offset or scale constant in a projection/conversion expression must have a **one-line audit comment** identifying the source space, destination space, units and conversion. Keep it directly beside the expression.
- Never reuse a raster-space focal length as a display-space focal length. Derive the correct value for the target space from the same camera FOV, or explicitly convert it.
- Treat `devicePixelRatio` as an explicit presentation/render-scale input, not an invisible change to world or camera coordinates. It must not silently alter focal length, world scale or camera motion.

Example:

```js
// RASTER -> RASTER: rasterHeightPx [raster px] and verticalFovRad [radians] define focalRasterPx [raster px].
const focalRasterPx = 0.5 * rasterHeightPx / Math.tan(verticalFovRad * 0.5);

// CAMERA -> RASTER: xCamera / zCamera is dimensionless; focalRasterPx and centerRasterPxX are [raster px].
const xRasterPx = centerRasterPxX + focalRasterPx * (xCamera / zCamera);

// RASTER -> DISPLAY: xRasterPx [raster px] divided by rasterPixelsPerCssPixel [raster px / CSS px] gives [display CSS px].
const xDisplayCssPx = xRasterPx / rasterPixelsPerCssPixel;
```

The names above are illustrative, not required APIs. The important requirement is that an AI agent can audit each projection constant and conversion without guessing which coordinate system it belongs to.

## 4. Deterministic animation

Use a single frame/time convention, for example:

```js
const time = frame / fps;
const state = evaluate(time, scene);
render(state);
```

Rules:

- The same scene and requested frame must produce the same state.
- Seeking, replaying, still-frame rendering and video export use the same evaluation logic.
- Do not use `Math.random()`, `setInterval()` or wall-clock callbacks inside frame evaluation. Use a seeded random generator for procedural variation.
- Preview playback may use elapsed time to choose which frame to request, but scene state must be calculated from that frame/time.
- Audio cues must derive from the same timeline as the visuals.
- Use smooth interpolation appropriate to the value being animated. For camera routes, centripetal Catmull–Rom is a sensible default; use arc-length sampling when roughly constant travel speed matters. For scalar values that must not overshoot, use monotone cubic Hermite interpolation. Do not apply one spline type indiscriminately to every value.

## 5. Geometry: create once, reuse often

### Indexed geometry

Represent meshes with a vertex buffer and triangle index buffer. A triangle references three vertices; it should not store duplicate copies of shared vertices unless there is a reason to split them (for example, a hard normal/material boundary).

Transform each unique vertex once per relevant change, then reuse its result for all connected triangles. For 20,000 triangles, this can reduce repeated transform work substantially when vertices are shared.

### Cache only what is useful

Cache compiled SVG paths, triangulation, local normals, bounds, material IDs and reusable primitive meshes. Do not rebuild geometry during animation unless its shape actually changes.

Invalidate derived data only when an input it depends on changes:

- geometry changes → rebuild geometry and dependent data;
- object transform changes → update world positions/bounds and dependent projection;
- camera changes → update projection and visibility, not SVG parsing or local geometry;
- light/material changes → update affected shading, not topology;
- unchanged inputs → reuse cached output where practical.

Keep this dependency logic straightforward. Version numbers or dirty flags are enough; do not introduce a complex cache framework without evidence that it is needed.

### Repeated objects

For repeated trees, rocks, bricks, leaves or stars, compile one base shape and reuse it with different transforms. Keep spatial chunks and bounds so distant or invisible groups can be skipped. Avoid merging the entire world into one un-cullable mesh.

## 6. The cheapest useful frame

Use this order to avoid unnecessary work:

1. Reject whole objects/chunks outside the camera view.
2. Reuse unchanged geometry, transforms, bounds and materials.
3. Transform each unique vertex only as needed.
4. Clip triangles against the near plane before perspective division; do not simply drop every triangle that crosses it.
5. Reject backfaces when the material is one-sided.
6. Remove degenerate geometry and simplify distant detail when doing so preserves important silhouettes.
7. Shade using quantized buckets and precomputed palettes where suitable.
8. Resolve visibility using the cheapest method that is correct for the scene.
9. Batch compatible output and update only what changed.

Use projected size as a rough level-of-detail guide. A tiny face may be merged, simplified or omitted if it cannot visibly affect the image, but protect silhouettes, readable text, hero objects and details that matter to the requested style.

## 7. Scene correctness: contact, attachment, layout and visibility

These rules apply to every animation, not to one particular world or art style. They apply to 3D scenes and to 2D/2.5D compositions wherever objects have spatial relationships, alignment, containment or overlap.

**Visual plausibility is part of correctness.** A page that runs without errors is not finished if objects unintentionally float, sink, detach from their targets, cross in the wrong order, disappear through surfaces, or violate the composition's layout.

### One source of truth for geometry

- The geometry used to position, attach, clip or test an object must agree with the geometry the renderer actually displays.
- Do not place an object using a continuous or approximate formula if the visible surface is a coarser triangulated mesh and the two can disagree.
- For mesh surfaces, derive contact/height queries from the final rendered triangles (or from the exact same triangulation and interpolation rules). For procedural surfaces, either use the same sampling to build and query the surface, or deliberately refine/modify the mesh around attached objects.
- Keep shared boundaries coherent. Adjacent terrain patches, walls, roads, water, platforms, panels or shapes must not develop accidental gaps, overlaps or mismatched edges because each was generated from a different approximation.
- Keep this lightweight: use direct mesh queries, simple bounds, or a few samples where sufficient. Do not add a full physics or collision engine unless the requested animation actually needs one.

### Attachment and support contracts

When an object is intended to rest on, attach to, follow, emerge from, or connect to another object, author that relationship explicitly. This is a design rule, not a requirement for a complex attachment framework.

- Identify the support/parent/target and the relevant anchor or contact points.
- For grounded or supported objects, validate more than the center when the footprint is wide. Sample corners, edges or representative contact points as appropriate.
- Align the object's base to the intended surface, or use an explicit foundation, socket, mounting point, or conforming base when that better fits the design.
- For uneven supports, choose deliberately whether the object follows the surface normal, stays upright, uses a level platform, or is embedded. Never let this behavior emerge accidentally from mismatched coordinates.
- For connected objects—labels and callouts, cables, pipes, roads, bridges, limbs, doors, moving parts—derive their endpoints and transforms from the same source anchors. Do not duplicate unrelated coordinates that can drift apart.
- Intentional embedding, intersection, floating, clipping or suspension is allowed when it is visibly purposeful. Mark or design it as intentional; do not treat every overlap as a defect.
- Check both the beginning and end of motion. An object may be correctly placed in its initial pose but penetrate, detach or drift during animation.

### 2D and 2.5D layout contracts

The unified world rule still applies. These checks do not create a separate overlay world.

- Anchor related graphics to shared objects or bounds when they are meant to stay connected. Examples include labels to nodes, arrows to their endpoints, subtitles to a camera-facing world panel, and diagrams to their parent group.
- Use consistent parent transforms for parts that must move together. Do not animate connected parts with independent hard-coded positions unless their relative motion is intentional.
- Define intended containment and clipping. Text or graphics should not accidentally escape their panel, diagram region or viewport. Use a clip path or explicit bounds where needed.
- Make overlaps and reading order intentional. If a label must remain in front of a world object, position/orient it intentionally in world space and ensure the chosen visibility method can actually preserve that ordering.
- When text must remain readable while the camera moves, use a camera-facing world object or another explicit orientation rule, but keep its position and depth in the same world projection.
- Check layout at the actual output aspect ratio and key camera positions, not only at the authoring viewport.

### Visibility must be resolved, not merely reported

- Every visible opaque pixel/region must have a correct frontmost surface according to the scene's intended depth ordering.
- Painter sorting by triangle-centre depth is a fast path, not a universal visibility solution. If intersecting or cyclic geometry makes the order ambiguous, do not leave it unresolved after logging a warning.
- Resolve ambiguity using the smallest effective correction: improve/split geometry, split only the problematic polygons, adjust an intentionally coplanar surface with a small documented bias, or use a raster/depth-buffer path when ambiguity is widespread.
- Never use CSS `z-index` as a substitute for per-surface depth in the unified 3D world. It orders DOM elements, not the surfaces inside them.
- Water, glass, fog, particles and other transparent effects need an explicit compositing rule. A technique that works for opaque faces may not work for transparent surfaces.
- For 2D scenes, the same principle means the intended layer/reading order must be explicit and stable. Accidental occlusion, flicker or elements switching order during movement is a correctness failure.
- A nonzero ambiguity counter is a diagnostic signal, not automatic failure by itself. The agent must inspect whether those cases affect the rendered image. Any visible wrong overlap must be fixed or the renderer/path must change before the scene is considered complete.

### Scene correctness is separate from scene style

A scene can be sparse by artistic choice, stylized, surreal or abstract. Do not force realism or add detail that the user did not ask for. But enforce the relationships that the chosen style implies: a grounded object should look grounded, a connected line should meet its target, a moving part should remain attached, and intentional overlaps should be visually legible as intentional.

## 8. Visibility and rendering

SVG is not automatically cheap, and Canvas is not automatically faster. Choose by scene properties and measure when performance matters.

### SVG fast path

Use SVG for vector illustrations, typography, diagrams, paths and sparse or stylized low-poly scenes.

- SVG does not provide general per-pixel 3D depth testing. For opaque meshes, use a painter's order only where it is valid enough for the scene.
- Detect likely ordering ambiguity for intersecting/cyclic geometry. For simple cases, split a problematic face or change the scene representation. If overlap complexity becomes widespread, use the raster path rather than adding endless painter-order exceptions.
- Batch compatible faces into fewer paths when doing so preserves required order and material. Faces can be subpaths in the same `<path>`.
- Reuse SVG elements. Update only changed attributes such as `d`, `transform`, or `fill`; avoid recreating the SVG subtree every frame.
- A single path with many subpaths reduces DOM element count but does not remove path parsing, tessellation or rasterization cost. Split extremely large batches if profiling shows they are slower.
- Keep static geometry separate from dynamic geometry so static output can remain untouched.

### Canvas/raster path

Use Canvas 2D when a dense scene is a poor fit for many SVG paths. For opaque 3D scenes that need robust intersections, a small software rasterizer can use a color buffer plus a Z-buffer.

- This is optional, not a requirement for every animation.
- Use typed arrays and reusable buffers. Avoid allocating new arrays/objects per face per frame.
- A Z-buffer compares depth per pixel, so it handles intersecting opaque surfaces better than sorting each triangle by its centre.
- A software Z-buffer costs CPU time and pixel memory. Use a reasonable internal resolution and scale up for a stylized appearance when appropriate.
- **Display-buffer sizing:** in interactive preview, the resolved display buffer must track the surface's CSS content-box width and height in CSS pixels. When the CSS size changes, update the dimensions explicitly. Do not confuse the resolved display buffer with the higher-resolution working raster. For final video export, the selected output frame width and height define the resolved display buffer; browser CSS dimensions do not override the export profile.
- **Integer supersampling for raster antialiasing:** use an explicit integer scale `S >= 1`. The working raster is exactly `displayWidth * S` by `displayHeight * S`. After visibility and color resolve, each display pixel is produced by an exact box downsample: average the corresponding `S × S` samples, with every sample in that block contributing equally. There is no fractional scale or unspecified resampling filter in this path. For transparency, average premultiplied color and alpha to avoid edge halos; use a consistent, documented color space for the averaging. `S = 1` disables supersampling.
- Supersampling is the defined antialiasing method for the software raster path, not a requirement to rasterize native SVG or ASCII output. Keep intentional pixel-art/ASCII styles at their authored resolution. Do not silently add both a device-pixel-ratio multiplier and `S`; any additional presentation scale must be explicit and documented.
- If vector and raster objects can occlude each other, they must share a compatible depth/visibility result or be rendered together through one appropriate path. Do not just stack independent outputs and assume occlusion will be correct.

### ASCII path

ASCII is a visual style and a low-resolution output mode. It can be efficient because it limits output to a fixed grid, such as 160×90, and can write one `textContent` string per frame. It does **not** remove the need for projection, visibility and shading. Use a coarse depth buffer or an appropriate triangle rasterization strategy to determine which surface contributes to each cell.

### Shading

The documented 32-bucket Blinn–Phong-inspired shading is CPU math, not a GPU shader. Keep it if it supports the visual style.

- Precompute final colour strings for `(material, shade bucket, fog bucket)` combinations.
- Quantize illumination and use palette lookups rather than building colour strings for every face.
- Approximate expensive specular functions with a small lookup table when the visual difference is acceptable.
- Cache a face's shade bucket while the geometry orientation, light and material inputs remain unchanged.
- Do not recalculate static lighting simply because the camera moved, unless the shading model is camera-dependent.

## 9. Performance model: geometry, output and pixel costs

Do not treat “rendering cost” as one undifferentiated number. Separate work whose cost is mostly independent of output resolution from work that grows with the number of pixels/samples, and measure vector-output work separately. This makes optimization targeted rather than guesswork.

### A. Resolution-independent geometry/face pass

For fixed geometry and a fixed visibility/LOD selection, this pass is driven mainly by unique vertices and faces—not directly by raster width and height. It includes model/world/camera transforms, bounds tests, clipping, backface tests, face normals, per-face shading and visibility ordering.

- Transform each unique indexed vertex once per invalidation, then reuse it for all referencing faces.
- Cache local normals, material IDs, bounds and other values while their dependencies remain unchanged.
- Note that algorithms still matter: painter sorting is commonly `O(F log F)`; a Z-buffer changes the visibility work but adds sample/depth-buffer costs.
- View-dependent culling and LOD selection can change how many vertices/faces reach the pass, but do not confuse that with a screen-area performance cap.

### B. Output/composition pass

This cost is driven by what must be emitted and processed: SVG path count, subpath count, path-string length, attribute updates, DOM parsing, and renderer flushes. It is neither purely face math nor purely per-pixel work.

- Batch compatible faces where order and material permit, and reuse persistent output nodes/buffers.
- Track path count and output size alongside timing. A smaller DOM can still contain an extremely expensive path, so element count alone is not a complete cost measure.
- If output costs dominate, reduce redundant markup and improve batching before reducing geometric detail blindly.

### C. Pixel/sample pass

This cost is driven by the working raster dimensions, covered samples, depth/color writes, full-buffer clears, per-pixel effects and antialiasing. For full-buffer operations, cost scales roughly with raster width × raster height. With integer supersampling, the working sample count grows approximately as `S²` for a fixed display size. Covered-triangle work additionally depends on the number of samples the triangles actually cover.

- Measure buffer clearing, rasterization/depth tests, per-pixel effects and box downsampling separately when practical.
- Use reusable typed arrays/buffers; avoid allocating a new color/depth buffer every frame when dimensions have not changed.
- If sample costs dominate, reduce the supersample factor by an integer step or reduce expensive pixel effects before discarding important scene geometry.

### The performance gate is a measured time budget, never an area cap

- Define the interactive target from the requested frame rate: `frameBudgetMs = 1000 / targetFps` (for example, about 16.67 ms at 60 fps or 33.33 ms at 30 fps). Treat this as a target, not a guarantee on every machine.
- Measure actual elapsed time with `performance.now()` or an equivalent monotonic timer. Keep separate measurements for geometry/face work, output/composition, pixel/sample work and total frame time. Use a short rolling window or smoothed statistic plus hysteresis so one noisy frame does not make quality oscillate.
- Adapt preview quality in response to the measured bottleneck: cull/reuse/simplify work when geometry dominates; batch/reuse output when SVG/DOM dominates; reduce integer supersampling or expensive pixel effects when sample work dominates. Preserve requested objects, hero silhouettes, readable text and other important visual details.
- **Do not use projected area, bounding-box area or a face's screen area as the pass/fail performance budget or as a hard cap that decides whether the scene is allowed to render.** Projected size may still guide visual LOD/culling when the detail genuinely cannot affect the image, and hard memory/buffer-size safety limits are allowed; these are distinct decisions, not substitutes for measured runtime.
- Runtime measurements may adapt interactive-preview quality, but they must never alter animation state or determine final-export quality implicitly. Final export uses explicit, fixed resolution, frame rate, supersample setting and quality options so the result remains reproducible across machines.
- Do not claim optimization succeeded until a comparable benchmark shows it. Test the same scene, resolution and quality, and report the bottleneck that actually improved.

## 10. SVG as a source for complex 3D shapes

SVG is a compact way to describe silhouettes and profiles. Compile it once into geometry, then animate the resulting object.

Provide a small number of useful operations and expand only when needed:

- **Flat path:** render a filled or stroked SVG path in the world.
- **Extrude:** triangulate a closed silhouette, create front/back caps and side walls, and give it depth.
- **Lathe/revolve:** rotate a radius/height profile around an axis for columns, towers, pots and domes.
- **Sweep:** move a profile along a 3D path for pipes, roots, rails and horns.
- **Loft:** connect compatible contours for organic or architectural forms.
- **Ribbon/tube:** turn a stroke into a strip or tube.
- **Layered relief:** place several SVG shapes at deliberate depths for 2.5D forms in the same world.

Geometry rules:

- Parse and flatten Bézier curves only when compiling or changing the SVG, not every frame.
- Make curve subdivision adaptive to shape curvature and desired output quality; apply safe limits to prevent runaway tessellation.
- Triangulate polygons with a defined winding and hole policy. If holes or self-intersections are unsupported, report that clearly rather than filling them incorrectly.
- Generate side-wall normals and cap normals intentionally; sharp edges may require duplicated vertices with different normals.
- Test winding, concave outlines, holes, degenerate paths and near-plane clipping.
- Expose only the quality controls that are useful to the author, such as curve detail, depth and bevel. Use safe defaults for the rest.

## 11. SVG, ASCII and frame-writing rules

- Do not create thousands of DOM nodes inside the animation loop.
- Keep persistent SVG paths, patterns, `<pre>` and canvas elements.
- Skip writes when the resulting value is unchanged.
- Batch compatible faces with string concatenation in hot loops; avoid temporary arrays and repeated `.map().join()` where profiling confirms they cause pressure.
- Reuse typed arrays and output buffers.
- A single DOM write is not free: replacing `innerHTML` recreates nodes and causes the browser to parse markup. Prefer updating retained nodes or swapping a compact batch only when necessary.
- Keep static and dynamic content separate for cacheability, while respecting the shared world and depth rules.

## 12. AI-agent authoring contract

The agent should spend its effort on the requested animation—not framework scaffolding.

1. Start with one `index.html` containing the scene, CSS, JavaScript and animation timing.
2. Build bespoke geometry and choreography for the user's prompt. Use a helper or component only when it makes the code clearer or is used more than once.
3. Reuse engine operations rather than copying long blocks of triangulation, projection or shading code into the scene.
4. Keep IDs and parameters readable. Prefer explicit names such as `riverPath`, `villageRoof`, `cameraTarget` and `revealStart` over unexplained numeric indices.
5. Keep authored scene data separate from generic helper functions when the separation makes editing easier. Do not split code simply to satisfy a folder pattern.
6. When something is approximate or unsupported, add a clear comment or diagnostic and use a deliberate fallback.
7. After editing, open the animation, inspect its first frame, key moments, close camera passes and end state. Fix actual visual defects before adding more architecture.
8. Check the scene-correctness contracts: supported objects remain supported, attached parts remain connected, shared boundaries meet, intended containment is respected, and visible occlusion matches the scene's intent.
9. Inspect key relationships while the camera and objects move. A single correct still frame is not proof that contact, attachment or visibility remains correct over time.
10. Avoid adding dependencies, build steps, test frameworks or configuration files unless the user explicitly needs them or the project cannot reasonably work without them.
11. When writing projection math, name the five spaces consistently and add a one-line space/unit audit beside every focal-length, viewport-size, raster-size, display-size or scale conversion.
12. When a frame is slow, measure geometry/face, output/composition and pixel/sample stages separately before changing quality. Use elapsed-time budgets—not screen area caps—to guide preview adaptation. Keep final-export quality settings explicit and fixed.
13. For software raster antialiasing, use integer supersampling and exact box downsampling; keep the resolved interactive display buffer aligned to CSS content-box dimensions.

## 13. Default project structure

**Small animation: one folder and one HTML file.** It should be possible to refresh the page and play the animation from a simple local web server such as `npx serve`.

```text
animation-name/
  index.html   # scene + styles + JavaScript; default and preferred
```

If the animation becomes difficult to maintain, an optional small split is allowed:

```text
animation-name/
  index.html
  components/              # only components used by this animation
    component-name/
      index.html           # optional markup/template or demo
      style.css            # optional, only if needed
      component.js         # component behavior
```

If multiple animations genuinely share stable engine code, a small shared file can be introduced deliberately. Do not create a default `engine/` directory with a file for every responsibility. Do not create empty folders or unused modules. A one-off shape, title or effect belongs directly in the animation unless there is a real reuse or clarity benefit.

The folder-per-animation rule is the default. A folder-per-component rule applies when the component is substantial or reusable—not to every primitive or one-line helper.

## 14. Practical validation

Validation should be lightweight and tied to the animation being built. Do not require a large testing framework for every one-off project.

Check at least:

- the page runs from a simple local server with no missing files;
- the first frame, important story beats and final frame render correctly;
- replaying or seeking to a frame produces the same visual state;
- the camera does not jump, flip unexpectedly or lose the subject;
- objects meant to touch or rest on a surface use the same geometry that is rendered, and their full relevant footprint is checked;
- related parts, connectors, labels and moving components remain attached to their intended anchors;
- adjacent regions and surfaces meet without accidental gaps, holes or contradictory overlaps;
- visible depth order is correct, with no unresolved painter-order errors, flickering intersections or unintended clipping;
- in 2D/2.5D, bounds, anchors, clipping and reading order remain correct at key frames and output aspect ratios;
- dense scenes stay within a reasonable measured frame-time and output-size budget;
- every projection constant declares its coordinate space and units, and each use of focal/size/scale constants has the required one-line source-to-destination audit;
- in interactive raster preview, the resolved display buffer tracks the CSS content-box dimensions; resizing updates it explicitly;
- integer supersampling uses a working buffer exactly `S` times display width and height, and the box downsampler maps each `S × S` block to exactly one display pixel; test `S = 1` and at least one `S > 1`;
- transparent downsampling does not introduce edge halos;
- audio remains aligned with the timeline if audio is present.

For performance tuning, compare the same scene at the same resolution and quality. Measure geometry/face time, SVG/output time, pixel/sample time, total frame time, path count/path-string size, visible triangle count, cache hits, memory and export time when available. The preview quality gate must use measured elapsed time; projected area can guide visual LOD but is not a performance budget. Benchmark before claiming that SVG batching, ASCII or raster rendering is faster for a particular scene.

## 15. Implementation priority

Do not implement all features before making useful animations. Build only what the current project needs, in this order:

1. Deterministic timeline and one reliable world/camera path.
2. Indexed geometry and reuse of compiled SVG/primitives.
3. Basic projection, near-plane clipping, winding and culling.
4. Quantized CPU shading and cached palettes.
5. Retained/batched SVG output for sparse scenes.
6. Clear detection of painter-order failures.
7. Optional Canvas/raster depth buffer only when a scene needs it.
8. More advanced SVG extrusion, sweep, loft, holes or bevels when requested by actual scenes.

Do not add a feature merely because it appears in this blueprint. Add it when it improves the requested output or removes a demonstrated limitation.

## Core invariants

- Vanilla HTML/CSS/JS; no mandatory external libraries, models or textures.
- One coherent world, one active camera/projection per composition.
- One deterministic frame/time source.
- CPU-side shading; no WebGL requirement.
- Geometry is compiled once and reused until its inputs change.
- Spatial relationships must use the same authoritative geometry and anchors as the rendered scene.
- Visible depth/overlap errors must be corrected, not merely logged.
- Shared indexed vertices are transformed once per invalidation, not once per triangle corner.
- SVG batching reduces DOM overhead but does not make complex paths free.
- ASCII reduces output resolution but does not eliminate scene calculations.
- Painter sorting is a fast path, not a universal solution for intersecting surfaces.
- CSS `z-index` is not a 3D depth buffer.
- Choose the simplest renderer that correctly serves the scene.
- Prefer one well-written HTML file; split only for real reuse or clarity.
- AI agents build bespoke animations, not fixed templates or unnecessary scaffolding.
- The five coordinate spaces are local, world, camera, raster and display; all projection constants declare space/units and all focal/size conversions are auditable.
- The interactive resolved display buffer tracks CSS content-box dimensions; video export instead follows its explicit output dimensions.
- Software-raster antialiasing uses integer supersampling followed by exact `S × S` box downsampling; supersampling is not forced on native SVG or intentionally pixelated ASCII output.
- Performance decisions distinguish resolution-independent geometry/face work, SVG/output work and per-pixel/sample work.
- The performance gate is measured elapsed time, never a screen-area cap; projected size may guide visual LOD, and hard memory limits remain separate safety checks.
- Interactive preview may adapt quality from measured time, but final-export quality is explicit and fixed.
