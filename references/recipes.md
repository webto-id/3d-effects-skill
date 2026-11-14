# 3D effect recipes

Three recipes, each checked on a live webto site. All three are ordinary WVF variants: every rule of the variant skill (`wvf.md`, `webgl.md`) applies, and each must pass `variant-check --strict`.

| Recipe | The owner needs | The page does | Libraries | Example |
|---|---|---|---|---|
| Scroll-scrubbed video | an MP4 of the product turning (a Blender turntable, a phone on a turntable) | GSAP ScrollTrigger moves `video.currentTime` with the scroll: the product turns as the visitor scrolls, and back | `gsap scroll-trigger` | `examples/hero-scroll-video.*` |
| Depth parallax | one photo or render, optionally its depth map | an OGL shader shifts each pixel by its depth with the pointer and a slow drift | `ogl` | `examples/hero-depth-parallax.*` |
| Jar label turn | a photo of the jar or bottle **without** its label (transparent background) and the flat label artwork | an OGL shader wraps the flat label around the jar and turns it with the scroll; GSAP leans the jar | `gsap scroll-trigger ogl` | `examples/hero-jar-label.*` |

## Rules for all three

- **Degrade to an ordinary hero.** The owner fills the section later and will often lack the special asset, or switches to your variant from another hero whose photo sits in `imageUrl`. Read the photo from `images[0]`, falling back to `imageUrl`; without a video show that photo, without a depth map assume a centred subject, without a label show the jar alone. A variant that shows nothing without its special asset is broken (an owner's first test of these recipes showed exactly that).
- **The editor, thumbnails and reduced motion run no GSAP and no OGL.** What shows there is the plain markup: the video's poster, the photo, the jar with its flat label laid over the front. Make that state look finished.
- **Document every asset in the field's doc comment**, in words an owner understands ("[1] the label as printed: the full flat strip that wraps around the jar"). The editor shows these as the field's help.
- **Do not name a callback parameter `self`** (`onUpdate: (self) => …`, as in GSAP's own docs): `self` is a window alias and fails `script-forbidden`. Call it `st`.
- **With GSAP and OGL in one script**, OGL's `vec.set(x, y)` reads as a GSAP tween and fails `gsap-vars`: assign the components instead (`webgl.md`).
- Test on a phone before you promise it, iPhone above all.

## Scroll-scrubbed video

- **The MP4 must have every frame as a keyframe.** A normal export (from Blender, a phone or an editor) has one keyframe every few seconds; every scroll step then decodes up to a few hundred frames and the video stutters badly. Measured on a 1080p50 file with a keyframe every 3 s: median seek 155 ms; the same video re-encoded: 5 ms. Tell the owner to re-encode, or do it for them:

  ```
  ffmpeg -i input.mp4 -vf "scale=1280:-2,fps=30" -c:v libx264 -preset slow -crf 26 \
    -g 1 -keyint_min 1 -pix_fmt yuv420p -an -movflags +faststart output.mp4
  ```

  1280 px at 30 fps is enough for a full-screen scrub and usually comes out smaller than the original; 3–8 seconds is plenty.
- From Blender: animate the product (or the camera) one full turn, render to an image sequence or MP4, then run the command above. Keep the background the colour of the section so the edges disappear.
- Markup: `<video data-scrub muted playsinline preload="auto" poster={photo} src={v.src}>` from `video(videoUrl)` (`wvf.md` §5), inside a `sticky top-0 h-screen` box in a section 2–3 screens tall (`h-[250vh]`). Only when there is a video; otherwise a normal-height section with the photo.
- Script: one `ScrollTrigger` with `scrub: true` whose `onUpdate` computes the target time, and **one seek at a time**: set `currentTime` only when the previous seek has fired `seeked`, keeping the latest target. Piling up seeks is what makes slow phones jerk.
- iOS Safari may not load a muted video's frames until it has played once: check on a real iPhone.

## Depth parallax

- From Blender: render the colour image, then the **Mist** or **Z** pass as a grayscale image (white = near, black = far) at the same resolution and camera. Normalise it so the subject is light and the background dark.
- Markup: the colour render as the poster `<img data-gl-poster crossorigin="anonymous" data-edit-image="images.0.url">`; the depth map, only when there is one, as a second `<img data-gl-depth crossorigin="anonymous" data-edit-image="images.1.url" class="hidden">`.
- Shader: `uv + (pointer + drift) * (depth - 0.35) * 0.045`, with `object-cover` UVs. Without a depth map, a radial depth around the centre. Keep the shift small (≤ 0.05): more tears the edges. Add a slow time drift so phones, which have no pointer, still see it move.
- Without a real depth map the effect is subtle; say so to the owner instead of promising 3D.

## Jar label turn

No 3D tool needed: the jar is a photo and the label is the flat file the owner already sent to the printer.

- **Assets:** `images[0]`, the jar or bottle **without** its label, upright and straight-on, on a transparent background (PNG/WebP); `images[1]`, the label as printed, the full strip that goes all the way round (its width is the jar's circumference). Three number fields place the label on the photo, in % of the photo: `labelTop`, `labelHeight`, `labelInset` (the bare glass at each side).
- **Markup:** a box with the jar photo's aspect ratio; inside it the jar `<img>`, the label `<img data-label>` laid flat over the front with an inline `style` built from the three numbers (this is what the editor and reduced motion show), and an empty `<div data-gl>` that receives the canvas.
- **Shader** (one full-box `Triangle`): for each pixel inside the label's columns, `nx` in [-1, 1] across the label, `theta = asin(nx)` is the angle round the jar, so the label's horizontal coordinate is `u = fract(turn + theta / 2π + 0.5)` (the `+ 0.5` puts the middle of the artwork, usually the product name, in front at the start) and its vertical one runs from `labelTop` to `labelTop + labelHeight`, lowered a little in the middle (`cos(theta) * 0.035`) because the jar is seen from slightly above. Darken towards the sides (`0.45 + 0.55 * cos(theta)^0.7`), add a soft highlight band, and fade the outermost 6 % so the edge does not alias. `discard` outside the label.
- **Texture:** `new Texture(gl, { generateMipmaps: false, flipY: false })`. With OGL's default `flipY: true` the label comes out upside down.
- **Turning:** GSAP tweens a plain object (`{ a: 0 }` → `{ a: 1 }`, one full turn) with a scrubbed ScrollTrigger; the render loop copies it into the shader's uniform. A second scrubbed tween leans and lifts the jar box (`rotation`, `yPercent`): plain GSAP, so it also runs where WebGL does not.
- Hide the flat label (`visibility: hidden`) only once the canvas is drawing, so there is never a frame without a label.
