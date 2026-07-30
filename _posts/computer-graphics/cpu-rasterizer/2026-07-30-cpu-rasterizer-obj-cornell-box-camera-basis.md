---
title: "CPU rasterizer (8): OBJ Cornell Box and a camera-basis bug"
date: 2026-07-30
ai_assisted: true
tags:
  - graphics
  - rendering
  - rasterization
  - obj
  - linear-algebra
---

This is the **eighth** post in my **CPU rasterizer** series for [Amit Labs](https://github.com/amitprakash07/amit-labs). The [previous post]({% post_url 2026-06-03-cpu-rasterizer-mvp-matrix-layout-and-depth-buffer %}) proved that the model/view/projection path, depth buffer, color interpolation, and checkerboard texture sampling could work together on hand-authored local-space geometry.

The next step was to stop proving the pipeline with a custom quad and feed it real asset data. For this milestone I wired OBJ mesh loading through the graphics pipeline and rendered a Cornell Box model through the CPU rasterizer.

The final image is not a physically lit Cornell Box. There are no normals, shadows, light transport, or material shading yet. The goal here was narrower: load a real OBJ model, preserve material colors, transform the mesh through the existing camera path, and rasterize a recognizable scene.

![Cornell Box OBJ rendered through the CPU rasterizer with corrected camera basis]({{ '/assets/img/blog/2026/cpu-rasterizer/08-obj-cornell-box-camera-basis/01_cornell_box_obj_camera_basis_fixed.png' | relative_url }})

## What changed

The renderer now has an OBJ-based path for both a simple textured quad and the Cornell Box:

```text
OBJ file
    -> asset_loading::ObjMeshData
    -> graphics_asset::ObjAdapter
    -> graphics::MeshData
    -> RenderPrimitiveTriangle<LocalSpace>
    -> model/view/projection
    -> ClipSpace
    -> ScreenSpace
    -> CPU rasterizer
```

The important architectural change is the `graphics_asset` layer. OBJ data has source-format details: groups, material names, and original asset metadata. The rasterizer should not need to know those details. The adapter keeps both sides available:

- `ObjMeshData` remains the source/import representation.
- `MeshData` is the graphics representation consumed by the renderer.
- The adapter keeps a handle-based import record so debug tools can still inspect the original OBJ data when needed.

That gives me a clean split: the renderer sees vertices, indices, color, and UV data; debugging can still ask which OBJ group or material produced a triangle.

## Material colors from the OBJ/MTL pair

The Cornell Box asset uses an OBJ file plus a companion MTL file. The OBJ names groups and assigns materials:

```text
g leftWall
usemtl leftWall

g rightWall
usemtl rightWall
```

The MTL file defines those material colors:

```text
newmtl leftWall
Kd 0.63 0.065 0.05   # red

newmtl rightWall
Kd 0.14 0.45 0.091   # green
```

The loader resolves vertex color from explicit OBJ vertex colors when present, otherwise from the material diffuse color (`Kd`). That makes the Cornell wall colors a useful visual test. If the asset says `leftWall` is red and `rightWall` is green, the rendered image should show the same orientation.

## The first Cornell output was mirrored

The first useful Cornell render showed the room, but the red and green walls were on the wrong sides. That was a good bug because the image was not completely broken. The rasterizer was drawing geometry, depth was working well enough to show the room, and materials were clearly reaching the framebuffer.

The problem was more specific: the camera's horizontal basis was flipped.

The matrix convention in this renderer is:

```text
storage: column-major
vectors: row-vector pre-multiply
         p' = p * M
composition:
         clip = local * model * view * projection
translation:
         m_30, m_31, m_32
projection:
         D3D-style left-handed clip space
```

For the screen mapping, positive NDC X goes to the right side of the framebuffer:

```text
x_screen = (ndc_x + 1) * 0.5 * width
```

So the first column of the view matrix should represent camera-right. Instead, my look-at matrix had built and stored a camera-left vector as the view X basis:

```cpp
const maths::Vector3 forward = (target - eye).CreateNormalized();
const maths::Vector3 left    = up_vector.Cross(forward).CreateNormalized();
const maths::Vector3 up      = forward.Cross(left);
```

With row-vector multiplication, view-space X is the dot product against that first basis column. Putting `left` there meant camera-left became positive view X, and positive view X maps to screen-right. The scene was horizontally mirrored.

## Fixing the camera basis

The corrected look-at basis uses camera-right for view X:

```cpp
const maths::Vector3 forward = (target - eye).CreateNormalized();
const maths::Vector3 right   = forward.Cross(up_vector).CreateNormalized();
const maths::Vector3 up      = right.Cross(forward);
const maths::Vector3 translation =
    maths::Vector3{-right.Dot(eye), -up.Dot(eye), -forward.Dot(eye)};
```

Then the view matrix stores:

```text
column 0: right
column 1: up
column 2: forward
```

After that change, the Cornell output matched the material layout: red wall on the visual left, green wall on the visual right.

## Why the boxes are hard to see

The Cornell model includes the short box and tall box, but they are difficult to read in the current output. That is expected. The floor, ceiling, back wall, short box, and tall box all use the same off-white diffuse material:

```text
Kd 0.725 0.71 0.68
```

Without lighting, normals, shadows, edge outlines, or ambient occlusion, those surfaces blend together. This renderer is still writing flat material color. The image validates geometry ingestion and camera orientation, not lighting.

That is an important scope boundary. The next shading improvement could be simple normal-based brightness or group-debug colors, but that is not required to close this milestone.

## What this milestone proves

This step proves that the renderer can now handle more than hand-authored triangles:

- OBJ mesh loading works for a non-trivial scene.
- MTL diffuse colors are carried into the rasterized image.
- The graphics asset adapter keeps source asset data separate from render data.
- The local -> clip -> screen pipeline works on imported mesh geometry.
- The Cornell Box render exposed and helped fix a camera-basis convention bug.

The useful part of the bug was the diagnosis path. I did not start by changing the rasterizer. I compared the asset metadata against the visual result:

```text
OBJ/MTL says: leftWall = red, rightWall = green
image says:   left side = green, right side = red
conclusion:   material loading works, but camera X is flipped
```

That kind of debugging is exactly why asset metadata and render data need to remain connected without being mixed together in the rasterizer.

## What is still open

This is still not a complete production-style renderer:

- no real triangle clipping, only trivial reject
- no perspective-correct interpolation yet
- no lighting or normal-based shading
- no shadows
- no interactive camera controls
- Cornell boxes are not visually distinct without debug colors or lighting

Those are all valid future improvements, but they are not blockers for the larger roadmap. The next major milestone is still CPU volume rendering: ray generation, fixed-step ray marching, density evaluation, and alpha/transmittance accumulation.

## Takeaway

This milestone moved the project from "the graphics pipeline works on a custom demo" to "the graphics pipeline can ingest and render a real OBJ scene." It also clarified a deeper rule for this renderer: matrix storage, multiply order, projection convention, and camera basis direction all have to agree. If even one axis is named or placed incorrectly, the image can look plausible while still being wrong.

This post was written with AI assistance and reviewed/edited by me.
