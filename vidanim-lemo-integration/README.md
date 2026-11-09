# VidAnimEngine

VidAnimEngine is an AI-first, lightweight animation engine for bespoke procedural 2D, 2.5D and 3D-looking films made with vanilla HTML, CSS and JavaScript.

The default project remains a single self-contained HTML file. SVG, Canvas 2D, typed arrays and a small software rasterizer are available where appropriate. External libraries, downloaded assets and build systems are optional, not prerequisites.

## Create with an agent

1. Read `vidanimblueprint_v4.2.md` and the closest working example before choosing implementation details.
2. Pick a suitable visual direction from [`styles/README.md`](styles/README.md), or create a bespoke one when a listed style does not fit.
3. For a substantial film, write a treatment from [`templates/TREATMENT.md`](templates/TREATMENT.md) before authoring the scene.
4. Build the smallest complete animation, normally one HTML file.
5. Expose the optional agent-facing page contract described in [`docs/production-contract.md`](docs/production-contract.md) when the animation is intended for deterministic frame capture or video export.
6. Inspect representative frames and key motion before delivery. Report only checks that were actually run.

The authoring instructions live in [`agent-skills/vidanim-engine/SKILL.md`](agent-skills/vidanim-engine/SKILL.md).

## Starter style packs

The initial style packs are authoring recipes, not a new renderer or mandatory framework:

- Cinematic procedural journey
- Low-poly isometric world
- Editorial motion graphics
- Data storytelling
- Technical blueprint
- Exploded product view
- Layered paper cutout
- Pixel-art narrative

Each pack states what can be built with the current browser-native approach, where custom scene code is needed, and which effects should not be claimed unless implemented.

## Important compatibility rule

The blueprint describes design goals and possible techniques, not a guarantee that a general-purpose API already exists. Verify every API in a working example. In particular, the engine does not currently provide a universal style plug-in API, a built-in audio-mixing service, a generic 3D scene graph, or a complete headless video-export command. The style recipes and page contract document a path toward those capabilities without pretending they already exist.
