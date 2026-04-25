---
title: "CPU rasterizer (5): Rendering architecture, asset boundaries, and frame stats"
date: 2026-04-25
ai_assisted: true
tags:
  - graphics
  - rendering
  - architecture
---

This is the **fifth** post in my **CPU rasterizer** series for [Amit Labs](https://github.com/amitprakash07/amit-labs). The previous post added screen-space texture sampling with a generated checkerboard. That was the visible feature. The work after that was less visible, but probably more important for the long-term shape of the project.

I spent this step cleaning up some architectural boundaries:

- separating asset loading from renderer-owned mesh contracts
- creating a small `graphics_common` layer for shared graphics data structures
- changing `RenderOutputs` into a singular `RenderOutput`
- making `RenderFrame` own render outputs and frame-wide state
- turning `DrawContext` into `DrawOptions`
- splitting frame-level stats from detailed draw-call stats

None of these changes make the triangle look dramatically different. But they make the renderer easier to grow.

## Why this refactor happened

After adding UVs and texture sampling, the next natural milestone is loading real geometry. I do not want to keep hand-authoring triangles forever. Eventually I want to load simple OBJ assets, build a camera path, add transforms, and move from screen-space experiments toward a more complete 3D pipeline.

That raised a design question:

> If I add an OBJ loader, should the loader return renderer-specific mesh data?

At first, that sounds convenient. The loader could parse an OBJ file and directly return `graphics::MeshData`. But the more I thought about it, the more that felt wrong.

An asset loader is not necessarily a renderer. The same OBJ data could be used for rendering, tessellation, mesh analysis, offline processing, or some future tool. If the loader depends directly on the graphics module, the dependency direction becomes awkward:

```text
asset_loading -> graphics
```

That makes the loader feel like a graphics subsystem, when really it should be a neutral input layer.

## Asset loading versus graphics data

The first fix was to let the OBJ loader own its own neutral data format:

```cpp
namespace asset_loading {
    struct ObjPosition {
        float x;
        float y;
        float z;
    };

    struct ObjTextureCoordinate {
        float u;
        float v;
    };

    struct ObjVertex {
        ObjPosition          position;
        ObjTextureCoordinate texture_coordinate;
        ObjColor             color;
    };

    struct ObjMeshData {
        std::vector<ObjVertex> vertices;
        std::vector<std::uint32_t> indices;
    };
}
```

That means `asset_loading` can parse OBJ files without knowing what the renderer wants to do with them.

Then I added a bridge layer:

```text
asset_loading -> graphics_asset_adapter -> graphics_common
```

The adapter is the place where intent becomes explicit:

```cpp
graphics::MeshData ConvertObjMeshToGraphicsMesh(
    const asset_loading::ObjMeshData& obj_mesh_data);
```

This keeps the dependency direction cleaner:

- `asset_loading` knows how to load OBJ data.
- `graphics_common` defines shared graphics contracts.
- `graphics_asset_adapter` translates loaded assets into renderer-facing mesh data.
- `graphics` consumes renderer-facing data and does rendering work.

That is a small split, but it matters. It lets each layer answer a different question.

## The role of graphics_common

While doing the asset loader work, another issue became obvious. Some graphics types were not really part of the renderer implementation. They were data contracts:

- `Rgb8`
- `FloatColor`
- `UVCoordinate`
- `VertexAttributes`
- `MeshData`
- `RenderPrimitive`

These are the types that multiple systems may reasonably share. The asset adapter needs them. The rasterizer needs them. The graphics module needs them. But not every consumer of those types should depend on the full graphics implementation.

So I pulled the common contracts into a separate module:

```text
core/graphics_common
```

The rule for this module is intentionally strict:

> `graphics_common` should contain shared data contracts, not renderer behavior.

That means `Image2D`, `Texture`, `Viewport`, and text overlay stay outside of it. Those are implementation concepts. But a vertex layout or color structure can live in `graphics_common` because they are lightweight contracts.

## RenderOutput is a unit

The next cleanup came from looking at the render frame API.

Previously, the naming around render outputs was confused. A type named `RenderOutputs` represented what was effectively one output. That felt wrong because a render frame can eventually have multiple outputs:

- a color buffer
- a depth output
- an object-id output
- a normal/debug output
- any future attachment-like target

So I changed the model:

```text
RenderOutput  = one output target
RenderFrame   = owns a list of RenderOutput objects
```

The output itself is configured by a small description:

```cpp
struct RenderOutputDescription {
    Width              width;
    Height             height;
    RenderOutputFormat format;
};
```

And `RenderFrame` owns the list:

```cpp
class RenderFrame {
public:
    RenderOutput& GetRenderOutput(std::uint32_t index = 0);

private:
    RenderConfig              render_config_;
    RenderState               render_state_;
    std::vector<RenderOutput> render_output_list_;
};
```

For now, there is still only one default RGB output. But the API now says the correct thing: a render frame can own multiple outputs, and each output is a unit.

## RenderConfig, RenderState, and RenderFrame

This also clarified a bigger boundary:

```text
RenderConfig: mostly immutable setup, such as the viewport
RenderState: mutable rendering state, such as depth
RenderOutput: output storage, such as a color buffer
RenderFrame: the object that groups them for one frame
```

I like this split because it separates different kinds of authority.

The rasterizer should read configuration. It should update render state when it owns the decision, such as depth testing. The shader callback can write to the selected render output. The frame ties those pieces together without making every piece global.

This is still not a full GPU-style device/context/framegraph architecture. It does not need to be. But the names now point in that direction.

## DrawOptions, not DrawContext

The stats refactor started with a bug in the design.

I had a draw object that carried both configuration and mutable stats state. That seemed convenient at first, but it became confusing once there were multiple draw calls. Some values are options:

- solid versus wireframe draw mode
- debug bounding-box flag
- whether to collect stats

Other values are runtime results:

- how many fragments were rasterized
- how many primitives were submitted
- how long a draw call took

Those should not live in the same place.

So `DrawContext` became `DrawOptions`:

```cpp
struct DrawOptions {
    PrimitiveDrawMode    primitive_draw_mode;
    DrawDebugFlag        draw_debug_flag;
    StatsCollectionLevel stats_collection_level;
};
```

That is a better name and a better responsibility. Options configure the draw. They do not own the draw's results.

## Frame stats versus draw-call stats

The final part of the refactor was the stats system.

The original framebuffer text overlay used the global stats dump directly. That worked when there was only one thing to display. But it does not scale. If a frame has 10 or 20 draw calls, the image should not contain a giant text dump of every draw-call bucket.

The overlay wants a compact frame summary:

![CPU rasterizer output with aggregated frame stats overlay]({{ '/assets/img/blog/2026/cpu-rasterizer/05-rendering-architecture-refactor/01_frame_stats_overlay.jpg' | relative_url }})

```text
Frame Stats
Draw Calls: 2
Vertices: 6
Indices: 0
Primitives: 2
Rasterized Pixels: ...
Draw Time: ... ms
```

Detailed diagnostics want something else:

```text
DrawCallStats 1
  Duration
  VertexCount
  PrimitiveCount
  RasterizedPixelCount

DrawCallStats 2
  ...
```

Those are two different audiences, so they became two different layers.

`RenderFrame` now owns the frame summary:

```cpp
class RenderFrame {
public:
    RenderFrameStats& GetRenderFrameStats();

private:
    RenderFrameStats render_frame_stats_;
};
```

Each draw call creates a scoped stats helper:

```cpp
ScopedDrawCallStats draw_stats_scope(
    render_frame_stats,
    draw_options.GetStatsCollectionLevel(),
    primitive);
```

The important part is that the scope does not own a new frame stats object. It holds a reference to the frame's stats. Each draw call gets its own temporary scope, but all scopes aggregate into the same `RenderFrameStats` for that frame.

The scope has three jobs:

- on construction, record draw-call-level input counts
- during rasterization, increment rasterized pixel counts
- on destruction, add the elapsed draw time

If detailed draw stats are enabled, the same scope also creates a `DrawCallStats` bucket through `StatsCollector`.

## Stats collection levels

I added a small stats level enum:

```cpp
enum class StatsCollectionLevel : std::uint8_t {
    kNone,
    kFrame,
    kDraw,
    kPrimitive
};
```

Only the first three levels matter right now:

- `kNone` means collect nothing.
- `kFrame` means collect only the frame summary for the overlay.
- `kDraw` means collect frame stats and detailed draw-call stats.

`kPrimitive` is intentionally not used yet. It is a placeholder for a possible future mode. I do not need primitive-level profiling today, but the naming leaves room for it without changing the public idea again.

## What the rasterizer call looks like now

The draw call now passes the frame stats explicitly:

```cpp
rasterizer.Rasterize(render_frame.GetRenderConfig(),
                     render_frame.GetRenderState(),
                     render_frame.GetRenderFrameStats(),
                     draw_options,
                     triangle,
                     shader);
```

This is clean enough for now. It makes the ownership obvious: the frame owns the stats, the options configure collection, and the rasterizer records what happened.

There is still a future design question here. Passing `RenderConfig`, `RenderState`, and `RenderFrameStats` separately hints that a future `RenderFrameContext` or direct `RenderFrame&` API may make sense. I am not changing that yet because the current shape is explicit and easy to reason about.

## What changed conceptually

The bigger shift is that the renderer now has clearer layers:

```text
asset_loading
    loads neutral asset data

graphics_common
    defines shared graphics contracts

graphics_asset_adapter
    converts loaded assets into renderer-facing contracts

graphics
    owns rendering concepts such as images, textures, outputs, frame state, and stats

cpu_renderer
    rasterizes primitives into fragments

projects/cpu_rasterizer
    wires a sample scene together
```

That separation is not about adding abstraction for its own sake. It is about keeping decisions near the place that owns them.

The asset loader should not decide what a renderer mesh is. The draw options should not own mutable stats. The frame overlay should not dump every diagnostic bucket. A render output should represent one output target, not a vague collection.

## What I am intentionally not doing yet

There are still obvious next steps:

- write detailed draw stats to a text file
- add real OBJ model rendering into the rasterizer path
- pass a more coherent frame context into the rasterizer
- add transform matrices and camera projection
- revisit texture sampling once perspective enters the pipeline

But I do not want to do all of that in this refactor. The point of this step was to make the current system ready for those changes without overbuilding the final architecture too early.

## Why this was worth doing

This refactor did not add a flashy visual feature. But it made the project easier to continue.

Before this cleanup, the code was starting to blur responsibilities:

- loading and graphics contracts were too close
- render output naming was misleading
- draw configuration and draw statistics were mixed
- the image overlay depended on a global detailed stats dump

After the cleanup, the responsibilities are easier to describe:

- load neutral assets
- adapt assets into graphics data
- keep common contracts small
- let the frame own frame-wide state
- let draw scopes collect draw-call measurements
- show compact frame stats in the image
- keep detailed diagnostics available separately

That is the kind of architecture I want in this project: small pieces, clear ownership, and enough room to grow into the next milestone.

## Related

- [CPU rasterizer (1): Triangle fill with barycentric coordinates and interpolated colors]({% post_url 2026-04-10-cpu-rasterizer-triangle-barycentric-colors %})
- [CPU rasterizer (2): Framebuffer text overlay (pixel stats and timings)]({% post_url 2026-04-11-cpu-rasterizer-framebuffer-text-overlay %})
- [CPU rasterizer (3): C++ type safety, semantic wrappers, and render-state boundaries]({% post_url 2026-04-23-cpu-rasterizer-cpp-type-safety-and-render-state %})
- [CPU rasterizer (4): Screen-space texture sampling with a checkerboard]({% post_url 2026-04-24-cpu-rasterizer-screen-space-texture-sampling %})
