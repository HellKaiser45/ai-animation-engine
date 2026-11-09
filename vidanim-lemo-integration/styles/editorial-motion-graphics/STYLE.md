# Editorial motion graphics

**Compatibility: Native.** SVG, Canvas 2D, HTML and CSS are sufficient.

## Essence

A precise graphic composition in which typography, geometry, timing and negative space carry the message. Every movement should clarify hierarchy, comparison or emphasis. It is not a collection of decorative transitions.

## Use it for

Brand explainers, short announcements, title sequences, campaigns, abstract explainers, product launches and concise educational videos.

## Visual language

- Use a small palette with a strong contrast rule and one accent colour.
- Establish a typographic hierarchy: headline, supporting information, labels and fine detail.
- Align elements to a consistent grid. Use whitespace as an active compositional element.
- Prefer a few large graphic gestures over many small animated objects.
- Keep text editable and use system or locally supplied fonts unless the user approves font downloads.

## Camera and composition

A fixed or gently moving graphic camera is usually enough. Simulate depth with scale, overlap and layered planes only when it helps. Maintain a stable reading direction. Transitions should preserve spatial or semantic continuity between scenes.

## Motion grammar

Use deliberate reveals, wipes, line growth, scale changes, tracking, masks and shape transformations. Stagger related elements with consistent timing. Use overshoot sparingly; avoid easing every element identically. Text must finish animating early enough to be read.

## Sound direction

Use clean percussive accents, subtle whooshes and a score with a clear rhythmic grid. Important changes can land on beats, but not every movement needs a sound. Dialogue and information take priority over music.

## VidAnim implementation

- Prefer SVG for crisp text, paths and sparse vector compositions; use Canvas for dense particle or shape fields.
- Keep a persistent SVG DOM and update attributes rather than rebuilding the complete subtree each frame.
- Animate all elements from a shared deterministic timeline.
- Declare safe margins and test the real target aspect ratio, including vertical formats.
- Use actual text metrics where available; do not estimate title width by character count alone when clipping matters.

## Avoid

Tiny type, long copy, arbitrary rotation, excessive blur, decorative grids that compete with the message, inconsistent spacing and transitions that make the reader lose their place.

## Review checks

Read every frame at final size. Check title timing, line wraps, alignment, contrast, safe margins, reading order and continuity across transitions.
