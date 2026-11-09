# Technical blueprint

**Compatibility: Native.** SVG and Canvas 2D work well for diagrams, dimensions, leaders, callouts and layered technical drawings.

## Essence

A technical object or process is revealed as a readable engineered system. The style feels precise and measured; the animation should explain relationships rather than merely decorate a diagram.

## Use it for

Architecture, engineering, mechanisms, infrastructure, science explainers, product internals, system maps and instructional sequences.

## Visual language

- Use a limited drafting palette: dark or paper-like ground, one primary line colour, a secondary construction colour and a restrained highlight.
- Use consistent line weights for silhouettes, construction lines, dimensions and annotations.
- Make labels short and attach them to stable anchors.
- Use grid, measurement ticks and schematic symbols only when they help orientation or scale.
- Separate actual geometry from illustrative callouts so the viewer can distinguish physical structure from explanation.

## Camera and composition

Use orthographic or near-orthographic framing for diagrams and technical relationships. Use a controlled exploded view, section cut or orbit only when it reveals an actual relationship. Keep the subject large enough to inspect and reserve a predictable region for labels.

## Motion grammar

Reveal construction lines, trace connections, separate components along meaningful axes, rotate a component to show a hidden face, then reassemble or resolve the system. Keep connectors attached to their true anchors throughout the motion. Sequence one explanation at a time.

## Sound direction

Quiet mechanical clicks, ticks and subtle low-frequency cues can reinforce assembly. Avoid over-sounding every line draw. Voice, if present, should name the part currently emphasized.

## VidAnim implementation

- Use SVG paths and text for crisp technical lines; use Canvas for dense procedural grids or particles.
- Define a coordinate system, units and reference anchors before drawing.
- Derive leader endpoints and dimensions from the geometry they describe; do not duplicate unrelated coordinates.
- Build exploded offsets from component-local transforms so parts can return to their assembled positions exactly.
- Keep text in the same coherent world/camera model or explicitly define it as a camera-facing world object.
- Do not imply engineering accuracy when shapes are conceptual or dimensions are illustrative; label them accordingly.

## Avoid

Pseudo-precision, unreadable micro-labels, connectors that drift, arbitrary exploded offsets, inconsistent units, overlapping callouts and decorative technical noise.

## Review checks

Check dimensions, units, anchors, connector endpoints, component order, reassembly, text size and whether the animation explains the mechanism correctly.
