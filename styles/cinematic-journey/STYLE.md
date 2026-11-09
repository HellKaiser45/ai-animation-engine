# Cinematic procedural journey

**Compatibility: Native with custom scene code.** A proven direction in the Aethra example, not a universal 3D scene-graph API.

## Essence

A camera-led passage through a coherent procedural environment. The viewer should feel distance, scale and discovery. Terrain, architecture, weather, light and sound cues must describe one connected place rather than a collection of unrelated set pieces.

## Use it for

Fantasy landscapes, futuristic cities, natural environments, orbital or underwater journeys, product-scale reveals and location-based stories.

## Visual language

- Establish a small, controlled palette tied to time of day, geography or mood.
- Build major silhouettes first: ridge, skyline, bridge, tower, horizon or hero object.
- Use atmospheric perspective, carefully placed highlights and restrained fog to separate depth.
- Vary broad environment shots with close passes that reveal meaningful detail.
- Prefer coherent procedural materials over noisy decoration. Reserve the highest contrast for the subject and important landmarks.

## Camera grammar

Use an intentional route with a visible beginning, progression and destination. A continuous forward flight can work, but use lateral tracking, orbit, rise, pull-back or a held reveal when the story benefits. Control camera speed and look direction independently. Do not fly through geometry or reveal the main landmark before its beat.

## Motion grammar

Use slow environmental motion (clouds, water, foliage, particles) sparingly. Large objects should move only when the story calls for it. Animate a reveal with anticipation, a clear action and a settle. Use deterministic functions of time for all motion.

## Sound direction

Plan an ambience bed that changes with the location, low-frequency movement for scale, and a small number of material-specific effects for passes or reveals. Music should evolve with the route and leave breathing space. These are cues for an optional audio layer; this style pack does not supply an audio renderer.

## VidAnim implementation

- Inspect `examples/Aethra — a 60 second journey.html` and `examples/3d-world-tour.html` before reusing camera or raster code.
- Use the currently working software-raster / depth-buffer approach where a spatial scene needs robust opaque-surface ordering. Do not assume a reusable world API exists.
- Generate terrain and repeated props from seeded data. Reuse base geometry and cull invisible chunks where practical.
- Use the exact rendered surface or triangulation for grounded objects.
- For text, use world-anchored or camera-facing geometry with explicit depth and a legible safe area.
- Do not claim photorealistic PBR, physically based global illumination or volumetric lighting unless implemented and inspected.

## Avoid

A camera that moves constantly without revealing information; tiny subjects lost in wide frames; fog that hides the story; dense foliage that overwhelms landmarks; generic lens flares; random floating debris; and a soundtrack unrelated to the route.

## Review checks

Check the first reveal, closest camera pass, terrain contact, horizon crossings, final destination and any transparency. Compare `render(t)` at repeated timestamps if a deterministic page contract is exposed.
