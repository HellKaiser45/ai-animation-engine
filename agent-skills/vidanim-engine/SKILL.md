---
name: vidanim-engine
description: Create, edit, debug, and optimize bespoke animations with VidAnimEngine. Use for procedural 2D/2.5D/3D animation, SVG-derived geometry, camera journeys, and timeline-based motion using this repository's vanilla HTML/CSS/JavaScript engine.
---

# VidAnimEngine — AI authoring skill

## Source of truth
Before substantial work, inspect `vidanimblueprint_v4.2.md` and the closest working example in `examples/`. Verify APIs in actual code; the blueprint describes goals and optional capabilities, not proof that every feature is implemented.

Repository: https://github.com/HellKaiser45/ai-animation-engine
Examples: `examples/3d-world-tour.html`, `examples/Aethra — a 60 second journey.html`, `examples/Setup sans indicateurs.html`

If code and blueprint disagree, do not invent APIs or silently claim support. Follow the working implementation where necessary, report the discrepancy briefly, and use the smallest safe fallback.

## Core rules
1. **Default to one self-contained HTML file per animation.** Include markup, CSS, scene, timing and JavaScript. Split only for genuine reuse or clarity; do not scaffold a framework.
2. **No required external libraries or assets.** Use vanilla HTML/CSS/JS and built-in browser APIs. Generate geometry and stylized detail procedurally or from authored SVG. Do not add Three.js, WebGL/WebGPU, downloaded models/textures or build tooling unless the user changes this constraint.
3. **One coherent world and active camera/projection.** Text, diagrams, flat shapes and 3D objects share world/camera/depth rules. Use intentional position, depth and camera-facing orientation, not a separate overlay world. CSS `z-index` is not a 3D depth solution.
4. **Bespoke style, not a forced template.** Infer the requested art direction, pacing, hierarchy and visual density. Reuse operations, not mandatory scene templates.
5. **One deterministic time source.** Any requested frame/time must produce the same state. Preview may use elapsed time to select frames, but animation state derives from frame/time. Seed procedural randomness.
6. **Visual correctness is part of completion.** A page that runs is not done if objects float, sink, detach, flicker, intersect incorrectly or hide important content.
7. **Optimize by avoiding repeated work.** Cache compiled geometry, transform shared vertices once, batch compatible SVG output, and measure before claiming speedups.
8. **Choose the simplest correct renderer.** SVG for sparse vector scenes; Canvas/raster for dense geometry or when painter ordering visibly fails. ASCII is optional and reduces output resolution, not scene-calculation cost.
9. **Do not overbuild.** No mandatory engine directory, class hierarchy, test framework or generalized abstraction for a one-off animation.
10. **Validate rendered output.** Inspect key frames and motion, not just source code and logs. Never claim tests or visual inspection that did not happen.

## Token-efficient workflow
### 1. Inspect narrowly
- Read the relevant blueprint section and one closest example.
- Search for exact functions/constants to change; do not read or rewrite the entire engine without reason.
- Reuse existing conventions. Verify uncertain capabilities in code.

### 2. Make a compact shot plan
Identify style/palette/light, subject, story beats, camera path/framing, duration/aspect ratio/audio constraints, and likely risks (grounding, overlaps, readability, reveal timing). Keep this short unless the user asks for a full plan.

### 3. Build the smallest complete version
- Start with one HTML file.
- Establish camera, major silhouettes, composition and timing before decoration.
- Use meaningful names for important objects and beats.
- Add detail only when it improves scale, story, depth or readability.
- Create helpers only for repeated behavior or genuine clarity.

### 4. Preview and correct
Inspect the first frame, key beats, close camera passes, transitions and ending. Test seeking/replay if supported. Fix high-impact issues—composition, silhouette, camera, grounding, visibility—before adding polish. Re-check after substantial edits.

### 5. Deliver honestly
State what changed, how to run it, what was actually tested, and known limitations.

## Scene and motion
### Composition and camera
- Establish a readable silhouette and focal hierarchy before micro-detail.
- Camera movement must serve the subject. Avoid unexplained speed changes, clipping through geometry, or revealing a planned hero landmark too early.
- For journeys, connect regions through continuous camera motion, terrain, waterways, paths, lighting or other visual cues.
- Centripetal Catmull–Rom is a reasonable camera-route default; use arc-length sampling for roughly constant speed. Use monotone cubic Hermite for scalar tracks that must not overshoot. Do not apply one spline indiscriminately.
- Use seeded randomness.

### Geometry, grounding and attachments
- **The displayed surface is authoritative.** Position objects using the same mesh/triangulation/interpolation actually rendered. Do not use a smooth height formula when the visible surface is a coarse triangle mesh that differs.
- For wide grounded objects, check corners/edges or representative footprint points, not only the center. Use pads, foundations, conforming bases or deliberate tilt where appropriate.
- Adjacent surfaces (terrain patches, roads, riverbanks, water, platforms, walls) must have coherent boundaries; avoid gaps, contradictory overlaps and buried edges.
- Derive connected parts from shared anchors: labels to targets, pipes to endpoints, doors to hinges, bridges to banks, limbs to joints. Avoid duplicated coordinates that drift.
- Intentional floating, embedding, intersections or clipping are allowed when purposeful. Accidental ones are defects.
- Check relationships throughout motion, not only at the initial pose.

### Visibility and 2D/2.5D layout
- Painter sorting by triangle-center depth is only a fast path; it cannot reliably solve all intersecting/cyclic overlaps.
- If visible ordering is wrong, correct geometry, split problematic faces, apply a small documented bias only for intentional coplanar ties, or use a supported raster/depth-buffer path if ambiguity is widespread. Do not merely log a warning and leave the image wrong.
- Never use CSS `z-index` to solve per-face 3D visibility.
- Transparent water, glass, fog and particles need an explicit compositing rule.
- In 2D/2.5D, preserve intended reading order, anchors, containment and clipping throughout motion.

### SVG-derived geometry
Compile SVG paths only when created/changed, not every frame. Define winding and hole behavior. For extrusion, lathe, sweep, loft, ribbon/tube or layered relief, validate the result and report unsupported cases instead of silently producing incorrect fills.
- Use bounded/adaptive curve flattening where supported.
- Check concavity, holes, self-intersections, degenerate segments and near-plane clipping.
- Give caps and side walls appropriate normals; split vertices at hard edges when needed.
- Avoid extreme tessellation without visible benefit.

## Coordinate-space contract
Keep five spaces distinct:
1. **Local:** object-relative coordinates.
2. **World:** shared scene coordinates after model transform.
3. **Camera:** camera-relative coordinates before perspective division.
4. **Raster:** integer pixels/samples in the working buffer.
5. **Display:** logical CSS pixels in preview, or selected output pixels in video export.

Normal flow: `local → world → camera → raster → display`. Vector output may project directly to display space, but conversion must remain explicit.
- Every projection constant declares its space and units in a nearby comment.
- Every focal-length, viewport/raster/display-size, center-offset or scale conversion has a one-line audit comment stating source space, destination space and units.
- Never reuse a raster-space focal length as display-space focal length without deriving/converting it.
- `devicePixelRatio` is explicit render scale; it must not silently change world scale or camera motion.
- The resolved interactive display buffer follows CSS content-box dimensions. Supersampled working raster is separate. Video export follows selected output dimensions.

## Rendering and performance
### Avoid repeated work
- Use indexed geometry. Transform each unique vertex once per relevant change, then reuse it across faces.
- Cache compiled SVG paths, triangulation, normals, bounds, material IDs, primitives and palettes where useful.
- Invalidate only dependent data: camera changes should not retriangulate SVG; material changes should not rebuild topology.
- Cull invisible objects/chunks before per-face work. Reuse base geometry for repeated objects.
- Clip triangles against the near plane before perspective division. Cull backfaces only for one-sided materials.
- Quantized shading/palettes can reduce repeated color work; measure before claiming a benefit.
- Batch compatible SVG faces and retain nodes. Avoid rebuilding the whole SVG subtree or setting `innerHTML` every frame. Batching reduces DOM overhead but does not make huge paths free.
- Reuse typed arrays/buffers; avoid per-face allocations.

### Measure the right costs
Measure separately:
1. **Geometry/face:** transforms, clipping, culling, shading, visibility ordering.
2. **SVG/output:** path construction, output bytes, DOM updates and path processing.
3. **Pixel/sample:** rasterization, buffer clearing, pixel effects and downsampling.

The performance gate is a **measured elapsed-time budget, never a screen-area cap**. Projected size may guide visual LOD only when detail cannot affect the image. Memory/buffer safety limits are separate. Preview may adapt to measured bottlenecks; final export must use explicit fixed quality settings.

### Raster antialiasing
For software-raster output, integer supersampling factor `S` means:
- working raster is exactly `S × displayWidth` by `S × displayHeight`;
- each exact `S × S` block box-averages to one output pixel;
- alpha is handled correctly to avoid halos;
- test `S = 1` and at least one `S > 1`.
Do not force supersampling on native SVG or intentionally pixelated ASCII. Cost grows approximately as `S²`; profile it.

### Display/export
- Interactive resolved buffer follows CSS content-box dimensions; resize updates it explicitly.
- Export dimensions come from the selected output profile, independent of CSS layout.
- Keep camera projection and raster/display conversion explicit.

## Minimal structure
Default:
```text
animation-name/
  index.html
```
Optional only when useful:
```text
animation-name/
  index.html
  components/
    component-name/
      component.js
      style.css
```
One folder per animation. A component folder is for substantial/reused components only. No default `engine/` directory split into many files. Director/World/Renderer are conceptual responsibilities and can live in one HTML file.

## Completion checklist
Apply the checks relevant to the scene:
- [ ] Runs from the expected local server; no missing files or console errors.
- [ ] First frame, key beats, transitions and final frame look correct.
- [ ] Same requested frame/time produces the same state.
- [ ] Camera frames the subject and avoids accidental clipping/jumps/premature reveals.
- [ ] Grounded objects use the exact rendered support surface; relevant footprint points checked.
- [ ] Connected parts remain attached; shared boundaries meet.
- [ ] No visible wrong depth order, flicker, accidental holes, clipping or buried surfaces.
- [ ] Labels/connectors/clipping/containment/read order remain correct during motion.
- [ ] Aspect ratio and display/export dimensions are correct.
- [ ] Projection constants and conversions declare space/units.
- [ ] Performance claims are measured and geometry/output/pixel costs distinguished.
- [ ] If raster supersampling is used, integer scale and exact box downsample verified.
- [ ] Audio timing aligns if audio is present.
- [ ] Report limitations and only tests actually performed.

## Efficient editing protocol
For an existing animation:
1. Find the exact function/constant controlling the request.
2. Change the smallest relevant region and preserve unrelated design.
3. Preview affected frames.
4. If the defect persists, trace its underlying cause (geometry, coordinates, visibility, timing); do not stack arbitrary offsets or decorative fixes.
5. Summarize root cause and validation performed.

For a new animation:
1. Make a compact shot plan.
2. Build a coherent first pass in one HTML file.
3. Preview early.
4. Correct silhouette, composition, camera, grounding and visibility.
5. Polish only after the core reads correctly.
