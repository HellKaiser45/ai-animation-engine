# Apply this integration package

This package is a prepared change set for `HellKaiser45/ai-animation-engine`. It has **not** been pushed to GitHub because the connected GitHub integration rejected branch creation with HTTP 403 (`Resource not accessible by integration`).

## Apply locally

1. Download and extract this package into the root of a fresh clone of `ai-animation-engine`.
2. Review the overlay. It adds the new README, docs, treatment template and style folders, and replaces `agent-skills/vidanim-engine/SKILL.md`.
3. From the repository root, apply the focused Aethra update:

   ~~~sh
   git apply patches/aethra-render-contract.patch
   ~~~

4. Review and commit:

   ~~~sh
   git diff --check
   git status --short
   git diff -- agent-skills/vidanim-engine/SKILL.md examples/
   ~~~

5. Open the Aethra example in a browser and verify `window.DUR`, `window.FPS`, `window.READY`, `window.render`, `window.EV` and `window.TEXTS` in the console. Call `window.render(44)` and `window.render(55)` to inspect the reveal and end title.

The patch deliberately does not change the renderer, add dependencies, or claim to provide an audio mixer or video-export CLI. The contract is an adapter convention; a separate export/capture implementation remains future work.
