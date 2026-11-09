# VidAnimEngine style catalogue

These are **authoring recipes**, not drop-in renderer plugins. They adapt visual families found in code-driven motion design and short-film libraries to the current VidAnimEngine philosophy: vanilla HTML/CSS/JS by default, procedural geometry, SVG/Canvas, a single deterministic timeline and one coherent world/camera where spatial content is used.

## Compatibility labels

- **Native** — can be authored with existing browser primitives (HTML/CSS/JS, SVG, Canvas 2D) without a new runtime dependency.
- **Native with custom scene code** — achievable without external dependencies, but the film must implement its own projection, geometry, layering, visibility or raster techniques.
- **Optional / not core** — advanced fidelity may need a dependency or capability not guaranteed by the current engine. Do not enable it without user agreement and code verification.

The repository does not yet have a universal style loader, scene graph, audio mixer or headless video exporter. Read the relevant STYLE.md and implement the needed scene locally. The style pack must never override the user's subject or force a demo's story.

## Starter packs

| Style | Compatibility | Best for | Adapted visual family |
|---|---|---|---|
| [Cinematic procedural journey](cinematic-journey/STYLE.md) | Native with custom scene code | Landscapes, environments, continuous camera journeys | Procedural 3D / desktop flythrough |
| [Low-poly isometric world](low-poly-isometric/STYLE.md) | Native with custom scene code | Dioramas, miniature worlds, process journeys | Low-poly island / isometric infographic |
| [Editorial motion graphics](editorial-motion-graphics/STYLE.md) | Native | Titles, campaigns, abstract explainers | Swiss motion / constructivist montage |
| [Data storytelling](data-storytelling/STYLE.md) | Native | Statistics, timelines, comparisons, systems | Data storytelling / transit map |
| [Technical blueprint](technical-blueprint/STYLE.md) | Native | Engineering, architecture, mechanisms, processes | Blueprint / engineering briefing board |
| [Exploded product view](exploded-product/STYLE.md) | Native with custom scene code | Product features, component relationships, teardown explainers | Exploded teardown / product visualization |
| [Layered paper cutout](layered-paper-cutout/STYLE.md) | Native | Storybooks, educational metaphors, paper craft | Paper-cut lightbox / pop-up book / shadow theatre |
| [Pixel-art narrative](pixel-art-narrative/STYLE.md) | Native | Retro stories, game-like explainers, playful sequences | Pixel RPG / microgame / terminal graphics |

## Style selection for agents

Choose the style that best serves the subject and audience, not the one with the most impressive effects. A style can be combined with one secondary influence only when their rules are compatible. If the request is a factual explainer, clarity and data integrity outrank decoration. If the request is a cinematic journey, camera, scale and atmosphere should carry the story.

## Adding a style

Create a folder with a concise `STYLE.md`. Include: essence, when to use it, what not to do, palette/materials, composition, camera grammar, motion grammar, typography, transitions, sound suggestions, engine implementation notes, risks and review checks. Label compatibility honestly. Describe techniques, not fictional APIs. Never copy a demo film, its characters, logos, protected designs, distinctive shots or soundtrack.
