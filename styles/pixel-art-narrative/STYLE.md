# Pixel-art narrative

**Compatibility: Native.** Canvas 2D on an explicit low-resolution grid is enough; external pixel-art libraries are unnecessary.

## Essence

Tell a compact story through deliberate pixels, limited palettes and readable poses. Pixel art is a design system, not a blur filter applied to high-resolution graphics.

## Use it for

Retro game-inspired stories, playful explainers, game histories, simple character adventures and short sequences with clear state changes.

## Visual language

- Define the logical resolution before drawing and scale it by an integer factor where possible.
- Use a restricted palette with stable role assignments: background, outline, surface, highlight and accent.
- Design silhouettes and facial features for the chosen pixel scale; do not shrink detailed vector art and call it pixel art.
- Keep typography pixel-aligned and large enough to read.
- Use dithering only when it supports tone or material.

## Camera grammar

Use a stable side view, top-down view or deliberately staged screen composition. Camera movement should be stepped only if the style calls for it; if smooth camera travel is needed, keep the camera smooth while the world artwork remains on the pixel grid.

## Motion grammar

Use clear key poses, short cycles, readable anticipation and impact frames. Deliberate stepping can create charm, but key actions must remain understandable. UI-like transitions should have consistent timing and not overwhelm the story.

## Sound direction

A small original synthesized palette can support the retro feel: short bleeps, noise bursts and percussive notes. Avoid copying recognizable game melodies or sound effects.

## VidAnim implementation

- Draw at a fixed logical grid, then scale by an integer factor using a documented nearest-neighbour approach.
- Keep all geometry and text aligned to the pixel grid after projection.
- Use seeded random generation for stars, particles and procedural variation.
- Avoid fractional translations that create inconsistent pixel shimmer unless the effect is deliberate.
- Treat any ASCII/terminal variant as its own low-resolution renderer; it still needs visibility and depth logic.

## Avoid

Uncontrolled palette growth, fractional resampling blur, unreadable micro-text, random flicker, copied game sprites and smooth vector animation that fights the pixel grammar.

## Review checks

Check silhouette readability, palette consistency, grid alignment, animation stepping, camera movement, text contrast and integer scaling.
