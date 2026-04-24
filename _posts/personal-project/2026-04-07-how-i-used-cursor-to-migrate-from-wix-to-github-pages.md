---
title: "How I used Cursor to migrate my Wix site to GitHub Pages"
date: 2026-04-07
ai_assisted: true
tags:
  - software-engineering
  - tools
  - AI
---

This post is a short write-up of how I moved my old Wix website into this GitHub Pages site, and where Cursor helped most during the migration.

The migration happened in stages. From the repo history, the first migration setup landed on **Mar 25, 2026**, and most of the content and layout work happened between **Apr 5 and Apr 7, 2026**. I wanted this post dated near the point where the migration was actually usable, which is why it sits here before the newer Amit Labs posts.

## Why I migrated

Wix was useful for getting a site online quickly, but over time I wanted a setup that was:

- easier to version in Git
- easier to edit as plain files
- better for long-form technical writing
- easier to restructure as the site evolved

GitHub Pages felt like a better long-term fit because the site itself becomes part of the codebase.

## What Cursor was useful for

Cursor was most helpful in the parts of the migration that were repetitive, structural, or spread across many files.

### 1. Rebuilding structure from scattered content

The old site had content spread across many pages and older blog entries. Cursor helped me:

- inspect existing pages and posts quickly
- suggest clean folder structure for Jekyll posts
- preserve the distinction between older archived posts and newer original writing
- keep categories, permalinks, and layouts consistent

That was especially useful because I was moving from a page-builder mindset to a file-and-template mindset.

### 2. Converting rough content into maintainable files

A lot of migration work is not “hard” in the algorithmic sense. It is just tedious:

- front matter
- titles
- categories
- links
- layout cleanup
- repeated formatting fixes

Cursor helped reduce that friction. Instead of hand-editing every piece in isolation, I could iterate on a structure and update the site in a more systematic way.

### 3. Preserving intent while changing the system

One thing I cared about was keeping the site faithful to what already existed, without freezing it in its old form. Cursor helped with that balance:

- some posts stayed as redirects or archival references to Wix
- some were migrated into cleaner Markdown posts
- new content could now be written directly for GitHub Pages

That let the site become both an archive and a living technical notebook.

## The hard part: exact migration was not always worth it

One important lesson from the migration was that moving content from Wix into Markdown was not always clean or even desirable.

In theory, I could have tried to migrate every old post into GitHub Pages as a fully local Markdown page. In practice, that was difficult for a few reasons:

- Wix did not make it easy to extract content in a clean, structured way
- formatting, spacing, embeds, and media often changed during migration
- some older posts looked correct on Wix because of Wix's own rendering engine, but did not translate neatly into static Markdown and Jekyll layouts
- trying to reproduce everything exactly started turning into restoration work rather than useful migration work

That led to an important final decision:

> for many older posts, preserving the original Wix version through a redirect was better than forcing a lossy rewrite into GitHub Pages.

This ended up being the right tradeoff for me. It kept the historical posts readable in their original format, while still letting the new site act as the main navigation layer and home for newer writing.

So the migration did **not** end as “move every page into Markdown no matter what.” It ended as a hybrid model:

- keep older posts accessible, often by redirecting to the original Wix page
- migrate or rewrite content only when it made sense
- write all new work natively in GitHub Pages

## What I would do differently next time

If I were doing this again, I would make the redirect decision much earlier.

I spent multiple days over the course of roughly two weeks experimenting with ways to migrate older Wix content more faithfully into Markdown and Jekyll. That was useful up to a point, because it helped me understand the limits of the source platform. But in hindsight, once it became clear that the content was hard to parse cleanly and difficult to preserve exactly, the better move would have been to stop pushing for a perfect conversion.

My rule of thumb now would be:

> if the source platform does not allow clean extraction and exact migration starts consuming serious time, prefer redirects early.

That approach is:

- faster
- more convenient
- less lossy
- more respectful of the original formatting

For this migration, that would have saved time and reduced a lot of unnecessary effort spent trying to reproduce posts that already existed in a usable form on Wix.

## What Cursor did not replace

Cursor helped a lot with execution, but the important product decisions were still manual:

- what sections should exist
- what should be removed
- what should stay archived
- how the homepage should describe me
- which posts should be treated as current work versus older coursework

Those decisions still required judgment. The tool accelerated the migration, but it did not decide the shape of the site for me.

## The main technical changes in the migration

By the end of the migration, the site had a structure that made more sense for ongoing use:

- posts organized into Jekyll folders
- category defaults in `_config.yml`
- reusable layouts
- a blog index organized by year and topic
- space for newer original posts, not only archived material

That was the bigger win for me. The migration was not just about moving pages from one platform to another. It was about turning the website into something I could keep evolving in Git.

## What I liked most about this workflow

The best part for me was that the site stopped feeling like a separate publishing system and started feeling like part of my normal engineering workflow.

That means:

- content lives in the repo
- changes are reviewable
- structure is explicit
- writing and engineering work can live closer together

For a site that is mainly about projects, technical writing, and reflection, that is a much better fit.

## Closing

I do not think every website needs an AI-assisted migration, and I do not think Cursor magically solves messy content problems. But for this kind of migration, where there is a mix of restructuring, cleanup, repeated editing, and light content transformation, it was genuinely useful.

The end result is not just “my old Wix site, moved.” It is a site that is easier for me to maintain, easier to grow, and better aligned with the way I already work.
