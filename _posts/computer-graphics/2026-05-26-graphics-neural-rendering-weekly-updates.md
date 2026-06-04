---
title: "Graphics and neural rendering weekly updates"
date: 2026-05-26
ai_assisted: true
tags:
  - graphics
  - rendering
  - neural-rendering
  - weekly-update
---

This is a running weekly update log for my graphics and neural rendering work in
[Amit Labs](https://github.com/amitprakash07/amit-labs).

The milestone posts are still where I write the deeper technical explanations.
This page is different: it is an accountability log. Each week should capture what
actually happened, what shipped, what slipped, what I learned, and which artifact
or post came out of the work.

Newest updates go first. Starting in June 2026, I want to update this log weekly,
including weeks where progress is mostly planning, debugging, or cleanup.

---

## Week of May 26, 2026

### Focus

Recalibrate the roadmap and make the CPU rasterizer pipeline closer to a real
model/view/projection flow.

### Shipped

- Reworked the neural rendering roadmap into a stable milestone plan with rough
  timing instead of hard daily dates.
- Split the plan into a realistic baseline track and a mid-August
  portfolio-ready track.
- Continued the CPU rasterizer graphics-pipeline work: model transforms,
  perspective projection, clip-space rasterizer entry, and scaled-quad MVP
  experiments.
- Moved graphics-specific matrix construction out of the generic `Matrix4x4`
  class and into graphics-level transform/projection functions.

### Missed / Still Open

- CPU Cornell Box is still not reached.
- Depth buffer and full texture validation still need to be integrated into the
  graphics-pipeline path.
- Partial clipping is still missing; large scaled primitives can expose this
  limitation.

### Learned

- The roadmap should not change every time execution changes. The milestone plan
  should stay stable; weekly updates should capture execution reality.
- Projection is a view-to-clip transform, not a view-to-screen transform.
- Scaling the same local-space quad is a clean way to prove that MVP is doing
  real work.

### Links

- Milestone post: [CPU rasterizer (6): Graphics pipeline, clip space, and coordinate-safe primitives]({% post_url 2026-05-25-cpu-rasterizer-graphics-pipeline-and-coordinate-spaces %})

---

## Week of May 19, 2026

### Focus

Move from screen-space rasterization toward local-space assets and transform-aware
geometry.

### Shipped

- Added OBJ loading and graphics asset adapter work.
- Started moving mesh data into local space instead of treating all geometry as
  screen-space data.
- Refactored render primitives and vertex attributes toward explicit coordinate
  spaces.
- Added the bare-bones graphics pipeline path that transforms local-space
  geometry into clip-space primitives before rasterization.

### Missed / Still Open

- The MVP path was not yet fully real; identity transforms still dominated the
  first version.
- Cornell Box loading/rendering was still pending.
- Clipping was limited to trivial rejection.

### Learned

- OBJ data belongs in local/model space.
- Render-call stats and geometric space should be separate ideas.
- Strong coordinate-space types are worth the refactor because they prevent local,
  clip, NDC, and screen data from being mixed accidentally.

### Links

- Milestone post: [CPU rasterizer (6): Graphics pipeline, clip space, and coordinate-safe primitives]({% post_url 2026-05-25-cpu-rasterizer-graphics-pipeline-and-coordinate-spaces %})

---

## Week of Apr 21, 2026

### Focus

Strengthen the CPU rasterizer architecture after triangle fill started working.

### Shipped

- Added UV data through the rasterization path.
- Added checkerboard texture sampling in screen space.
- Added or refined depth-test related experiments.
- Split math folders and cleaned up render state boundaries.
- Added frame-level and draw-level render stats.
- Clarified asset-loading boundaries and renderer-owned graphics contracts.

### Missed / Still Open

- Texture sampling was still a screen-space proof of concept, not yet a full
  model-space textured pipeline.
- OBJ and Cornell Box rendering were not complete yet.
- Perspective-correct interpolation remained future work.

### Learned

- A small generated checkerboard is enough to prove the texture sampling path.
- Refactors that do not change the image can still be critical if they make the
  next rendering feature possible.
- Asset loading should stay neutral; graphics-owned mesh data should be adapted
  at the boundary.

### Links

- Milestone post: [CPU rasterizer (3): C++ type safety, semantic wrappers, and render-state boundaries]({% post_url 2026-04-23-cpu-rasterizer-cpp-type-safety-and-render-state %})
- Milestone post: [CPU rasterizer (4): Screen-space texture sampling with a checkerboard]({% post_url 2026-04-24-cpu-rasterizer-screen-space-texture-sampling %})
- Milestone post: [CPU rasterizer (5): Rendering architecture, asset boundaries, and frame stats]({% post_url 2026-04-25-cpu-rasterizer-rendering-architecture-refactor %})

---

## Week of Apr 14, 2026

### Focus

Make the rasterizer less like a one-off triangle demo and more like a small render
system.

### Shipped

- Added render stats and text overlay work.
- Started expanding toward depth buffer support.
- Split primitive concepts and render primitive responsibilities.
- Performed namespace/header/CMake cleanup that made the project easier to grow.

### Missed / Still Open

- The renderer was still largely screen-space based.
- Texture, OBJ loading, and camera transforms were still ahead.

### Learned

- Instrumentation matters early. Frame stats and overlays make later experiments
  easier to understand.
- Primitive topology and vertex-space data should not be treated as the same
  responsibility.

---

## Week of Apr 3, 2026

### Focus

Start the CPU rasterizer and establish the first visible rendering milestones.

### Shipped

- Added triangle edge functions.
- Added barycentric-coordinate based triangle filling.
- Started the first neural rendering roadmap.
- Published early CPU rasterizer posts.

### Missed / Still Open

- No depth buffer, texture path, real camera, or object loading yet.
- The initial roadmap was too optimistic for the amount of architecture work the
  rasterizer eventually needed.

### Learned

- A single filled triangle is a useful first artifact, but the architecture has to
  evolve quickly once depth, textures, and assets enter the project.
- The roadmap should reserve time for debugging and refactoring, not just visible
  rendering features.

### Links

- Milestone post: [CPU rasterizer (1): Triangle fill with barycentric coordinates and interpolated colors]({% post_url 2026-04-10-cpu-rasterizer-triangle-barycentric-colors %})
- Milestone post: [CPU rasterizer (2): Framebuffer text overlay (pixel stats and timings)]({% post_url 2026-04-11-cpu-rasterizer-framebuffer-text-overlay %})
