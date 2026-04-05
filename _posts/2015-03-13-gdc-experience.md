---
layout: post
title: "GDC experience"
date: 2015-03-13
tags:
  - gdc
  - conference
  - directx
  - nvidia
  - vxgi
  - game-industry
source_wix: "https://amitprakash07.wixsite.com/home/post/2015/03/13/gdc-experience"
---

![GDC 2015]({{ '/assets/img/blog/2015/gdc-experience/gdc-2015-banner.jpg' | relative_url }})

**Game Developer Conference 2015** was a great experience: talks on cutting-edge **game engine** tech and meeting people passionate about games and graphics. I especially enjoyed sessions on **real-time graphics** for games—notably **DirectX 12** and NVIDIA’s **VXGI** talk on **voxelization**-based techniques for **dynamic lighting**.

**DirectX 12** adds more than raw features; it gives programmers more **flexibility and control** over renderers and shaders compared with **earlier Direct3D versions**. The memory model is also different from older APIs, which helps performance. Previously, some work that could have stayed on the **GPU** still bounced through the **CPU**; with DirectX 12, more of that load can stay on the GPU when available, which can improve frame rate and scaling.

NVIDIA’s **VXGI** for lighting stood out: it could noticeably improve **shadow clarity** (on the order of **50–60%** in the examples they showed) while still aiming for reasonable CPU/GPU cost and solid frame rates. Like any technique, it has **limitations**, but the tradeoffs were compelling.

Beyond the technical talks, I met several interesting people. **Matt Rix** talked about **decompiling** libraries and engine packages to learn from them—focused in part on **Unity**—and how that can inform your own work. I also met **Gus Class** from **Google** during a **Unity on Android** workshop using **Google Cardboard**.

GDC ’15 was valuable for **seeing where the industry was headed** and **meeting practitioners**. On **internships**, I didn’t find much that matched students: many booths were focused on **full-time** hiring, and **NVIDIA** wasn’t taking interns at the time when I asked.

**Advice for upcoming students (e.g. cohort C6):** if you can, get the **full access** pass—many of the best talks were only included there.
