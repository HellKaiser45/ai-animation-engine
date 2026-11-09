# Exploded product view

**Compatibility: Native with custom scene code.** A stylized exploded view is achievable with projected polygons or the engine's verified raster/depth path. Photorealistic product rendering is not assumed.

## Essence

Reveal a product's construction and the relationships between its parts. Each separation should answer a question: what is inside, what connects, what moves, or why a feature matters.

## Use it for

Product explainers, fictional devices, mechanical systems, educational teardowns and component demonstrations.

## Visual language

- Give the product a clear silhouette and a material palette that distinguishes components.
- Use a small number of surfaces and controlled highlights; avoid pretending that flat shading is a physically accurate material simulation.
- Name the important components and use consistent colour coding.
- Keep the assembled product visible as a reference when parts separate.
- Use a clean studio-like background and avoid unnecessary reflections or particles.

## Camera grammar

Start with a readable three-quarter view. Orbit only enough to reveal the relevant structure. Use close-ups for a specific component and a measured pull-back for the final reassembly. Maintain the same camera reference so the audience understands where each component belongs.

## Motion grammar

Separate components along clear assembly axes. Use anticipation, a controlled ease-out and a stable stop. Keep parts aligned to their original parent transform and use shared anchors for connectors. Sequence the teardown, hold long enough to understand it, then reassemble or resolve the product.

## Sound direction

Use restrained material-specific clicks, slides, magnetic snaps and low mechanical impacts. A short voice line can explain the purpose of each newly revealed layer. Do not allow sound to imply a mechanism that the geometry does not show.

## VidAnim implementation

- Build the product from named component records containing local geometry, transform, material and anchor points.
- Compute exploded transforms from a shared assembly definition; derive assembled and exploded states from time, not from incremental position updates.
- Use SVG/Canvas for simple silhouettes and callouts; use a verified depth-buffer raster path for overlapping opaque components when painter order fails.
- Keep labels tied to component anchors and readable through camera motion.
- Use stylized geometry when it improves clarity. Add no downloaded CAD/model asset unless the user approves the dependency and licence.

## Avoid

Parts that drift away from the product, arbitrary separation directions, impossible connections, over-complicated internals, excessive specular tricks and misleading labels.

## Review checks

Check component count, assembly order, attachment points, depth ordering, label association, camera continuity and exact return to the assembled state.
