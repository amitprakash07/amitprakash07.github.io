# Game engine coursework images

Drop screenshots and diagrams for each assignment in the **numbered folder** for that post (chronological order). Name files however you like (`01.png`, `triangle-output.png`, etc.); they can be renamed when wired into the post.

**Markdown path pattern** (same as other blog images on this site):

```liquid
![Short description]({{ '/assets/img/blog/game-engine/NN-game-engineering-slug/your-file.png' | relative_url }})
```

## Folder ↔ post

| # | Folder | Post (`_posts/game-engineering/…`) | Date |
|---|--------|-------------------------------------|------|
| 1 | `01-game-engineering-asset-build-system` | `2014-10-05-game-engineering-visual-studio-solution-asset-build-system.md` | 2014-10-05 |
| 2 | `02-game-engineering-lua-integration` | `2014-10-19-game-engineering-lua-integration-cpp-opengl-directx.md` | 2014-10-19 |
| 3 | `03-game-engineering-mesh-vertex-index-buffer` | `2014-11-02-game-engineering-index-vertex-buffer-cross-platform.md` | 2014-11-02 |
| 4 | `04-game-engineering-mesh-lua-shared-shader` | `2014-11-16-game-engineering-mesh-loading-lua-shared-shader.md` | 2014-11-16 |
| 5 | `05-game-engineering-robust-asset-build` | `2014-11-30-game-engineering-asset-build-system-general.md` | 2014-11-30 |
| 6 | `06-game-engineering-mesh-builder-binary` | `2014-12-14-game-engineering-mesh-builder-binary-assets.md` | 2014-12-14 |
| 7 | `07-game-engineering-shader-constants` | `2014-12-28-game-engineering-shader-constants-uniforms.md` | 2014-12-28 |
| 8 | `08-game-engineering-shader-effect-builder` | `2015-01-11-game-engineering-shader-builder-effect-builder.md` | 2015-01-11 |
| 9 | `09-game-engineering-platform-independent-shaders` | `2015-01-25-game-engineering-platform-independent-shaders-refactor.md` | 2015-01-25 |
| 10 | `10-game-engineering-depth-buffer-3d` | `2015-02-08-game-engineering-depth-buffer-3d-transforms.md` | 2015-02-08 |
| 11 | `11-game-engineering-maya-exporter-alpha-blending` | `2015-02-22-game-engineering-maya-exporter-alpha-blending.md` | 2015-02-22 |
| 12 | `12-game-engineering-material-builder` | `2015-03-08-game-engineering-material-builder.md` | 2015-03-08 |

## Directory tree

```
assets/img/blog/game-engine/
├── README.md
├── 01-game-engineering-asset-build-system/
├── 02-game-engineering-lua-integration/
├── 03-game-engineering-mesh-vertex-index-buffer/
├── 04-game-engineering-mesh-lua-shared-shader/
├── 05-game-engineering-robust-asset-build/
├── 06-game-engineering-mesh-builder-binary/
├── 07-game-engineering-shader-constants/
├── 08-game-engineering-shader-effect-builder/
├── 09-game-engineering-platform-independent-shaders/
├── 10-game-engineering-depth-buffer-3d/
├── 11-game-engineering-maya-exporter-alpha-blending/
└── 12-game-engineering-material-builder/
```

Post titles in front matter are prefixed with **Game engineering:**. Published URLs are `/blogs/game-engineering/YYYY/MM/DD/<slug>/` (folder name plus date plus filename slug).

When you add images, put them in the matching folder and tell the assistant which post (or number **1–12**); the post will be updated to reference `/assets/img/blog/game-engine/<folder>/…`.
