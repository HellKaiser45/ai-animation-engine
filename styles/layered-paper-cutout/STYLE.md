# Layered paper cutout

**Compatibility: Native.** Layered SVG or Canvas 2D with deterministic parallax, masks and simple shadows is sufficient.

## Essence

A world made from distinct sheets of coloured paper. Depth comes from overlap, cut edges, soft cast shadows and restrained parallax rather than realistic 3D geometry. The transitions should feel like paper being folded, lifted, peeled or opened.

## Use it for

Storybooks, educational metaphors, gentle narratives, maps, nature sequences and playful brand films.

## Visual language

- Use a tactile but simple palette, with each layer having a clear silhouette.
- Build depth from a few separated planes. Use soft offset shadows to show which layer sits above another.
- Prefer slightly irregular silhouettes and restrained edge texture to excessive noise.
- Keep characters and props simple, expressive and easy to read.
- Make the paper medium visible in the transitions, not just in the background.

## Camera grammar

Use a mostly frontal camera with subtle push-ins, lateral drift or shallow parallax. A fold-out or pop-up reveal can change the composition, but maintain stable layer relationships. Do not simulate deep perspective if the style depends on shallow paper relief.

## Motion grammar

Use hinge rotations, sliding layers, unfolding panels, paper wipes, cutout reveals and small squash/settle actions. Keep a common parent transform for parts that move together. Use masks to ensure pieces emerge from the correct edge.

## Sound direction

Paper rustles, soft taps, folds and gentle impacts are appropriate. Use silence when a page opens or the story turns. Keep foley tied to visible paper movement.

## VidAnim implementation

- Represent each paper plane as a stable SVG group or Canvas layer.
- Use explicit depth/layer ordering and shared parent transforms.
- Simulate shallow 3D rotations only when the projection stays understandable; otherwise use 2D transforms and designed parallax.
- Use masks/clipping for fold and reveal transitions. Ensure no blank frame appears between layers.
- Avoid image textures unless user-supplied or appropriately licensed; simple procedural shading is enough.

## Avoid

Plastic-looking 3D shading, too many depth planes, harsh black shadows, arbitrary spinning pages, broken layer attachment and busy textures that weaken the cutout silhouette.

## Review checks

Check edge continuity, shadow direction, layer ordering, hinge attachment, masks, text contrast and how the folded/unfolded composition reads at every beat.
