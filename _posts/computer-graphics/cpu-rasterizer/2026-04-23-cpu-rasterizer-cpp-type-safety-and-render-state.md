---
title: "CPU rasterizer (3): C++ type safety, semantic wrappers, and render-state boundaries"
date: 2026-04-23
ai_assisted: true
tags:
  - software-engineering
  - graphics
---

This is the **third** post in my **CPU rasterizer** series for [Amit Labs](https://github.com/amitprakash07/amit-labs). Unlike the first two, this one is less about a new rendering feature and more about the **C++ design lessons** that came up while I was refactoring the rasterizer code: **strong typing**, **semantic wrappers**, **typed initialization**, and **render-state ownership**.

The trigger was simple. I was working in `Image2D`, and I wanted the constructor to make it hard to accidentally swap **width** and **height**.

## The original problem: `width` and `height` are not the same thing

An image constructor often looks like this:

```cpp
template <typename T>
class Image2D {
public:
    Image2D(std::uint32_t width, std::uint32_t height);
};
```

This is convenient, but it also means this compiles just fine:

```cpp
Image2D<Rgb8> color_buffer(height, width);
```

The compiler cannot help here because both parameters are the same type.

In my code, I wanted to make that mistake harder, especially as more helper functions and overloads were being added around image creation and buffer setup.

So I introduced distinct wrappers:

```cpp
struct Width {
    std::uint32_t value;
};

struct Height {
    std::uint32_t value;
};

template <typename T>
class Image2D {
public:
    Image2D(Width width, Height height);
};
```

Now this is no longer interchangeable by accident:

```cpp
Image2D<Rgb8> color_buffer(Width{800}, Height{600});   // correct
Image2D<Rgb8> color_buffer(Height{600}, Width{800});   // does not compile
```

That was the real goal: not just naming, but **compile-time protection**.

## The tempting abstraction that did not actually solve it

Once I started thinking in terms of types, I went one step more generic: what if I create a reusable **semantic wrapper** template and define `Width` and `Height` as aliases?

Conceptually:

```cpp
template <typename T>
struct SemanticValue {
    T value;
};

using Width = SemanticValue<std::uint32_t>;
using Height = SemanticValue<std::uint32_t>;
```

At first this looked elegant to me, but it quietly lost the safety I actually wanted.

`Width` and `Height` are now just **two names for the same type**.

That means the compiler sees this as legal again:

```cpp
void CreateImage(Width width, Height height);

Height h{600};
Width  w{800};
CreateImage(h, w);  // still compiles if Width and Height are aliases of the same type
```

This was the key lesson for me:

> A type alias improves readability, but it does **not** create a new type.

If the goal is vocabulary, aliases are fine. If the goal is **semantic correctness enforced by the compiler**, I need **distinct types**.

## Distinct types vs semantic aliases

That led to a clearer rule of thumb:

- Use **aliases** when I want better naming.
- Use **distinct wrapper structs** when I want the compiler to reject invalid substitutions.

For graphics code, this matters more than it first appears. `Width`, `Height`, `TileSize`, `Row`, `Column`, `Depth`, and `PixelCoordinate` may all be backed by integers, but they do not mean the same thing.

## Interval design: naming, boundaries, and type-level semantics

Another useful design discussion from the same refactor was about **intervals**. I wanted a reusable type for things like normalized coordinates and viewport-like domains, but my first naming instinct was `Range`, which is not ideal in modern C++ because `std::ranges` already uses that word for iterable sequences.

`Interval` turned out to be the better name because it matches the mathematical idea directly: values between two endpoints, possibly **open** or **closed** on either side.

The more interesting part was how much of that should live in the type itself.

A runtime version would store:

```cpp
template <typename T>
struct Interval {
    T min;
    T max;
    Boundary left;
    Boundary right;
};
```

But for fixed interval kinds like `[a, b]`, `(a, b]`, and `[a, b)`, it is cleaner to make the boundary kind part of the type:

```cpp
enum class IntervalBoundary {
    Open,
    Closed
};

template <
    typename T,
    IntervalBoundary LeftBoundary = IntervalBoundary::Closed,
    IntervalBoundary RightBoundary = IntervalBoundary::Closed>
struct Interval {
    T min;
    T max;
};
```

That way, `Interval<float, Closed, Closed>` and `Interval<float, Open, Closed>` are not just different values. They are different **types** with different semantics.

This led to another naming lesson that fits well with the `Width` / `Height` problem:

- generic templates are good for reusable machinery
- semantic aliases are good for intent
- aliases are **not** the same as strong typing

So names like `ClosedInterval<float>` are useful as generic building blocks, while names like `UnitInterval` or `PixelInterval` are useful when a graphics concept has a stable meaning in the codebase.

The biggest takeaway for me was that templates are most useful here when they encode **real semantic differences** like boundary behavior, not when they only rename the same thing in a more abstract way.

## Another C++ trap I hit the same day: `memset` is byte-based

While working on the depth buffer, I initialized the image with:

```cpp
depth_buffer_.FillImage(std::numeric_limits<float>::infinity());
```

That part was fine. The bug was in my `FillImage()` implementation:

```cpp
void FillImage(ImageDataType image_data) {
    memset(image_data_, image_data, sizeof(ImageDataType) * count);
}
```

I expected that to fill every `float` with `+infinity`, but `memset()` does **not** work that way. It fills memory with a **single byte pattern** repeated over and over. It does not copy the typed bit-pattern of a `float` value into each element.

So the fix was to use a typed fill:

```cpp
void FillImage(ImageDataType image_data) {
    std::fill_n(image_data_, count, image_data);
}
```

This was a good reminder for me that:

> `memset()` is for bytes, not for “set every element of an array to this typed value”.

## Render context was carrying too much authority

The other design issue I hit was in the rasterizer pipeline itself.

Originally, a single `RenderContext` owned:

- viewport
- color buffer
- depth buffer

and the pixel callback received the whole context.

That meant shader-like code could also touch depth, even though depth testing is supposed to be owned by the rasterizer. The problem was not just mutability. The bigger problem was **authority**.

So I split the responsibilities:

- `RenderConfig` for read-only setup like the viewport
- `RenderState` for mutable rasterizer-owned state like depth
- `RenderOutputs` for output targets like the color buffer

That way the rasterizer controls **visibility**, while the callback controls **shading and output writes**.

This felt like the right separation:

> Rasterizer decides whether a fragment survives.  
> Shader/output code decides what to write and where to write it.

## What I am taking forward

The most useful takeaway from this refactor was not a rendering formula. It was a C++ design reminder:

1. **If I want real type safety, I need distinct types.**
2. **If I want only naming clarity, aliases are enough.**
3. **Templates are most valuable when they encode real semantic differences, not just prettier names.**
4. **If I want to initialize typed arrays, use typed algorithms, not `memset()`.**
5. **If a rendering object owns too many responsibilities, split it by authority, not just by data members.**

For me, that was probably the most valuable part of this stage of the rasterizer project. The triangle still rasterizes the same way, but the code now says more clearly who is responsible for what, and the compiler helps more with mistakes that should never compile in the first place.

## Related

- [CPU rasterizer (1): Triangle fill with barycentric coordinates and interpolated colors]({% post_url 2026-04-10-cpu-rasterizer-triangle-barycentric-colors %})
- [CPU rasterizer (2): Framebuffer text overlay (pixel stats and timings)]({% post_url 2026-04-11-cpu-rasterizer-framebuffer-text-overlay %})
