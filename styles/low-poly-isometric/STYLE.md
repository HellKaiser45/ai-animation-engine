# Low-poly isometric world

**Compatibility: Native with custom scene code.** Implementable with projected polygons, Canvas or SVG; no Three.js dependency is required.

## Essence

A miniature world made from a clear set of geometric forms. The camera is usually orthographic or isometric in appearance, giving the world a designed, tactile quality. Growth, assembly or travel through the model can become the story.

## Use it for

Small towns, islands, process explainers, ecosystems, factory systems, product ecosystems and playful educational films.

## Visual language

- Choose one ground family, one structural neutral and a short accent palette.
- Use flat-shaded facets, clean silhouettes and repeated geometric primitives.
- Convey material through shape, value and facet colour, not photorealistic textures.
- Leave negative space around important groups. Keep the scene readable at the final output size.
- Avoid black outlines unless the chosen sub-style explicitly needs them.

## Camera grammar

Use a fixed isometric view for a calm overview, a controlled zoom for scale, a turntable-like orbit for inspection, or a lateral truck to reveal a sequence. If a full camera system is not implemented, use one explicit world-to-screen projection function and keep every element in the same coordinate convention. Do not mix screen and world coordinates casually.

## Motion grammar

Objects may rise, drop, stack, pop or assemble. Use short ease-out motion with restrained overshoot and a clear settle. Schedule repeated arrivals on a beat grid when music is present. Avoid making every object bounce at once; hierarchy matters.

## Sound direction

Pitched percussion and small material-specific impacts work well: wood, ceramic, stone, glass and water should not all sound alike. A repeating motif can evolve as the world grows. Keep the score optional and event-aligned.

## VidAnim implementation

- Build polygons from a deterministic axial or grid layout; compute their projected positions through one shared transform.
- Canvas 2D or SVG is appropriate for low/medium complexity. For dense overlapping 3D-looking geometry, inspect the software raster/depth-buffer examples and use them only when needed.
- Use a single source of truth for tile height, support and attachment. Do not place a tree using a different terrain approximation from the rendered terrain.
- Use explicit draw order for 2D layers; do not use CSS `z-index` to fake 3D face visibility.
- If an orthographic camera is not implemented in the chosen example, author the required projection locally rather than assuming a built-in camera class.

## Avoid

Uncontrolled random tile colours; a spreadsheet-like uniform layout; excessive tiny details; perspective changes that break the miniature model feeling; intersecting faces with unresolved ordering; and effects that obscure the silhouettes.

## Review checks

Inspect tile seams, shared borders, height transitions, grounded props, roof/wall overlap, title readability and the timing of object arrivals.
