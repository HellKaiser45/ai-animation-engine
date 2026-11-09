# Data storytelling

**Compatibility: Native.** SVG and Canvas 2D support charts, labels, timelines and animated diagrams without a charting dependency.

## Essence

Turn data into a visual argument. The animation should make one meaningful pattern, contrast or change easy to understand, then support it with enough context to be trusted.

## Use it for

Statistics, history, science, market or product metrics, timelines, network flows, comparisons and process explainers.

## Visual language

- Start with the viewer's question, not the chart type.
- Choose an honest scale and a visual encoding that matches the data: position for comparison, length for magnitude, area only when necessary, colour for category or emphasis.
- Label units, time ranges, sources and important caveats in readable language.
- Use restrained colour and consistent category assignments across scenes.
- Give the key figure or pattern a clear visual hierarchy without distorting it.

## Camera and composition

Use a stable, mostly frontal composition for charts and a controlled pan or zoom for long timelines, maps or networks. Camera motion must not distort axes, labels or perceived comparisons. Treat a chart transition as a change in the argument: overview → evidence → comparison → takeaway.

## Motion grammar

Animate bars from a common baseline, lines along their measured path, points into their actual positions and networks through meaningful connections. When values change, interpolate consistently and retain a visible reference scale. Avoid racing data labels or moving axes independently from their plotted values.

## Sound direction

A restrained pulse can mark chapter changes or milestones. Use silence or reduced music during dense data moments. Sound is optional and must never imply a value or event that the data does not support.

## VidAnim implementation

- Keep the source data in one structured object and derive geometry, labels and event times from it.
- Use SVG for axes, labels, paths and small chart sets; use Canvas for dense marks.
- Preserve exact values and units in code. Do not hand-position marks independently from their data mapping.
- Animate transitions from a shared time function; keep baselines and scales explicit.
- If the data is supplied by the user, do not invent missing values. Show gaps or label estimates clearly.

## Avoid

Truncated axes without an explicit reason; 3D perspective for quantitative comparisons; unexplained dual axes; overloaded legends; decorative motion that changes the perceived magnitude; and unsupported causal claims.

## Review checks

Verify every plotted value against the source data, units, scale, axis direction, label placement, source attribution, transitions and the final takeaway.
