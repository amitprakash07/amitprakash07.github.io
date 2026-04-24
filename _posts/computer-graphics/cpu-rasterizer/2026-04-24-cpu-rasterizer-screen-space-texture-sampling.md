---
title: "CPU rasterizer (4): Screen-space texture sampling with a checkerboard"
date: 2026-04-24
ai_assisted: true
tags:
  - graphics
  - rendering
---

This is the **fourth** post in my **CPU rasterizer** series for [Amit Labs](https://github.com/amitprakash07/amit-labs). The previous posts got the triangle fill, barycentric color interpolation, framebuffer output, and render-state boundaries into a better shape. This post adds the next visible milestone: sampling a generated checkerboard image through UV coordinates.

The goal here was intentionally small. I did not want to build a full material system, image loader, wrap modes, mipmaps, or filtering yet. I only wanted to prove this path:

```text
generated Image2D<Rgb8> -> Texture<Rgb8> -> interpolated fragment UV -> color buffer
```

That gives the rasterizer a real texture sampling step while keeping the implementation easy to inspect.

Here is the result: the first submitted triangle uses the generated checkerboard texture, while the second triangle keeps the interpolated vertex-color shader.

![Screen-space checkerboard texture sampled on one triangle]({{ '/assets/img/blog/2026/cpu-rasterizer/04-screen-space-texture-sampling/01_checkerboard_textured_triangle.jpg' | relative_url }})

## Starting point

At this stage, the rasterizer is still operating directly in screen space. The sample program creates triangle vertices with positions already in viewport coordinates, then derives a simple UV from those positions:

```cpp
auto make_vertex = [&viewport](const Point3D& position, const Rgb8& color) {
    const float uv_x = position.x / static_cast<float>(viewport.GetWidth().value);
    const float uv_y = position.y / static_cast<float>(viewport.GetHeight().value);

    return VertexAttributes{
        .position = position,
        .color = color,
        .uv = {uv_x, uv_y}
    };
};
```

This is not a final model-space or perspective-correct texturing setup. It is a fast screen-space proof of concept: if the vertex has a normalized UV, the rasterizer should carry that UV to each fragment, and the shader should be able to sample a texture with it.

## Image storage versus texture sampling

The checkerboard already exists as an `Image2D<Rgb8>`. That type is storage: it owns pixels, width, and height, and supports integer pixel access.

But a texture is a slightly different idea. A texture is something I want to sample with normalized coordinates:

```text
Image2D<T>: what value lives at pixel (x, y)?
Texture<T>: what value do I get at UV (u, v)?
```

So I added a small templated texture wrapper:

```cpp
template <typename TextureDataType>
class Texture {
public:
    explicit Texture(const Image2D<TextureDataType>& image)
        : image_(image) {
    }

    TextureDataType Sample(const UVCoordinate& coordinate) const {
        const uint32_t width  = image_.GetWidth().value;
        const uint32_t height = image_.GetHeight().value;

        const float clamped_u = ClampNormalizedCoordinate(coordinate.x);
        const float clamped_v = ClampNormalizedCoordinate(coordinate.y);

        const uint32_t image_x = static_cast<uint32_t>(clamped_u * static_cast<float>(width - 1));
        const uint32_t image_y = static_cast<uint32_t>(clamped_v * static_cast<float>(height - 1));

        return image_.GetImageData({image_x, image_y});
    }

private:
    const Image2D<TextureDataType>& image_;
};
```

The important decision is that `Texture<T>` does **not** own or copy the image. It holds a const reference to an `Image2D<T>`. That fits the current design because the image can come from anywhere:

- a procedural checkerboard generator
- a future file loader
- a generated debug image
- a scalar image such as a height map or mask

For this milestone, the checkerboard is generated procedurally:

```cpp
Image2D<Rgb8> checker_board =
    ImageGenerator::GetCheckerBoard(Width{200}, Height{200}, TileSize{8});

Texture<Rgb8> checker_texture(checker_board);
```

Later, a texture could have a file-loading constructor or factory. I did not add that now because it would mix two separate concerns: creating image data and sampling image data.

## Why the texture is templated

The first visible example is an RGB checkerboard, but a texture should not be tied to color only.

The image type is already templated:

```cpp
using ColorBuffer = Image2D<Rgb8>;
using DepthBuffer = Image2D<float>;
```

So the texture follows the same shape:

```cpp
Texture<Rgb8>  color_texture;
Texture<float> scalar_texture;
```

That means `Sample()` returns the same type stored in the image. A color texture samples an `Rgb8`. A scalar texture samples a `float`. The sampler does not need to know what the value means.

## Fragment layout

This was also the right moment to make the fragment payload more explicit.

The rasterizer computes visibility and interpolation. The fragment shader receives the result. Right now the fragment contains:

```cpp
struct RasterizedFragment {
    PixelCoordinate       coordinate;
    FloatColor            color;
    UVCoordinate          uv = {};
    BaryCentricCoordinate barycentric_coordinate;
    float                 depth = 0.0f;
    Kind                  fragment_kind;
};
```

That structure is useful because it separates a few concepts:

- `coordinate` says where this fragment lands in the output image.
- `color` is the interpolated vertex color path.
- `uv` is the interpolated texture coordinate path.
- `barycentric_coordinate` keeps the interpolation weights available for debugging or future attributes.
- `depth` is the value that survived the depth test.
- `fragment_kind` lets debug fragments, such as bounding boxes, keep their own color path.

I like this shape because it keeps the rasterizer output honest: a fragment is not only a pixel color. It is the set of per-fragment attributes that survived rasterization.

## Interpolating UVs

The first rasterizer post used barycentric coordinates for color interpolation. UVs follow the same rule:

```text
uv = alpha * uv0 + beta * uv1 + gamma * uv2
```

The helper is small:

```cpp
inline UVCoordinate InterpolateUV(const UVCoordinate& uv1,
                                  const UVCoordinate& uv2,
                                  const UVCoordinate& uv3,
                                  BaryCentricCoordinate barycentric_coordinate) {
    UVCoordinate interpolated_uv;
    interpolated_uv.x = uv1.x * barycentric_coordinate.alpha
                      + uv2.x * barycentric_coordinate.beta
                      + uv3.x * barycentric_coordinate.gamma;
    interpolated_uv.y = uv1.y * barycentric_coordinate.alpha
                      + uv2.y * barycentric_coordinate.beta
                      + uv3.y * barycentric_coordinate.gamma;
    return interpolated_uv;
}
```

Inside triangle rasterization, after the barycentric weights are known, the rasterizer now builds a fragment with interpolated UV:

```cpp
const UVCoordinate interpolated_uv =
    InterpolateUV(triangle.VertA().uv,
                  triangle.VertB().uv,
                  triangle.VertC().uv,
                  barycentric_coordinate);

RasterizedFragment fragment{
    .coordinate = {x, y},
    .color = interpolated_color,
    .uv = interpolated_uv,
    .barycentric_coordinate = barycentric_coordinate,
    .depth = interpolated_depth,
    .fragment_kind = RasterizedFragment::Kind::Primitive
};
```

This is still affine interpolation in screen space. That is fine for the current project stage because there is no perspective transform in the pipeline yet. When clip space and perspective projection enter the renderer, this will need to become perspective-correct interpolation.

## Two shaders, two triangles

The sample scene draws two triangles. To make the new texture path easy to verify, only the first submitted triangle uses the checkerboard shader. The second triangle stays on the original interpolated-color shader.

The color shader is the old behavior:

```cpp
auto color_shader = [&render_outputs](const RasterizedFragment& fragment) {
    ImageCoordinate image_coordinate{fragment.coordinate.x, fragment.coordinate.y};
    render_outputs.GetColorBuffer().SetImageData(image_coordinate, fragment.color.ToRgb8());
};
```

The checkerboard shader samples the texture for normal primitive fragments:

```cpp
auto checkerboard_shader =
    [&render_outputs, &checker_texture](const RasterizedFragment& fragment) {
        ImageCoordinate image_coordinate{fragment.coordinate.x, fragment.coordinate.y};

        if (fragment.fragment_kind == RasterizedFragment::Kind::BoundingBox) {
            render_outputs.GetColorBuffer().SetImageData(image_coordinate, fragment.color.ToRgb8());
            return;
        }

        render_outputs.GetColorBuffer().SetImageData(
            image_coordinate,
            checker_texture.Sample(fragment.uv));
    };
```

Then each draw call chooses the shader it wants:

```cpp
rasterizer.Rasterize(config, state, draw_context, first_triangle, checkerboard_shader);
rasterizer.Rasterize(config, state, draw_context, second_triangle, color_shader);
```

This is a small thing, but it is an important design direction. The rasterizer does not need to know whether the fragment will be shaded by vertex color, a texture lookup, or something more complex later. It just produces fragments. The shader decides how to turn those fragments into output pixels.

## What is intentionally missing

This version is deliberately simple:

- nearest-neighbor sampling only
- UVs clamped to `[0, 1]`
- no repeat or mirror wrap mode
- no bilinear filtering
- no mipmaps
- no image loading from disk
- no perspective-correct interpolation yet

Those are all real texture-system topics, but they are not needed to prove the first path. The current milestone is about connecting the pieces cleanly.

## What changed conceptually

Before this step, the rasterizer mostly answered:

> Which pixels are inside the triangle, and what interpolated color should they get?

After this step, the rasterizer can answer:

> Which fragments survived, what attributes do they carry, and how does a shader use those attributes?

That is the bigger shift. Texture sampling is the visible result, but the more useful internal change is that fragments now carry structured data. Color is one attribute. UV is another. Depth, barycentrics, and future normals can follow the same pattern.

## Related

- [CPU rasterizer (1): Triangle fill with barycentric coordinates and interpolated colors]({% post_url 2026-04-10-cpu-rasterizer-triangle-barycentric-colors %})
- [CPU rasterizer (2): Framebuffer text overlay (pixel stats and timings)]({% post_url 2026-04-11-cpu-rasterizer-framebuffer-text-overlay %})
- [CPU rasterizer (3): C++ type safety, semantic wrappers, and render-state boundaries]({% post_url 2026-04-23-cpu-rasterizer-cpp-type-safety-and-render-state %})
