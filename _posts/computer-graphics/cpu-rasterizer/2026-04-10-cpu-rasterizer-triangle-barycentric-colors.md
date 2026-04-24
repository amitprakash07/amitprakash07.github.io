---
title: "CPU rasterizer (1): Triangle fill with barycentric coordinates and interpolated colors"
date: 2026-04-10
ai_assisted: true
tags:
  - graphics
  - rendering
---

This is the **first** post in a **CPU rasterizer** series I am writing alongside my [Amit Labs](https://github.com/amitprakash07/amit-labs) work. I wanted to start from a simple place and build it step by step. In this post, that means three things: **wireframe edges**, **inside tests with an edge function and bounding box**, and then **barycentric weights for smooth per-vertex colors**.

The C++ below is mainly there to explain the ideas clearly. The full wiring lives in the repo under `source/projects/cpu_rasterizer`.

## Motivation

I wanted to see how a triangle turns into shaded pixels without depending on the GPU yet. For me, the right order was: edges first, then “is this pixel inside?”, and then the same signed areas again as **barycentric coordinates** for interpolation.

## Step 1 — Triangle edges

A triangle is three segments: Line(v0, v1), Line(v1, v2), Line(v2, v0). Drawing only the edges makes winding and vertex order easy to check before you fill the interior.

Edges are stepped with an integer **Bresenham** line routine; the outline calls it three times.

```cpp
#include <cstdint>
#include <cmath>

void DrawLineBresenham(int32_t x1, int32_t y1, int32_t x2, int32_t y2, auto&& plot) {
    int32_t dx = std::abs(x2 - x1), dy = std::abs(y2 - y1);
    int32_t sx = (x1 < x2) ? 1 : -1;
    int32_t sy = (y1 < y2) ? 1 : -1;
    int32_t err = dx - dy;
    for (;;) {
        plot(x1, y1);
        if (x1 == x2 && y1 == y2) break;
        int32_t e2 = 2 * err;
        if (e2 > -dy) { err -= dy; x1 += sx; }
        if (e2 < dx)  { err += dx; y1 += sy; }
    }
}

void DrawTriangleOutline(int32_t x0, int32_t y0,
                         int32_t x1, int32_t y1,
                         int32_t x2, int32_t y2,
                         auto&& plot) {
    DrawLineBresenham(x0, y0, x1, y1, plot);
    DrawLineBresenham(x1, y1, x2, y2, plot);
    DrawLineBresenham(x2, y2, x0, y0, plot);
}
```

![Step 1 — triangle drawn as three edges (wireframe)]({{ '/assets/img/blog/2026/cpu-rasterizer/01-triangle-barycentric/01_triangle_edges.jpg' | relative_url }})

## Step 2 — Edge function and fill

For a point P relative to the triangle, signed “areas” come from the **edge function** (orient2d): E0 = Orient2D(v1, v2, P), E1 = Orient2D(v2, v0, P), E2 = Orient2D(v0, v1, P). Together they say which side of each directed edge P lies on. A pixel is **inside** when all three agree with your winding (for example `E0 >= 0 && E1 >= 0 && E2 >= 0`—flip to `<= 0` everywhere if your order is opposite).

To avoid scanning the whole framebuffer, restrict the double loop to the triangle’s **axis-aligned bounding box**; that is the only change between the two renders below.

```cpp
#include <algorithm>

struct ImplicitLine {
    float a, b, c;
};

ImplicitLine MakeLine(float ax, float ay, float bx, float by) {
    return {ay - by, bx - ax, -(ay - by) * ax - (bx - ax) * ay};
}

float Orient2D(ImplicitLine L, float px, float py) {
    return L.a * px + L.b * py + L.c;
}

float Orient2D(float ax, float ay, float bx, float by, float px, float py) {
    return Orient2D(MakeLine(ax, ay, bx, by), px, py);
}

struct Point2 {
    float x, y;
};

void RasterizeTriangleFill(const Point2& v0, const Point2& v1, const Point2& v2,
                           auto&& plot_pixel) {
    int x0 = static_cast<int>(std::min({v0.x, v1.x, v2.x}));
    int x1 = static_cast<int>(std::max({v0.x, v1.x, v2.x}));
    int y0 = static_cast<int>(std::min({v0.y, v1.y, v2.y}));
    int y1 = static_cast<int>(std::max({v0.y, v1.y, v2.y}));

    for (int y = y0; y <= y1; ++y) {
        for (int x = x0; x <= x1; ++x) {
            float e0 = Orient2D(v1.x, v1.y, v2.x, v2.y, float(x), float(y));
            float e1 = Orient2D(v2.x, v2.y, v0.x, v0.y, float(x), float(y));
            float e2 = Orient2D(v0.x, v0.y, v1.x, v1.y, float(x), float(y));
            if (e0 >= 0 && e1 >= 0 && e2 >= 0)
                plot_pixel(x, y);
        }
    }
}
```

Filled triangle (inside test only):

![Step 2a — filled triangle using edge functions]({{ '/assets/img/blog/2026/cpu-rasterizer/01-triangle-barycentric/02_triangle_filled.jpg' | relative_url }})

Same fill, with the **bounding box** drawn so the scan region is obvious:

![Step 2b — same fill with bounding box]({{ '/assets/img/blog/2026/cpu-rasterizer/01-triangle-barycentric/03_triangle_filled_with_bounding_box.jpg' | relative_url }})

## Step 3 — Barycentric coordinates and color

With `Area = Orient2D(v0, v1, v2)` (twice the signed triangle area), the barycentric weights are α = E0 / Area, β = E1 / Area, γ = E2 / Area. At any interior pixel, α + β + γ = 1. Vertex colors C0, C1, C2 in **[0, 1]** per channel interpolate the same way as any other attribute: **C = αC0 + βC1 + γC2**. Convert to **RGB8** for the framebuffer when you commit pixels.

```cpp
#include <algorithm>

struct Barycentric {
    float alpha, beta, gamma;
};

Barycentric ComputeBarycentric(const Point2& v0, const Point2& v1, const Point2& v2,
                               float px, float py) {
    float area = Orient2D(v0.x, v0.y, v1.x, v1.y, v2.x, v2.y);
    float e0   = Orient2D(v1.x, v1.y, v2.x, v2.y, px, py);
    float e1   = Orient2D(v2.x, v2.y, v0.x, v0.y, px, py);
    float e2   = Orient2D(v0.x, v0.y, v1.x, v1.y, px, py);
    return {e0 / area, e1 / area, e2 / area};
}

struct ColorF {
    float r, g, b;
};

ColorF InterpolateColor(const ColorF& c0, const ColorF& c1, const ColorF& c2,
                        const Barycentric& w) {
    return {
        w.alpha * c0.r + w.beta * c1.r + w.gamma * c2.r,
        w.alpha * c0.g + w.beta * c1.g + w.gamma * c2.g,
        w.alpha * c0.b + w.beta * c1.b + w.gamma * c2.b,
    };
}

struct Rgb8 {
    uint8_t r, g, b;
};

Rgb8 FloatToRgb8(const ColorF& c) {
    auto clamp01 = [](float x) { return std::max(0.f, std::min(1.f, x)); };
    return {
        static_cast<uint8_t>(std::lround(255.f * clamp01(c.r))),
        static_cast<uint8_t>(std::lround(255.f * clamp01(c.g))),
        static_cast<uint8_t>(std::lround(255.f * clamp01(c.b))),
    };
}
```

Interpolated vertex colors (barycentric visualization):

![Step 3a — barycentric interpolation]({{ '/assets/img/blog/2026/cpu-rasterizer/01-triangle-barycentric/04_barycentric_interpolated.jpg' | relative_url }})

Same shading with the **bounding box** overlaid:

![Step 3b — barycentric interpolation with bounding box]({{ '/assets/img/blog/2026/cpu-rasterizer/01-triangle-barycentric/05_barycentric_interpolated_with_bounding_box.jpg' | relative_url }})

## Closing

From here the same weights extend naturally to **depth**, **UVs**, and normals. For this first milestone, I wanted to stop at the point where the basic triangle fill felt solid: a **filled triangle** with **colors driven by barycentric interpolation**.
