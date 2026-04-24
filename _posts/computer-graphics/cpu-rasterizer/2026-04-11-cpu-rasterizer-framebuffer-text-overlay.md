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
- The text should become part of the final saved image, so the screenshot carries the same stats I saw while rendering.

Here is one of the framebuffer outputs with the overlay baked in:

![Framebuffer output with text overlay]({{ '/assets/img/blog/2026/cpu-rasterizer/01-triangle-barycentric/05_barycentric_interpolated_with_bounding_box.jpg' | relative_url }})

## Approach (high level)

1. **Collect stats** after rasterization.
2. **Convert the stats into a string** using the shared stats collector.
3. **Draw ASCII glyphs** from a tiny 8x8 bitmap font directly into the `ColorBuffer`.
4. **Write the framebuffer to disk** after the overlay pass.

## Where the overlay happens

The overlay is drawn after the geometry pass and before the framebuffer is saved:

```cpp
std::string stats = amit::StatsCollector::GetInstance().GetAllStatsAsString();

amit::graphics::ColorBuffer& color_buffer = render_outputs.GetColorBuffer();
amit::graphics::TextOverlay::Render(stats, 10, 20, amit::graphics::kRgb8ColorWhite, color_buffer);

amit::image::WriteColorBufferToPPM(color_buffer, "output.ppm");
```

That ordering matters. The triangle rasterizer and pixel shader produce the image first; the text overlay is just a final 2D write into the same color target.

## Bitmap font, not UI text

The implementation uses `font8x8_basic.h`, a simple fixed-size bitmap font. Each ASCII character maps to an 8-byte glyph, where each byte represents one row of 8 bits.

At render time:

- iterate over the input string one character at a time
- look up the glyph by ASCII code
- for every set bit in the 8x8 glyph, write one destination pixel into the color buffer

That gives a tiny monospace overlay without needing any OS or browser text rendering.

## The actual rendering loop

The core `Render()` function is intentionally straightforward:

```cpp
bool TextOverlay::Render(const std::string& text,
                         uint16_t x,
                         uint16_t y,
                         Rgb8 color,
                         ColorBuffer& color_buffer) {
    uint16_t x_iter = x;
    uint16_t y_iter = y;

    for (char ch : text) {
        if (ch == '\n') {
            y_iter += 10;
            x_iter = x;
            continue;
        }

        if (ch == '\t') {
            x_iter += 4;
            continue;
        }

        uint8_t ascii_code = static_cast<uint8_t>(ch);
        char* bitmap = font8x8_basic[ascii_code];
        RenderGlyph(bitmap, x_iter, y_iter, color, color_buffer);
        x_iter += 8;
    }

    return true;
}
```

The two small text-layout rules I liked here are:

- `\n` moves to the next line and resets `x`
- `\t` advances by a few columns without needing anything fancy

For a debug overlay, that is enough.

## Rendering a single glyph

Each glyph is drawn with a double loop over rows and bits:

```cpp
void TextOverlay::RenderGlyph(const char* bitmap,
                              uint16_t x,
                              uint16_t y,
                              Rgb8 color,
                              ColorBuffer& color_buffer) {
    for (uint16_t row = 0; row < 8; ++row) {
        for (uint16_t bit = 0; bit < 8; ++bit) {
            if (bitmap[row] & (1 << bit)) {
                color_buffer.SetImageData({x + bit, y + row}, color);
            }
        }
    }
}
```

This is exactly the kind of implementation I like in a CPU rasterizer project: simple, explicit, and easy to debug. There is no abstraction barrier hiding what is really happening. A glyph is just a tiny bitmap, and drawing text is just drawing a lot of pixels.

## What the stats string contains

The text itself comes from the shared stats system. In the current rasterizer flow, it includes things like:

- the render-stats bucket name
- rasterization duration in milliseconds
- pixel count

So the overlay is effectively a tiny HUD for the render pass. Because it is written into the framebuffer itself, the output image already contains the timing and count information when I save it.

## Why I like this approach

- It keeps the debug information close to the image itself.
- It avoids any dependency on external UI layers.
- It is easy to port anywhere the `ColorBuffer` exists.
- It reinforces the idea that the framebuffer is just another writable 2D image.

For now, this overlay is intentionally minimal: fixed-size font, fixed color, and top-left placement. That is enough for profiling and debugging, and it fits the spirit of the CPU rasterizer well.

## Related

- [CPU rasterizer (1): Triangle fill with barycentric coordinates and interpolated colors]({% post_url 2026-04-10-cpu-rasterizer-triangle-barycentric-colors %}) — screenshots whose overlays are described here.
