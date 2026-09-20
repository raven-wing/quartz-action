---
title: Build time and caching
tags:
  - reference
---

Why the first build is slow, why later ones aren't, and what makes the difference. None of this needs doing — the action handles it. It's here for when a build takes longer than you expected and you want to know why.

## The first build

Several minutes, usually around ten. Quartz up to v5.0.0 ships no prebuilt distribution, so its plugins are cloned and compiled from source — 42 of them in the default set, about 2.2 GB, even with a warm npm cache.

After that, the action caches the compiled plugins and later runs restore them in about a second. Each repository keeps its own cache.

## What makes a build slow again

Only a new `quartz-version`. That starts from nothing and pays the full cost once more.

Editing your config does not, despite the cache key being a hash of the whole file. The run misses that exact key, falls back to any cache for the same `quartz-version`, and `quartz plugin install` checks what it restored against Quartz's `quartz.lock.json` — resetting anything on the wrong commit. In practice the step finishes in seconds.

> [!note] A restored cache is checked, not trusted
> Because of that fallback, a run can be handed the plugins a different config built. `quartz plugin install` reads each installed plugin's commit, compares it against `quartz.lock.json`, and fetches and resets the ones that differ. What you build against is what the lockfile pins, whatever the cache happened to hold.

## Trimming your config won't speed it up

A reasonable thing to try, and it doesn't work.

`quartz plugin install` installs what Quartz's own `quartz.lock.json` pins, not what your config lists — the same 42 plugins are built whatever you write. Deleting a plugin from your config, or setting `enabled: false`, changes which plugins your *site uses*, not which ones get installed.

Trimming Quartz's lockfile instead isn't an option either: Quartz's own source imports named exports from the generated `.quartz/plugins/index.ts`, so dropping `og-image` fails the build outright with `No matching export ... CustomOgImagesEmitterName`.

So the cache is what makes builds fast, and there's no configuration that makes the first one cheaper.
