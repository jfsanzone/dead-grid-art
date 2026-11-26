# drone-strike-v1: Kate's Drone Strike (replaces the mud-drop visuals)

A small drone crosses the map, drops a bomb on the target spot, and the bomb explodes into an orange fireball and smoke, leaving a scorch mark. The style matches towers-aim-v1, grounding-v1 and fx-v1: cel-shaded, #2A1A10 outlines, light from the upper left, cyan only on powered gear (the drone's visor and motor lights), no gore. Everything is a transparent PNG at @2x, with 1x copies (= world px) in `1x/`. Sheets run left to right.

## The drone (on-model)
It's Kate's quadcopter from the cover (cover-v4b) and hero sheet v2: a warm off-white armoured shell, a cyan visor slit, a camera pod under the nose, four dark gunmetal arms and motor pods with small cyan lights. It's drawn from the map's 3/4 camera (the same angle as the towers), not straight top-down, because a pure top-down drone would be the only thing on the map seen from directly above.

## Files
| file | frames | fps | cell @2x | anchor @2x | what |
|---|---|---|---|---|---|
| `drone-fly-east@2x.png` | 3 | 24, loop | 120x100 | (58.2, 55.8) body centre | Drone flying east, with spinning rotor-blur discs |
| `drone-shadow@2x.png` | 1 | – | 96x40 | (48, 20) | Soft ground shadow for the drone |
| `bomb-fall@2x.png` | 2 | 8, loop | 44x60 | (22, 31) bomb centre | Falling finned bomb with a hazard band. It rocks ±6°, and the amber arming light blinks on frame 1 |
| `bomb-shadow@2x.png` | 1 | – | 36x18 | (18, 9) | Optional. Grows as the bomb falls |
| `explosion@2x.png` | 8 | 16, no loop | 432x432 | (216, 300) ground point | Flash, fireball and shockwave, then a dust ring at the full ability radius, then rising smoke |
| `scorch@2x.png` | 2 | – | 294x186 | (147, 93) | Ground decal. Frame 0 has embers and frame 1 is plain |
| `drone-strike-v1.json` | | | | | Every file, cell, anchor (2x and 1x), frame count, fps, the drop offset, the explosion radius, the timeline and the layers |
| `contact-sheet-drone-strike.png` | | | | | Every sprite over map-v12 grass at @2x with a 1x row. The explosion has the ability radius dashed in |
| `preview-drone-strike.mp4` / `.gif` | | | | | The whole sequence on map-v12 at @2x with three zombies for scale (MP4 1120x660; GIF 840 px wide), 4.4 s |

## Size and explosion radius
- The radius comes from Builder's code: Drone Strike is `tiles: 2` and the radius is `tiles * Ve`, with `Ve = 46` world px per tile. That gives **92 world px = 184 px @2x**. (It's the same 2-tile radius as hero sheet v2.)
- In the explosion, the dust ring on frames 3–4 sits on that radius, drawn as a ground ellipse (184 x 107 @2x, squash 0.58, the same as the tower turntables). The fireball is about 0.6 of the radius.
- **If the radius changes** (for example, the code also has `ct.radius 234 / wideRadius 328`, a 1.4x ratio that looks like the wide upgrade), scale the explosion and scorch by `radiusWorld / 92` about their anchors.
- Source checked: the Builder bundle snapshot `ingame/towers-v2/work/index.js` (dated Nov 25). If the live game has changed the tile size, use the new `Ve`.
- The drone is about 60 world px across its rotors, a touch bigger than Kate's shoulder drone (`kate-drone@2x` 64 px @2x), because it's flying high and needs to read on the move.

## Draw layers
1. **Ground layer** (under units): scorch, then the drone shadow and the bomb shadow.
2. **Units** (y-sorted as usual).
3. **Above units:** the bomb.
4. **FX layer:** the explosion.
5. **Top:** the drone. It flies above everything, including the explosion smoke.

## Placement
- **Drone:** draw the anchor at `(x, groundY - 70)` world px. The suggested altitude is 70. Bob it ±1.5 px with `sin(t*9)`, and don't move the shadow when it bobs.
- **Drone shadow:** centre it at `(x + dropOffset.x, groundY)`.
- **Drop point:** `dropOffset1x = (-2.1, +12.9)` from the drone anchor (`dropOffset2x = (-4.2, +25.7)`). That's the belly under the camera pod. If you mirror the drone, negate x.
- **Bomb:** spawn it with the anchor at drop point + `(0, +10.5)` 1x (`spawnOffset`), so the bomb's top sits on the release point. It falls straight down to the target. Detonate when the nose (anchor + 8.5 px 1x) reaches `groundY`.
- **Explosion and scorch:** anchor on the target ground point.

## Facing
The art faces **east**. For a westbound pass, mirror it horizontally and negate `dropOffset.x`. For other headings, either:
- (a) keep the sprite upright and just mirror by the sign of dx. This is the cleanest option, because the 3/4 drone keeps its lighting.
- (b) rotate about the anchor to the flight angle. It reads fine up to about ±30°, but beyond that the lighting looks wrong.

For a fully top-down build, say so and I can do a straight-down version.

## Suggested timeline (t = 0 at release)
| t | event |
|---|---|
| -1.1 s | The drone enters at 300 world px/s toward the target (from Kate or off-screen), with the rotors at 24 fps. |
| 0.00 | The drop point is over the target: spawn the bomb, and the drone keeps flying and exits. Play a short whistle. |
| 0.00–0.45 | The bomb falls with `y = y0 + (y1 - y0)·t²` and blinks at 8 fps. The bomb shadow at the target scales 0.4→1.0. |
| 0.45 | Impact: explosion frame 0 (flash). |
| 0.51 | Frame 1: **apply the effect** (damage or slow), shake the camera about 0.15 s (±3 px world, decaying), play the boom, and spawn the scorch (embers frame) on the ground layer. |
| 0.51–0.95 | Explosion frames 2–7. The dust ring on frames 3–4 marks the radius. Then despawn. |
| 1.5 | The scorch switches to the plain frame. |
| zone end − 1 s → zone end | The scorch fades to 0. With the 5 s zone that's 4.5–5.5 s after impact, or 7.5 s with the long upgrade. If the reworked ability does instant damage instead of a zone, fade the scorch over 3–4 s. |

## Checks done here
- Checked by eye: the contact sheet, every explosion frame on grass, the drone at 4x zoom, and the preview frames (fly-in, mid-fall, impact, smoke, scorch fade).
- No frame has opaque pixels on a cell edge, at 2x or 1x. Frames were downscaled one at a time so nothing bleeds between them.
- No magenta fringe on the drone (keyed from magenta, despilled, and checked by pixel count). Backgrounds are fully transparent, and stray alpha below 4 is cleared.

## Weak spots
- The drone came from the image generator, and two of the three takes had five arms. The final one is an edit back to four arms. It's on-model in colours and parts, but it's a bit chunkier and rounder than the sleek cover drone. The rotor discs are drawn procedurally (translucent discs with 2-blade smears).
- At 1x the drone is about 60 px wide, so the cyan visor reads but the arms get thin. It reads best at @2x.
- The explosion is the flat fx-v1 cel style (it uses the same toolkit as `fx-hit-mortar`), not painterly. That's deliberate, so it matches the other hits, but it's simpler than the drone and towers.
- The dust ring on frames 5–6 thins into a ring of beads as it fades. It's only on screen for about 0.12 s.
- The 92 px radius is from the Nov 25 bundle snapshot. If Builder has changed `Ve` or the tile count, scale as above.
- Sprites only, no sound.

## Rebuild
Scripts are in `work/`: `build_drone.py` (key, finish and rotor frames from `work/gen/drone-gen-c.jpg`), `build_fx.py` (bomb, explosion, scorch, shadows), `make_json.py`, `contact.py` and `preview.py`.
