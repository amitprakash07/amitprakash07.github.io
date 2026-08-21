---
title: "CPU rasterizer (9): SDL windowing, interactive camera, and Dear ImGui"
date: 2026-08-20
ai_assisted: true
tags:
  - graphics
  - rendering
  - sdl
  - imgui
  - rasterization
---

This is the **ninth** post in my **CPU rasterizer** series for [Amit Labs](https://github.com/amitprakash07/amit-labs). Up to now the demos were mostly **offline**: rasterize a frame, write a PPM, inspect the image. That is still the right loop for learning the pipeline itself. But the next Labs steps (volume rendering, more camera work, later neural rendering) need a **window**, a **frame loop**, and a way to **tweak parameters** without rebuilding.

This post is about that bridge: **SDL3** for windowing and present, a small reusable `sdl_windowing` library, keyboard camera fly-through, and **Dear ImGui** for settings and frame stats — without pretending the SDL/ImGui glue is the hard graphics problem.

![Interactive Cornell Box with SDL present and Dear ImGui Settings/Output sidebar]({{ '/assets/img/blog/2026/cpu-rasterizer/09-sdl-imgui-windowing/01_sdl_imgui_cornell_interactive.png' | relative_url }})

*Cornell Box from the CPU rasterizer, presented live. Left: scene color buffer. Right: ImGui Settings (move speed, save actions) and Output (frame stats).*

## Why this step

PPM dumps are great for blog images of the **renderer**. They are awkward for day-to-day iteration once you want to move the camera or dial a speed.

I wanted:

- the same CPU `ColorBuffer` as before
- a window that shows it every frame
- WASD / arrows (and Q/E) to fly the camera
- UI for settings and live stats
- still a path to save images for posts

SDL + ImGui are familiar tools. The point for Labs is the **seam**: keep the rasterizer honest, keep windowing reusable for the next project (ray marching can link the same static lib).

## Layout: scene vs UI

The live window is wider than the framebuffer:

```text
+---------------------------+------------------+
| CPU ColorBuffer (scene)   | Settings (ImGui) |
| same size as before       |------------------|
|                           | Output (ImGui)   |
+---------------------------+------------------+
```

- The **rasterizer viewport** stays 800×600 (or whatever the demo uses).
- ImGui sits in a **right sidebar**, not over the image.
- Frame stats show in ImGui **Output**. The old 8×8 framebuffer text overlay stays as a capability, but the interactive path does not burn it into the live buffer.

That matches how I want to work: clean scene on the left, tooling on the right.

## What landed in the repo

### `sdl_windowing`

Opt-in static library under `source/core/sdl_windowing` (not part of the `core` INTERFACE, so coding problems and other targets do not pull SDL).

It owns:

- SDL window + renderer
- streaming texture upload from `ColorBuffer` (`LockTexture` path)
- interactive loop: poll → update/render callback → blit scene → ImGui → present
- keyboard sampling (WASD / arrows / Q/E), with movement ignored while ImGui wants the keyboard

### Settings registry

A small ImGui-agnostic registry in `common` (`SettingsRegistry`: float / bool / int with label and ranges). The Cornell demo registers **Move Speed**; ImGui just walks the list and draws widgets.

That split matters: the next app can reuse the registry even if the present backend changes.

### Dear ImGui + SDL_Renderer

The Labs present path is **SDL_Renderer**, so ImGui uses the **SDL3 + sdlrenderer3** backends — not DX12. ImGui draws after the color buffer blit and before `SDL_RenderPresent`.

### Save for blog posts

Two different save buttons, on purpose:

| Action | What it captures | Good for |
|--------|------------------|----------|
| **Save PPM** | Extended buffer: scene on the left, 8×8 stats panel on the right | Renderer / pipeline posts |
| **Save Screenshot** | Full SDL window via `SDL_RenderReadPixels` (scene + ImGui) | Tooling / ImGui posts |

Files go next to the executable (`tmp/x64/bin/...`) using `SDL_GetBasePath()`, so they do not land in the CMake `bld` working directory by accident.

The screenshot at the top of this post is exactly that second path.

## Camera in the loop

The Cornell demo loads the OBJ once, then each frame:

1. clears color + depth
2. rebuilds MVP from a mutable `CameraView`
3. rasterizes
4. updates the ImGui output string

WASD / arrows move along view forward / strafe; Q/E move up / down. Position and look-at translate together so the view direction stays stable while you fly.

## What this is not

This is not a game engine architecture pass. The loop is prototype-shaped on purpose. The learning value for me here was less “discover SDL” and more:

- when to leave PPM-only behind
- where to cut a reusable windowing library
- how to keep renderer output and UI tooling on separate surfaces
- how to save the right artifact for the right blog post

## Next

With a live window and tunable settings, the rasterizer is in a better place for interactive experiments. The longer Labs arc still points toward volume rendering and beyond; this was the plumbing step so those experiments are not stuck on “write a PPM and reopen the file.”

## Related posts

- [CPU rasterizer (1): Triangle fill with barycentric coordinates and interpolated colors]({% post_url 2026-04-10-cpu-rasterizer-triangle-barycentric-colors %})
- [CPU rasterizer (2): Framebuffer text overlay (pixel stats and timings)]({% post_url 2026-04-11-cpu-rasterizer-framebuffer-text-overlay %})
- [CPU rasterizer (4): Screen-space texture sampling with a checkerboard]({% post_url 2026-04-24-cpu-rasterizer-screen-space-texture-sampling %})
- [CPU rasterizer (5): Rendering architecture, asset boundaries, and frame stats]({% post_url 2026-04-25-cpu-rasterizer-rendering-architecture-refactor %})
- [CPU rasterizer (6): Graphics pipeline, clip space, and coordinate-safe primitives]({% post_url 2026-05-25-cpu-rasterizer-graphics-pipeline-and-coordinate-spaces %})
