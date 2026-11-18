# Changelog

## 0.2.0 — 2026-11-18

- **Real 3D models.** New recipe **3D model turn** (`examples/hero-model-glb.*`): the platform now hosts three.js for variants that declare `data-uses="three"`; a `.glb` (Draco OK, ≤ 10 MB) uploaded in the editor's "🧊 Model 3D" dialog is loaded with `THREE.loadModel(el)`, turned by the scroll and tilted by the pointer, with a poster photo wherever 3D is off. `references/recipes.md` covers what `THREE` holds (no loaders), the Blender export settings, markup, script order and the lint notes. Needs `variant-check` ≥ 0.1.62.
- Thumbnails now draw WebGL (OGL and three); the rules say so.

## 0.1.1 — 2026-11-16

Fixes from an end-to-end test in which a fresh agent built a whole template (a jam shop) with only these skills and shipped it to a staging site.

- **`examples/hero-jar-label.astro`:** the `wvInit` guard now comes before the GSAP tweens (a re-run created a second set); the jar box takes the photo's own aspect ratio (it was fixed at 5:8, so other photos were letterboxed and the label landed off the glass); `imageUrl` is read as the fallback photo, as the rules require; the canvas is set back to 100 % after every `setSize`; the flat label is hidden only after the first frame; on phones the headline sits above the jar instead of behind it; a dashed placeholder shows while there is no jar photo.
- **`examples/hero-scroll-video.astro`:** `wvInit` guard added.
- **`references/recipes.md`:** "Init guard first" is a rule for all recipes. The jar recipe now says how to prepare and measure the photo (crop it centred, put the label on the straight part of the glass only, leave room for the 3.5 % lift), how to check the artwork against the jar (a label taller than the straight part shows squashed), that a wordmark wider than about 45 % of the artwork wraps out of view at the start, and why the box must never have a fixed aspect ratio.

## 0.1.0 — 2026-11-14

- First release, split out of the variant skill (where the first two recipes shipped in 0.1.66). Three recipes for 3D-looking effects in webto variants, each tested on a live site: **scroll-scrubbed video** (`examples/hero-scroll-video.*`, all-keyframe MP4, one seek at a time), **depth parallax** (`examples/hero-depth-parallax.*`, OGL, optional Blender depth map) and the new **jar label turn** (`examples/hero-jar-label.*`, GSAP + OGL: a flat printed label wrapped around a label-less jar photo and turned by the scroll). Needs `variant-check` ≥ 0.1.61 and the html-to-webto-variant skill.
