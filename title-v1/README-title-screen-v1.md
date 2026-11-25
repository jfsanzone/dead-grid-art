# Title screen v1 (landscape), built from cover v4b

The source is `brand/cover-v4/cover-v4b.jpg`, the full-cast cover: Jess, Kate, Coupler, Relay with the radio, Breaker with the laptop, Crown Station and the crossing signal.

## Files
| file | what |
|---|---|
| `title-bg-1920x1080.jpg` | Landscape background with no logo. This is the master. |
| `title-bg-1280x720.jpg` | The same, downscaled. |
| `title-bg-wide-2338x1080.jpg` | **iPhone version** (2.165:1, the same shape as 844x390). It is the 16:9 master with about 209 px of extra station on the left and field/rail on the right. Use it on phones wider than about 2:1. |
| `title-bg-wide-1688x780.jpg` | The wide version at 844x390 @2x. |
| `logo-dead-freight.png` | DEAD FREIGHT logo on transparency, 2048x602, no shadow. |
| `logo-dead-freight-1024.png` | The logo at 1024x301. |
| `logo-dead-freight-shadow.png` / `-1024-shadow.png` | The logo with the cover's soft dark drop shadow baked in, padded about 3%. Use it straight over the sky. |
| `title-mock-1920x1080.png`, `title-mock-844x390.png` | Background, logo and placeholder PLAY button. |
| `title-mock-safearea-844x390.png` | The same with the iPhone safe areas tinted and the logo and button boxes outlined. |
| `work/` | Outpaint passes, alignment, compositing, logo and mock scripts. |

There is **no @2x 3840x2160.** The heroes come from the cover's original pixels (864x1152). At 1920 that is roughly 1:1, but at 3840 the whole frame would be a 2x+ upscale, so the quality doesn't hold. If you need a 4K title, it needs a native high-res repaint.

## How the background was made
1. I took the logo-free v4b painting (`cover-v4/v4b-work/base4.png`) and re-applied the cover's finish grade (bloom, split tone, contrast, vignette, grain). It matches `cover-v4b.jpg` to about 2/255 outside the logo.
2. Two outpaint passes extended the scene. To the left: the Crown Station platform, a water tower and more grey-green horde. To the right: the rail track, telegraph poles and more horde. The sky goes up and the dirt ground goes down.
3. The original cover painting was pasted back on top with a feathered edge. All five heroes, Coupler (the worn-white robot with the cyan visor), the drone, the lead zombies, the station and the crossing are the cover's own pixels, so they are on-model. The outpaint only completed the boots at the old bottom edge.
4. A duplicate "CROWN STATION" sign that the outpaint added on the far-left building was blanked out, so there is still only one station sign.

No head, hand or held item is cropped. Jess's rifle, Kate's controller, Relay's radio and Breaker's laptop are all well inside frame and inside the iPhone safe area.

## Recommended placement (sizes in % of screen HEIGHT, so one rule covers 16:9 and the wide phone)
- **Background:**
  - Pick `title-bg-wide-*` when the screen aspect is above about 2.0, otherwise the 16:9 file.
  - Fit it to the screen height and centre it. Extra width is fine: crop the sides, because the edges are disposable station and field.
  - **Don't crop the top or bottom.** The composition uses the full height.
- **Logo:**
  - Centred horizontally, top edge at **1.5% H**, width **60.5% H**. Height follows the aspect ratio, about 17.8% H, so the logo spans y 1.5–19.3% H.
  - 1920x1080: 653x192 px at (633,16), which is 34% W.
  - 844x390 pt: 236x69 pt at (304,6), which is 28% W.
  - Don't make it bigger or lower: the drone's propellers start at 19.6% H, just under the logo's left half.
  - Use the `-shadow` file, or add a drop shadow of #140812 at 65%, blurred about 1.1% of the logo width and offset about 0.6% down.
- **PLAY button:**
  - Centred horizontally, centre at **88.5% H**, size **39% H wide × 11% H tall**, spanning y 83–94% H.
  - 1920x1080: 421x119 px at (749,896).
  - 844x390: 152x43 pt at (346,324). If you want a tap target of 44 pt or more, go to 12% H tall, about 47 pt; that still clears both the boots and the home indicator.
  - It sits on the empty dirt below the heroes. The lowest boot, Kate's, ends at about 82% H.
  - Placeholder style: Oswald Bold "PLAY" in cream #F1E4CC with a #2A1A10 stroke and hard shadow, on a #E0701F orange plate lit from the upper left, with a #2A1A10 edge and a thin cream keyline. These are the UI colours from the hero sheet.

## Safe area (iPhone landscape, 844x390 pt)
- **Sides:** notch / Dynamic Island inset of about **60 pt** on either side (7.1% W). The phone can be rotated either way, so keep both sides clear.
  - On the wide background, all of the 16:9 master's content starts 209 px in at 1080 height (about 76 pt at 390). So the heroes, logo, button, drone and crossing are all far inside the insets.
  - The insets cover only the added station roof, background zombies, field and rail. Jess, the leftmost hero, starts at about 33% W.
- **Bottom:** home indicator about **21 pt** (5.4% H). The button's bottom edge at 94% H clears it.
- **Top:** no inset in landscape with the status bar hidden. The logo starts at 1.5% H.
- See `title-mock-safearea-844x390.png`: red bands are the notch insets, the amber band is the home indicator, and the cyan boxes are the logo and button rects.
- **Known soft spot:** the outer 209 px of the wide version are a separate outpaint. They're a little softer and more painterly, but they sit almost entirely under the notch insets.
