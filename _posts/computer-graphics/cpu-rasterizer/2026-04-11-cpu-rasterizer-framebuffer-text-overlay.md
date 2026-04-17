---
title: "CPU rasterizer (2): Framebuffer text overlay (pixel stats and timings)"
date: 2026-04-11
tags:
  - personal-project
  - computer-graphics
  - cpu-rasterizer
  - rasterization
  - amit-labs
  - "2026"
---

This post explains **how I draw the on-screen labels** you see in the CPU rasterizer screenshots—things like **pixel counts** and **timings**—as **text rendered into the same color buffer** after the triangle (or wireframe) pass, so the stats are **baked into the saved image** rather than coming from the web page.

The implementation lives in the same project as [CPU rasterizer (1): Triangle fill with barycentric coordinates and interpolated colors]({% post_url 2026-04-10-cpu-rasterizer-triangle-barycentric-colors %}) under `source/projects/cpu_rasterizer`.

## What we are solving

- After rasterizing geometry, I want a **small, monospace-style readout** in a corner of the framebuffer.
- The font is **not** OS text rendering in the browser—it is **my own bitmap / glyph blit** into the RGB buffer I already own.

## Approach (high level)

1. **Bitmap font** — a tiny atlas (or a few rows) of glyphs; each character maps to a rectangle of source texels (really just bytes in a header or embedded array).
2. **String layout** — format numbers and labels with `std::snprintf` (or similar), then iterate UTF-8 or ASCII bytes.
3. **Blit** — for each glyph pixel that is “ink”, write a **contrasting RGB8** into the destination rectangle in the framebuffer (with a simple shadow or outline if you want legibility on arbitrary backgrounds).
4. **Placement** — fixed margin from top-left (or whichever corner you prefer); keep y advance constant for a single-line HUD.

## Code pointers (to be expanded)

I will flesh this out with the exact structs and draw calls I use in Amit Labs (glyph stride, kerning if any, and where I hook the overlay after `present` / screenshot). For now, treat this as the **anchor post** the series links to.

## Related

- [CPU rasterizer (1): Triangle fill with barycentric coordinates and interpolated colors]({% post_url 2026-04-10-cpu-rasterizer-triangle-barycentric-colors %}) — screenshots whose overlays are described here.
