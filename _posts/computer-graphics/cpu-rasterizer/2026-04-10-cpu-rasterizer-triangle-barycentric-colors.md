---
title: "CPU rasterizer (1): Triangle fill with barycentric coordinates and interpolated colors"
date: 2026-04-10
tags:
  - personal-project
  - computer-graphics
  - cpu-rasterizer
  - rasterization
  - amit-labs
  - "2026"
---

This is the **first** post in a **CPU rasterizer** series I am writing alongside my [Amit Labs](https://github.com/amitprakash07/amit-labs) work. Later posts will go deeper (depth, textures, and beyond). This one is about the **steps I took** to reach a point where I can **draw a filled triangle** and shade it with **per-vertex colors interpolated using barycentric coordinates**.

The C++ below is **illustrative** of each idea; the full wiring lives in the repo under `source/projects/cpu_rasterizer`.

## Motivation

I wanted to see how a triangle turns into pixels, without leaning on the GPU yet. The path was incremental: from wireframe edges to a correct inside test, then barycentrics, then color interpolation.

## Step 1 — Triangle edges

A triangle is three segments:

- Line(v0, v1)
- Line(v1, v2)
- Line(v2, v0)

Drawing only the edges makes orientation and vertex order easy to sanity-check before filling the interior.

```cpp
// Wireframe: rasterize each edge (uses the integer Bresenham routine from Step 2).
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

## Step 2 — Bresenham line drawing

I used an integer incremental line routine so edge stepping stays stable and cheap compared to naive per-pixel line math.

```cpp
#include <cstdint>
#include <cmath>

// Integer midpoint / Bresenham style (0 ≤ |slope| ≤ 1 case shown; mirror for steep lines.)
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
```

![Step 2 — lines via Bresenham]({{ '/assets/img/blog/2026/cpu-rasterizer/01-triangle-barycentric/01_triangle_edges.jpg' | relative_url }})

## Step 3 — Edge function (Orient2D)

For a point P relative to triangle edges, signed “areas” come from the edge function (orient2d):

- E0 = Orient2D(v1, v2, P)
- E1 = Orient2D(v2, v0, P)
- E2 = Orient2D(v0, v1, P)

Together these encode which side of each directed edge P lies on, and set up the same weights used for barycentric coordinates.

```cpp
// Implicit line ax + by + c = 0 through (ax,ay) -> (bx,by)
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
```

![Step 3 — edge function / orient2d intuition]({{ '/assets/img/blog/2026/cpu-rasterizer/01-triangle-barycentric/02_triangle_filled.jpg' | relative_url }})

## Step 4 — Bounding box and inside test

Scan only pixels in the triangle’s axis-aligned bounding box, and treat P as inside when all edge functions agree with the triangle’s winding (for consistent CCW/CW rules), e.g.:

`if (E0 >= 0 && E1 >= 0 && E2 >= 0)` (adjust signs to match your vertex order).

```cpp
#include <algorithm>

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
            if (e0 >= 0 && e1 >= 0 && e2 >= 0)  // flip to <= 0 everywhere if your winding is opposite
                plot_pixel(x, y);
        }
    }
}
```

![Step 4 — bounding box scan and inside test]({{ '/assets/img/blog/2026/cpu-rasterizer/01-triangle-barycentric/03_triangle_filled_with_bounding_box.jpg' | relative_url }})

## Step 5 — Barycentric coordinates

With

`Area = Orient2D(v0, v1, v2)` (double the triangle area, signed),

the barycentric weights are:

- α = E0 / Area  
- β = E1 / Area  
- γ = E2 / Area

At any interior pixel, α + β + γ = 1, and each weight tells you how “close” that pixel is to the opposite vertex.

```cpp
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
```

![Step 5 — barycentric weights visualized]({{ '/assets/img/blog/2026/cpu-rasterizer/01-triangle-barycentric/04_barycentric_interpolated.jpg' | relative_url }})

## Step 6 — Interpolating color

Given vertex colors C0, C1, C2 (as floats in [0, 1] per channel), the interpolated color is the same linear combination as any other per-vertex attribute:

**C = αC0 + βC1 + γC2**

That is how the triangle becomes a smooth blend instead of flat fills.

```cpp
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
```

![Step 6 — per-vertex colors blended with barycentrics]({{ '/assets/img/blog/2026/cpu-rasterizer/01-triangle-barycentric/05_barycentric_interpolated_with_bounding_box.jpg' | relative_url }})

## Step 7 — Color representation

- **ColorF** — float RGB in [0, 1] for the interpolation math.  
- **RGB8** — 8-bit channels for writing the framebuffer (e.g. PPM or similar).

```cpp
#include <algorithm>

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

![Step 7 — float color vs 8-bit framebuffer output]({{ '/assets/img/blog/2026/cpu-rasterizer/01-triangle-barycentric/05_barycentric_interpolated_with_bounding_box.jpg' | relative_url }})

## Closing

From here the same barycentric weights extend naturally to **depth**, **UVs**, and normals. This post stops where the **first milestone** felt solid: a **filled triangle** with **colors driven by barycentric interpolation**.