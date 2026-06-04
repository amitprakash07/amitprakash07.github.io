---
title: "CPU rasterizer (7): MVP transforms, matrix layout, and depth-buffer proof"
date: 2026-06-03
ai_assisted: true
tags:
  - graphics
  - rendering
  - rasterization
  - linear-algebra
---

This is the **seventh** post in my **CPU rasterizer** series for [Amit Labs](https://github.com/amitprakash07/amit-labs). The [previous post]({% post_url 2026-05-25-cpu-rasterizer-graphics-pipeline-and-coordinate-spaces %}) wired up the local → clip → screen pipeline, but the MVP chain was still mostly identity. The geometry path was real; the transform math was not.

This step closes that gap. The renderer now runs a real model matrix, a look-at view matrix, and a left-handed perspective projection. It also surfaced one of the most useful debugging lessons so far: when rasterized pixels drop to zero, the bug is not always in the rasterizer. Sometimes the triangle never makes it through clip space because the matrix layout does not match the multiply convention.

## What changed since the last post

The graphics-pipeline demo now does this end to end:

```text
local-space quad (two triangles)
    -> model (scale, then translate)
    -> view (look-at camera)
    -> projection (perspective LH)
    -> clip-space primitive
    -> trivial clip reject
    -> perspective divide
    -> screen-space rasterizer
    -> depth test
    -> color or checkerboard fragment shader
```

Concrete code changes:

- `MakeModelMatrix()` builds `scale * translation` for row-vector multiply
- `MakeLookAtMatrix()` builds a real camera view matrix
- `MakePerspectiveLH()` builds a D3D-style left-handed projection matrix
- `MakeTransformMatrices()` composes `model * view * projection`
- coordinate-space and primitive `ToString()` helpers for debug output

The sample scene draws two local-space triangles from the textured quad:

- triangle 1: flat interpolated color, drawn first
- triangle 2: checkerboard texture, drawn second with a small translation offset

## The matrix convention matters

`Matrix4x4` in Amit Labs uses a specific contract:

```text
storage: column-major
vectors: row-vector pre-multiply
         v' = v * M
composition:
         clip = local * model * view * projection
         MVP  = model * view * projection
translation lives in m_30, m_31, m_32
```

That last line is easy to get wrong. In this layout, translation is **not** in column 3 (`m_03`, `m_13`, `m_23`). It lives in the bottom row.

The row-vector multiply makes the order read naturally left to right:

```cpp
// Matrix4x4 uses row-vector pre-multiply: clip = local * model * view * projection.
result.model_view_projection_matrix = result.model_matrix * result.view_matrix * result.projection_matrix;
```

If view or projection put translation or `w` in the wrong slots, clip space looked plausible in isolation but failed the clip test in practice.

## Real model transform: scale first, then translate

The model matrix is built as:

```cpp
const maths::Matrix4x4 scale_matrix       = MakeScaleMatrix(transform.scale);
const maths::Matrix4x4 translation_matrix = maths::Matrix4x4{transform.translation};

// Matrix4x4 uses row-vector pre-multiply: local * scale * translation.
return scale_matrix * translation_matrix;
```

That means:

```text
local point
    -> scaled in object space
    -> translated in world space
```

For a corner at `(-0.5, -0.5, 0)` with scale `(5, 5, 1)` and translation `(1, 0, -0.5)`:

```text
after scale : (-2.5, -2.5, 0)
after translate : (-1.5, -2.5, -0.5)
```

Important detail: **translation is not multiplied by scale**. A translation of `(1, 0, -0.5)` moves the already-scaled quad by one world unit in X and half a unit closer to the camera in Z. It does not become `(5, 0, -2.5)`.

Rotation is still not wired into `MakeModelMatrix()` yet. For this milestone, scale and translation were enough.

## View and projection

The demo camera is simple:

```cpp
const graphics::CameraView camera_view{
    .position = maths::Vector3{0.0f, 0.0f, -10.0f},
    .look_at  = maths::Vector3{0.0f, 0.0f, 0.0f},
    .up       = maths::Vector3{0.0f, 1.0f, 0.0f}};
```

The view matrix uses `MakeLookAtMatrix()`, with translation stored in the bottom row:

```cpp
// Row-vector layout: translation lives in m_30, m_31, m_32 (bottom row).
return maths::Matrix4x4{left.x(), left.y(), left.z(), translation.x(),
                        up.x(), up.y(), up.z(), translation.y(),
                        forward.x(), forward.y(), forward.z(), translation.z(),
                        0.0f, 0.0f, 0.0f, 1.0f};
```

Perspective projection follows the same row-vector LH contract:

```cpp
// D3D LH row-vector layout: w_clip = z_view, z_clip = z_view * A + B.
return maths::Matrix4x4{x_scale, 0.0f, 0.0f, 0.0f,
                        0.0f, y_scale, 0.0f, 0.0f,
                        0.0f, 0.0f, z_distance_scale, z_offset,
                        0.0f, 0.0f, 1.0f, 0.0f};
```

That gives the expected clip-space shape:

```text
clip.x = x_view * x_scale
clip.y = y_view * y_scale
clip.z = z_view * A + B
clip.w = z_view
```

## The zero-pixel bug

After wiring up the pipeline shape, the first real MVP run reported:

```text
Rasterized Pixels: 0
```

That was misleading at first. The rasterizer, edge functions, and depth buffer were still fine. The triangles were failing before pixel traversal because clip-space `w` was wrong.

Broken case for a visible corner:

```text
clip ≈ (-0.649, -0.866, -0.100, 0)
```

With `w = 0`, perspective divide and clip tests break down. Nothing useful reaches the screen-space rasterizer.

After fixing view translation placement and projection layout, the same vertex became:

```text
clip ≈ (-0.649, -0.866, 9.910, 10.0)
```

That is inside the canonical D3D clip volume, and pixels started appearing again.

This was a good reminder: debug the transform chain from local space outward before assuming the rasterizer regressed.

## Debug output across coordinate spaces

To make that debugging easier, I added a small `ToString()` hierarchy in `render_primitives.h`:

- coordinate-space labels: `LocalSpace`, `WorldSpace`, `ClipSpace`, `NDCSpace`, `ScreenSpace`
- per-vertex debug strings for position, color, and UV
- primitive debug strings for points, lines, and triangles

That made it much easier to inspect one vertex at a time:

```text
local position
    -> clip position
    -> NDC after divide
    -> screen position
```

It also pairs well with `TransformMatrices::ToString()`, which prints model, view, projection, and MVP together.

## Proving the depth buffer with overlapping geometry

Once MVP was working, the next goal was to prove depth testing in the graphics-pipeline path, not just in the older screen-space demo.

The setup:

```cpp
// Triangle 1 — flat color, farther, drawn first
ModelTransform{.translation = {0.0f, 0.0f, 0.0f}, .scale = {5.0f, 5.0f, 1.0f}}

// Triangle 2 — checkerboard, closer, drawn second
ModelTransform{.translation = {1.0f, 0.0f, -0.5f}, .scale = {5.0f, 5.0f, 1.0f}}
```

Both triangles use the same scale. Triangle 2 is shifted slightly in X and moved a little closer in Z. The draw order is intentional:

```text
1. draw farther triangle first
2. draw closer triangle second
3. depth test keeps the closer fragment
```

The depth buffer stores NDC `z` after perspective divide. `TryUpdateDepth()` accepts a fragment when:

```cpp
candidate_depth < stored_depth
```

So in overlap regions, the checkerboard triangle should win.

## What looked like a scale bug was mostly perspective

While tuning translations, one experiment felt wrong immediately: adding translation seemed to make the triangle "scale up," especially when Z changed.

That was confusing until I separated three different effects:

### 1. Translation is not being scaled by the model matrix

With `scale * translation`, world size stays the same. Only position changes.

### 2. Z translation changes apparent size in perspective

Because `w_clip = z_view`, moving a quad closer makes it occupy more NDC space:

```text
ndc_x ≈ x_world * x_scale / z_view
```

If `z_view` goes from `10` to `9.5`, the same world-sized triangle renders roughly 5% larger on screen. That is expected perspective behavior, not broken scale math.

### 3. Large translations can push vertices outside the frustum

The current renderer only does **trivial reject**, not full triangle clipping. If a translated/scaled triangle crosses a clip plane, out-of-frustum vertices still get projected and the triangle can look stretched or oversized.

That is why small Z offsets work better for depth experiments than aggressive ones like `(2, -1, -5)`.

For a clean depth test:

```text
keep XY mostly the same
use a small negative Z on the closer triangle
draw the same screen footprint twice
```

Example:

```text
triangle 1: (0, 0, 0)
triangle 2: (0, 0, -0.5)
```

The final demo uses a slightly larger offset in X plus a small Z shift so the overlap is visible without making the closer triangle dominate the whole frame.

## Final render

This is the current `RasterizeGraphicsPipeline()` output:

![Overlapping local-space triangles through the MVP pipeline: interpolated color triangle with a checkerboard triangle in front where they overlap]({{ '/assets/img/blog/2026/cpu-rasterizer/07-mvp-matrix-layout-and-depth-buffer/01_overlapping_quad_graphics_pipeline.png' | relative_url }})

What this image shows:

- the local-space quad is going through real model, view, and projection transforms
- one triangle uses interpolated vertex color
- the other uses UV-driven checkerboard sampling
- the overlap region shows the closer checkerboard triangle winning the depth test
- frame stats report two draw calls and a non-zero rasterized pixel count

That makes this the first graphics-pipeline screenshot that validates both **transform correctness** and **depth behavior** together.

## What works now

The graphics-pipeline path can now:

- scale and translate local-space geometry through a real model matrix
- look at geometry from a camera positioned off the origin
- project geometry with a perspective frustum
- reject fully outside clip-space triangles
- perspective-divide into NDC
- map NDC into screen space
- interpolate color and UV
- sample a checkerboard texture in the fragment shader
- resolve overlap with the depth buffer

That is enough to call Phase 1 MVP/depth/texture work largely functional for this scope.

## What comes next

The next rasterizer milestones I care about are:

- load the textured quad through the OBJ adapter instead of hand-authoring two triangles
- add rotation to `MakeModelMatrix()`
- add golden-value tests for view, projection, and MVP matrices
- implement real clip-space edge clipping instead of only trivial reject
- attempt a Cornell-box-like scene on the CPU path

After that, the roadmap moves toward CPU volume rendering, but this post is the point where the rasterizer stops being "pipeline-shaped" and starts being "camera-shaped."

## Related

- [CPU rasterizer (1): Triangle fill with barycentric coordinates and interpolated colors]({% post_url 2026-04-10-cpu-rasterizer-triangle-barycentric-colors %})
- [CPU rasterizer (2): Framebuffer text overlay (pixel stats and timings)]({% post_url 2026-04-11-cpu-rasterizer-framebuffer-text-overlay %})
- [CPU rasterizer (3): C++ type safety, semantic wrappers, and render-state boundaries]({% post_url 2026-04-23-cpu-rasterizer-cpp-type-safety-and-render-state %})
- [CPU rasterizer (4): Screen-space texture sampling with a checkerboard]({% post_url 2026-04-24-cpu-rasterizer-screen-space-texture-sampling %})
- [CPU rasterizer (5): Rendering architecture, asset boundaries, and frame stats]({% post_url 2026-04-25-cpu-rasterizer-rendering-architecture-refactor %})
- [CPU rasterizer (6): Graphics pipeline, clip space, and coordinate-safe primitives]({% post_url 2026-05-25-cpu-rasterizer-graphics-pipeline-and-coordinate-spaces %})
