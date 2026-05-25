---
title: "CPU rasterizer (6): Graphics pipeline, clip space, and coordinate-safe primitives"
date: 2026-05-25
ai_assisted: true
tags:
  - graphics
  - rendering
  - rasterization
  - architecture
---

This is the **sixth** post in my **CPU rasterizer** series for [Amit Labs](https://github.com/amitprakash07/amit-labs). The previous post focused on rendering architecture: separating asset loading from graphics contracts, introducing render-frame state, and making frame/draw stats more explicit.

This step moves the project closer to a real graphics pipeline.

The renderer already knew how to rasterize triangles in screen space. That was enough for early experiments because I could hand-author pixel coordinates and ask the rasterizer to fill triangles. But that is not how a normal graphics pipeline is structured. Real geometry starts in local/model space, goes through transform stages, becomes clip-space data, and only later becomes screen-space pixels.

So the goal for this step was not to add a big visual feature. The goal was to make the code express the pipeline:

```text
local-space vertices
    -> MVP transform
    -> clip-space primitive
    -> clip-volume reject
    -> perspective divide
    -> screen-space primitive
    -> existing rasterizer
```

There is still more work to do. The model matrix and projection matrix are not fully implemented yet, so the current demo mostly behaves like identity transform into clip/NDC coordinates. But the shape of the pipeline is now in the code.

## From screen-space demo to graphics pipeline

Before this change, the sample program built triangles directly in screen space:

```cpp
RenderPrimitiveTriangle<ScreenSpace> triangle{
    make_vertex(Point3D{500u, 100u, 10u}, kRgb8ColorRed),
    make_vertex(Point3D{300u, 500u, 10u}, kRgb8ColorGreen),
    make_vertex(Point3D{700u, 500u, 10u}, kRgb8ColorBlue)};
```

That is useful for proving the rasterizer, but it skips the interesting part of a graphics pipeline. There is no model transform, no camera, no projection, and no clip space.

I split the sample into two paths:

```text
RasterizeInScreenSpace
    old direct-to-screen demo

RasterizeGraphicsPipeline
    local-space geometry -> clip-space rasterizer -> screen-space rasterizer
```

The new graphics-pipeline path starts with a simple quad made from two local-space triangles:

```cpp
RenderPrimitiveTriangle<LocalSpace> local_space_triangle_1{...};
RenderPrimitiveTriangle<LocalSpace> local_space_triangle_2{...};
```

The vertices match the small textured quad used by the OBJ loader:

```text
(-0.5, -0.5, 0)
( 0.5, -0.5, 0)
( 0.5,  0.5, 0)
(-0.5,  0.5, 0)
```

That means the hand-authored path and the asset-loading path are moving toward the same representation.

## The current pipeline shape

The graphics-pipeline function now transforms each local-space vertex into clip space:

```cpp
Vector4 clip_position = model_view_projection_matrix.Mul(
    Vector4{local_position.x, local_position.y, local_position.z, 1.0f});
```

Then it builds a clip-space vertex:

```cpp
VertexAttributes<ClipSpace>{
    .position = ClipSpace::Position{clip_position},
    .color    = local_vertex.color,
    .uv       = local_vertex.uv};
```

The important part is that color and UV are carried along while the position changes coordinate spaces. That mirrors the idea of a vertex shader: position is transformed into clip space, while other attributes become varyings that will eventually be interpolated.

Right now the MVP matrix is effectively identity:

```cpp
model      = identity
view       = identity
projection = identity
```

In code terms, `MakeModelMatrix` is still returning identity, the demo uses an identity view matrix, and perspective projection is not active yet. So the local quad coordinates are already in a clip/NDC-friendly range. A vertex at `(-0.5, -0.5, 0)` stays near `(-0.5, -0.5, 0, 1)`, then the viewport transform maps it into the framebuffer.

That is still useful. It proves the shape:

```text
LocalSpace triangle
    -> ClipSpace triangle
    -> Rasterizer::Rasterize(ClipSpace)
    -> ScreenSpace triangle
    -> Rasterizer::Rasterize(ScreenSpace)
```

That makes the current output a pipeline-shape validation, not a full transform validation. The next step is to replace those identity matrices with a real model transform, camera view, and perspective projection. When those matrices become real, the same path should keep working.

## What the clip-space rasterizer does

The screen-space triangle rasterizer is still the core rasterizer. It owns:

- bounding-box construction
- edge functions
- barycentric coordinates
- depth test
- color/UV interpolation
- fragment shader callback

The clip-space overload is a preparation stage. It does not walk pixels. It:

1. Checks whether the primitive is fully outside the clip volume.
2. Converts clip space to NDC using perspective division.
3. Converts NDC to screen space using the viewport.
4. Calls the existing screen-space rasterizer.

That keeps the real pixel work in one place.

## Understanding the clip test

After MVP transform, each vertex has a homogeneous clip-space position:

```text
(x, y, z, w)
```

For a D3D-style clip volume, the visible region is:

```text
-w <= x <= w
-w <= y <= w
 0 <= z <= w
```

Those inequalities define the six sides of the canonical view volume:

```text
left   : x >= -w
right  : x <=  w
bottom : y >= -w
top    : y <=  w
near   : z >=  0
far    : z <=  w
```

The code rewrites those into signed plane-side values:

```cpp
return {
    x + w,  // left
    w - x,  // right
    y + w,  // bottom
    w - y,  // top
    z,      // near
    w - z   // far
};
```

If a value is positive or zero, the vertex is on the visible side of that plane. If it is negative, the vertex is outside that plane.

For a triangle, the reject rule is:

```text
if all three vertices are outside the same plane,
the whole triangle is outside the clip volume.
```

For example:

```text
left(A) < 0
left(B) < 0
left(C) < 0
```

means the whole triangle is left of the view volume. It cannot cover the screen, so the rasterizer can reject it before perspective divide.

But this is not enough for full clipping:

```text
A is outside left
B is inside
C is inside
```

That triangle crosses the left plane. A full GPU-style clipper would cut the triangle, create new vertices where edges intersect the plane, and possibly generate more triangles. This renderer does not do that yet.

So this implementation is intentionally a **trivial reject**:

```text
fully outside -> discard
partially outside -> let it continue
fully inside -> let it continue
```

That is not complete clipping, but it is the right first step. It catches obviously invisible primitives without turning this milestone into a full homogeneous polygon clipper.

## Perspective divide and viewport transform

If a clip-space primitive survives the trivial reject, each vertex goes through perspective divide:

```text
ndc.x = clip.x / clip.w
ndc.y = clip.y / clip.w
ndc.z = clip.z / clip.w
```

That produces normalized device coordinates. Then the viewport maps NDC to pixel coordinates:

```text
screen.x = viewport.x + (ndc.x + 1) * 0.5 * viewport.width
screen.y = viewport.y + (1 - ndc.y) * 0.5 * viewport.height
screen.z = ndc.z
```

The `z` value is preserved for depth testing. Once the primitive is in screen space, the old rasterizer can take over:

```text
screen-space bbox
    -> edge tests
    -> barycentric interpolation
    -> depth test
    -> fragment shader
```

This also answers an important design question: the bounding box belongs after viewport transform. A bounding box used by the rasterizer answers "which pixels might this triangle touch?" That question only makes sense in screen space.

## Why full clipping is deferred

Full clipping is not just an `if` statement. A triangle crossing a clip plane can produce new geometry.

One input triangle can become:

```text
0 triangles  -> fully outside
1 triangle   -> fully inside or clipped down to a smaller triangle
2 triangles  -> clipped into a quad, then triangulated
```

Generated vertices also need interpolated attributes:

- color
- UV
- depth-related values
- later normals, tangents, or world positions

That is valuable renderer work, but it is a separate milestone. For now, I want the local-to-clip-to-screen path to be understandable before adding full polygon clipping.

## Refactoring around coordinate spaces

The most interesting refactor in this step was not the matrix multiplication. It was the type model around coordinate spaces.

Originally, coordinate spaces were aliases:

```cpp
using LocalSpace  = geometry::Point3D;
using ScreenSpace = geometry::Point3D;
using ClipSpace   = maths::Vector4;
```

That looked convenient, but it was unsafe. `LocalSpace` and `ScreenSpace` were both just `Point3D`, so the compiler could not tell them apart.

This is exactly the kind of mistake a graphics pipeline should prevent:

```text
local-space position accidentally passed to screen-space rasterizer
```

So the coordinate spaces became semantic tags with nested position types:

```cpp
struct ScreenSpace
{
    struct Position : geometry::Point3D
    {
        using geometry::Point3D::Point3D;

        Position() = default;

        explicit Position(const geometry::Point3D& point)
            : geometry::Point3D(point)
        {
        }
    };
};
```

Then `VertexAttributes` uses the space's associated position type:

```cpp
template <PrimitiveAttributeCoordinateSpaceConcept AttributeSpaceType>
struct VertexAttributes
{
    AttributeSpaceType::Position position;
    Rgb8                         color;
    UVCoordinate                 uv;
};
```

Now:

```cpp
VertexAttributes<LocalSpace>
VertexAttributes<ScreenSpace>
```

are different types, and their positions are also different types:

```text
LocalSpace::Position
ScreenSpace::Position
```

Both may inherit from `Point3D`, so existing code can still read:

```cpp
position.x
position.y
position.z
```

But the semantic type is preserved. A screen-space position is not silently the same thing as a local-space position.

The `explicit` constructor is important here. It means converting from a raw `Point3D` into a space-specific position must be intentional:

```cpp
ScreenSpace::Position{point}
LocalSpace::Position{point}
```

That makes coordinate-space transitions visible in code.

## Primitive shape versus coordinate space

This refactor also clarified another distinction:

```text
RenderPrimitive<kTriangle>
    topology/stat identity

RenderPrimitiveTriangle<LocalSpace>
RenderPrimitiveTriangle<ClipSpace>
RenderPrimitiveTriangle<ScreenSpace>
    actual geometry in a coordinate space
```

Stats such as vertex count and primitive count do not depend on whether a triangle is in local space or screen space. But rasterization absolutely depends on space. A triangle in clip space is not ready for edge-function rasterization; a triangle in screen space is.

So the renderer now has a cleaner split:

```text
space-aware geometry flows through the pipeline
space-agnostic primitive identity supports stats
```

That is a small design decision, but it matters because meshes are coming next.

## What works now

The current path can:

- define a quad in local space
- transform it through the MVP hook into clip space
- trivially reject fully outside clip-space primitives
- perspective divide into NDC
- viewport transform into screen space
- reuse the existing screen-space rasterizer
- preserve color and UV through the path
- write a separate `output_graphics_pipeline.ppm`

![CPU rasterizer graphics pipeline output with checkerboard texture and interpolated color triangle]({{ '/assets/img/blog/2026/cpu-rasterizer/06-graphics-pipeline-coordinate-spaces/01_graphics_pipeline_output.png' | relative_url }})

The visual output is not dramatically different yet because the model/view/projection matrices are still mostly identity. But the renderer is no longer only a screen-space experiment. It now has the skeleton of a graphics pipeline.

## What comes next

The next step is to make the transforms real:

- implement model translation/rotation/scale
- add a useful camera view
- implement perspective projection
- replace the current identity model/view/projection chain with those real matrices
- test how a local-space triangle moves through the pipeline
- eventually load the OBJ quad through the adapter instead of hand-authoring it

I am intentionally leaving full clipping for later. The renderer now has a place where clipping belongs. That is enough for this milestone.

## Related

- [CPU rasterizer (1): Triangle fill with barycentric coordinates and interpolated colors]({% post_url 2026-04-10-cpu-rasterizer-triangle-barycentric-colors %})
- [CPU rasterizer (2): Framebuffer text overlay (pixel stats and timings)]({% post_url 2026-04-11-cpu-rasterizer-framebuffer-text-overlay %})
- [CPU rasterizer (3): C++ type safety, semantic wrappers, and render-state boundaries]({% post_url 2026-04-23-cpu-rasterizer-cpp-type-safety-and-render-state %})
- [CPU rasterizer (4): Screen-space texture sampling with a checkerboard]({% post_url 2026-04-24-cpu-rasterizer-screen-space-texture-sampling %})
- [CPU rasterizer (5): Rendering architecture, asset boundaries, and frame stats]({% post_url 2026-04-25-cpu-rasterizer-rendering-architecture-refactor %})
