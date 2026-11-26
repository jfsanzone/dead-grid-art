# drone-strike-v2: Kate's Drone Strike, on-model drone with 16 headings and a painterly explosion

v2 replaces the v1 drone, bomb, explosion and scorch. v1 is unchanged in `../drone-strike-v1/`. Everything is a transparent PNG at @2x, with 1x copies (= world px) in `1x/`. All sizes, anchors, offsets and timings are in `drone-strike-v2.json`.

## What changed from v1
- **Drone:** rebuilt in 3D from Kate's cover drone (cover-v4b) and hero sheet. It has an off-white panelled shell, a cyan visor on the front-top, a camera pod with a cyan lens under the nose, four gunmetal arms, motor pods with cyan lights, and translucent rotor-blur discs. It's rendered at the map camera (35°) with the towers-aim-v1 pipeline: painted cel bands, #2A1A10 ink, rim softening, the same grade and texture as the Sentry and Mortar. It's slimmer and flatter than v1's chunky round body.
- **16 headings × 3 rotor frames.** Every heading is a real render, not a rotated copy, so the light stays top-left and every heading has four arms. Each heading also has its own matching ground shadow.
- **Explosion:** redone painterly, in the Kingdom Rush / Iron Marines direction. It has a white-hot core and layered fire with soft gradients inside cel bands. The smoke is billowing and volume-lit as whole clouds (lit top-left, shaded bottom-right) with an ink silhouette. There are dirt chunks, and the dust ring is a soft rolling cloud bank instead of a ring of beads. Same 2-tile radius (92 world px) and same ground-point anchor idea. No gore.
- **Scorch:** a painted burn with a shallow crater, a lit lip, radial blast streaks, a soot texture in bands, a few left-over dirt clods, and embers on frame 0. It has no hard outline, so it sits in the ground.
- **Bomb:** rendered with the drone's materials: an off-white nose, a gunmetal body, dark fins, and a cyan arming band that lights on frame 1.

## Files
| file | frames | fps | cell @2x | anchor @2x | what |
|---|---|---|---|---|---|
| `drone-16dir@2x.png` | 16 rows × 3 | 24, loop | 140x100 | (70, 59) body centre | Row = heading (0 = N, clockwise, 22.5° apart). Columns = rotor frames |
| `drone-shadow-16dir@2x.png` | 16 | – | 144x96 | (72, 48) ground point | One soft shadow per heading, same index as the drone rows |
| `bomb-fall@2x.png` | 2 | 8, loop | 56x80 | (28, 42) bomb centre | Falls nose down and rocks ±7°. The cyan band lights on frame 1 |
| `bomb-shadow@2x.png` | 1 | – | 36x18 | (18, 9) | Optional. Grows as the bomb falls |
| `explosion@2x.png` | 8 | 16, no loop | 528x488 | (264, 326) ground point | Flash, fireball, fire breaking into smoke and the dust ring, ring at full radius, then rising smoke |
| `scorch@2x.png` | 2 | – | 294x186 | (147, 93) | Ground decal. Frame 0 has embers and frame 1 is plain |
| `drone-strike-v2.json` | | | | | Every file, cell, anchor (2x and 1x), fps, the heading pick formula, drop offsets per heading, radius, timeline and layers |
| `contact-sheet-drone-strike-v2.png` | | | | | Style check (v1 vs v2 next to the Sentry L2, Kate, a Drifter and Kate's hero-sheet drone), all 16 headings with shadows and drop points, the rotor frames, 1x, bomb and scorch v1 vs v2, and the explosion v2 vs v1 with the radius dashed in |
| `preview-drone-strike-v2.mp4` / `.gif` | | | | | The drone flies a curved path over map-v12 (headings 2→7 with banking), drops the bomb, the explosion and scorch play, and the drone leaves. MP4 1120x760 @2x, GIF 840 px wide, 4.9 s |

## Headings and turning
- **Pick the row from velocity:** `i = round(atan2(dx, -dy) / 22.5°) mod 16`, where (dx, dy) is screen velocity with y down. 0 = N (up), 4 = E (right), 8 = S (toward the camera), 12 = W. In JS: `((Math.round(Math.atan2(dx, -dy) * 180 / Math.PI / 22.5) % 16) + 16) % 16`.
- The anchor is the same body point in every heading, so swapping rows mid-turn doesn't jump. 22.5° steps look smooth at 250 world px/s (see the preview).
- **Tilt:** every heading has a baked 9° nose-down "moving" tilt.
- **Bank:** not baked. For turns, rotate the drone sprite about its anchor by up to ±6°, leaning into the turn. The preview uses `bank = clamp(-yawRate°/s × 0.06, -6, 6)`, eased 25% per frame. Don't rotate the shadow.
- Use the same index for `drone-shadow-16dir`.
- If you'd rather use one round shadow, use any single frame. They're all soft blobs of about the same size.

## Placement
- **Drone:** anchor at `(x, groundY - 70)` world px (altitude 70). Bob it ±1.5 px with `sin(t*9)`.
- **Shadow:** anchor at `(x, groundY)`. It doesn't bob.
- **Drop point:** drone anchor + `dropOffset[i]`, per heading in the JSON. For example: E `(+6.45, +4.05)` 1x, S `(0, +7.75)` 1x, N `(0, +0.35)` 1x. It's the belly under the camera pod.
- **Landing point:** drone ground point + `dropGround[i]`. The bomb falls straight down on screen, so it has the same x as the drop point. To hit a target, release when `ground + dropGround[i]` reaches it.
- **Bomb:** spawn with the anchor at drop point + `spawnOffset` (0, +12.5 1x), so its top sits on the release point. Detonate when the nose (anchor + 9 px 1x) reaches the landing groundY.
- **Explosion and scorch:** anchor on the landing point.

## Size and explosion radius
- The radius is unchanged from v1. Builder's code has Drone Strike at `tiles: 2` with `radius = tiles * Ve` and `Ve = 46`, which gives **92 world px = 184 px @2x**. The dust ring on frames 3–4 sits on that ground ellipse (squash 0.58).
- The source is the Nov 25 bundle snapshot (`ingame/towers-v2/work/index.js`). If the radius changes (the code also has `ct.radius 234 / wideRadius 328`, which looks like the wide upgrade), scale the explosion and scorch by `radiusWorld / 92` about their anchors.
- **Cell size:** the explosion cell grew to 528x488 (v1 was 432x432) so the bigger smoke and dust bank fit without clipping. Use the new anchor (264, 326).
- **Drone size:** about 56 world px across the rotors, the same footprint as v1.

## Draw layers
1. Ground: scorch, drone shadow, bomb shadow.
2. Units (y-sorted).
3. Above units: bomb.
4. FX: explosion.
5. Top: drone.

## Timeline (t = 0 at release)
| t | event |
|---|---|
| −1.9 s | The drone enters on a curved path at 250 world px/s, picking the row from velocity every frame. |
| 0.00 | The landing point is over the target: spawn the bomb. The drone keeps flying and turning, and leaves. |
| 0.00–0.45 | The bomb falls with `y = y0 + (y1 − y0)·t²`. The bomb shadow scales 0.4→1.0. |
| 0.45 | Explosion frame 0 (flash). |
| 0.51 | Frame 1: apply the effect, shake the camera for about 0.15 s, and spawn the scorch (embers). |
| 0.95 | The explosion ends. |
| 1.5 | The scorch switches to the plain frame. |
| zone end − 1 s | The scorch fades to 0 over 1 s. |

## Checks done here
- **Four arms:** checked by eye at 2x zoom for all 16 headings × 3 rotor frames (`work/qc/arms-0.png`, `arms-1.png`). Every one has four arms, four pods and four rotor discs.
- **Clipping:** a script checked every cell of every sheet at 2x and 1x for alpha on the cell edges, and none has any. The explosion got 24 px of extra height after the haze touched the bottom edge.
- **Alpha:** no pixels with alpha 1–3 are left, there's no magenta, and transparent pixels have zeroed RGB. Each 1x frame was downscaled on its own, so nothing bleeds between frames.
- **By eye:** the contact sheet, every explosion frame on grass and at full size, the scorch at 3x, and preview frames covering the fly-in, release, fall, impact, ring, smoke, scorch and exit.

## Style comparison (honest)
- **Drone vs the towers:** this is the closest match. It's the same renderer family, cel bands, ink weight and grade as the Sentry and Mortar, so the shading language matches. It's lighter in value than the towers because the shell is off-white on-model, so it pops a little more than they do. That's useful for a flying unit.
- **Drone vs v1:** v2 is less cartoony. It's slimmer, with proper panel lines, a forward tilt and true 3/4 views. v1 read like a round toy, while v2 reads like a piece of kit.
- **Drone vs Kate and the zombies:** v2 is still cleaner and smoother than the heroes and zombies. Those are painted illustrations with grit, folds and lots of small texture. At game scale (about 56 px across) the difference is small. In the 2x zoom on the contact sheet, the drone looks more "rendered" than painted.
- **Drone vs Kate's hero-sheet drone:** that tiny drone (`kate-drone@2x`, 64x30) is dark grey. v2 follows the off-white cover drone, as asked.
- **Explosion:** it's much closer to Kingdom Rush / Iron Marines than v1's flat fx-v1 burst. The fire gradients and volume-lit smoke work well. It's still more stylised than KR, with rounder puffs and ink outlines on every cloud, so it's a little more cartoony than the gritty zombies. It's in line with the towers and the cel FX, though.
- **Dust ring:** it now reads as a cloud bank. On frames 5–6 the individual puffs show a bit, and frame 7 is a faint ring.
- **Scorch:** it's painted and grounded. Its edge is softer and more airbrushed than the crisp painted texture of the map, which helps it sit under units but makes it a touch blurry up close.

## Weak spots
- **No baked bank:** turning tilt is a ±6° code rotation (see above). If you want true baked banks, I can render ±12° bank variants (16 × 3 × 2 more cells).
- **Faint drone shadow:** it's about 40% alpha at the centre and reads lightly on bright grass. Raise its alpha if needed.
- **1x arms:** at 1x the arms are 2–3 px thick. The drone reads best at @2x, the same as v1.
- **Contact-sheet grass:** the grass is a mirrored tile of map-v12, so the pattern repeats.
- Sprites only, no sound.

## Rebuild (scripts in `work/`)
Run in this order:
1. `build_drone.py` renders 184x156 cells.
2. `crop_drone.py` crops them to 140x100 and the shadows to 144x96, and writes `drone_info.json`. Run it once after each `build_drone.py`.
3. `build_bomb.py`, `build_expl.py`, `build_scorch.py`.
4. `make_json.py`, `contact.py`, `preview.py`.

The shared modules are `r3.py` (painterly ray caster), `drone3d.py` (drone model), `paint.py` (painterly FX painter), `common.py` and `grass.py`.
