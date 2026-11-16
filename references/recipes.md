# 3D effect recipes

Three recipes, each checked on a live webto site. All three are ordinary WVF variants: every rule of the variant skill (`wvf.md`, `webgl.md`) applies, and each must pass `variant-check --strict`.

| Recipe | The owner needs | The page does | Libraries | Example |
|---|---|---|---|---|
| Scroll-scrubbed video | an MP4 of the product turning (a Blender turntable, a phone on a turntable) | GSAP ScrollTrigger moves `video.currentTime` with the scroll: the product turns as the visitor scrolls, and back | `gsap scroll-trigger` | `examples/hero-scroll-video.*` |
| Depth parallax | one photo or render, optionally its depth map | an OGL shader shifts each pixel by its depth with the pointer and a slow drift | `ogl` | `examples/hero-depth-parallax.*` |
| Jar label turn | a photo of the jar or bottle **without** its label (transparent background) and the flat label artwork | an OGL shader wraps the flat label around the jar and turns it with the scroll; GSAP leans the jar | `gsap scroll-trigger ogl` | `examples/hero-jar-label.*` |

## Rules for all three

- **Init guard first.** Put `if (root.dataset.wvInit) return; root.dataset.wvInit = "1";` at the very top of the script, before any GSAP tween or ScrollTrigger: a re-run must not create a second set.
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
- **Prepare the jar photo:** crop it so the jar is centred left-to-right (`labelInset` is one number for both sides) and touches the top and bottom of the image. Keep only the jar: no shadow, no table.
- **Measure the three numbers** on the cropped photo, in % of its height (`labelTop`, `labelHeight`) and width (`labelInset`):
  - Put the label on the **straight part of the glass** only. A label running over the shoulder or the bulge at the bottom looks wrong, because the glass sticks out past a flat strip. Find it by looking at the photo row by row (or scanning the alpha channel): the rows where the jar's width stays constant.
  - `labelInset` is the bare glass between the jar's edge and the label's edge on that straight part, usually 3–6 %.
  - The shader lowers the middle of the label by 3.5 % of its height; leave that much room above the bottom bulge.
- **Check the label artwork against the jar.** The artwork's width is the jar's circumference, so its height should be about `π × jar width ÷ artwork aspect`. When the straight part of the glass is shorter than that, the label shows squashed: tell the owner, and suggest a label printed for this jar (`artwork height ≈ π × straight-part width × (labelHeight ÷ jar width in the same units)`).
- **Only half the label faces the visitor.** A wordmark wider than about 45 % of the artwork wraps out of view at both ends at the start of the turn. The flat fallback (`object-cover`) shows more of it than the turning view does; that is expected.
- **Markup:** a box sized by the jar photo itself (the `<img>` in normal flow with a fixed height and `w-auto`, so the box takes the photo's own aspect ratio and the % coordinates match it); inside it the label `<img data-label>` laid flat over the front with an inline `style` built from the three numbers (this is what the editor and reduced motion show), and an empty `<div data-gl class="absolute inset-0">` that receives the canvas. Never fix the box's aspect ratio in CSS: any photo with another ratio is letterboxed and the label lands off the glass.
- **The headline** sits behind the jar from `sm:` up; on a phone the jar covers it, so put it above the jar there (in normal flow), as the example does.
- **Script order:** the `wvInit` guard first, then the GSAP tweens, then `if (!OGL) return;` and the renderer. After every `renderer.setSize(…)` set the canvas style back to `100%` (setSize writes pixel sizes, which go stale when the box resizes).
- **Shader** (one full-box `Triangle`): for each pixel inside the label's columns, `nx` in [-1, 1] across the label, `theta = asin(nx)` is the angle round the jar, so the label's horizontal coordinate is `u = fract(turn + theta / 2π + 0.5)` (the `+ 0.5` puts the middle of the artwork, usually the product name, in front at the start) and its vertical one runs from `labelTop` to `labelTop + labelHeight`, lowered a little in the middle (`cos(theta) * 0.035`) because the jar is seen from slightly above. Darken towards the sides (`0.45 + 0.55 * cos(theta)^0.7`), add a soft highlight band, and fade the outermost 6 % so the edge does not alias. `discard` outside the label.
- **Texture:** `new Texture(gl, { generateMipmaps: false, flipY: false })`. With OGL's default `flipY: true` the label comes out upside down.
- **Turning:** GSAP tweens a plain object (`{ a: 0 }` → `{ a: 1 }`, one full turn) with a scrubbed ScrollTrigger; the render loop copies it into the shader's uniform. A second scrubbed tween leans and lifts the jar box (`rotation`, `yPercent`): plain GSAP, so it also runs where WebGL does not.
- Hide the flat label (`visibility: hidden`) only once the canvas is drawing, so there is never a frame without a label.
