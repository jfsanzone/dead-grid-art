# anim-v1: full-motion character animation

Real walk cycles, idles and attacks for the 5 heroes and 4 zombies. They drop in like grounding-v1: same art, outline, foot point and shadow treatment, with more frames. `anim-v1.json` is the machine-readable spec, and Builder should read it rather than this file.

## Files
| file | what |
|---|---|
| `<name>-walk@2x.png`, `1x/<name>-walk.png` | 8 frames × 8 rows. All 9 characters |
| `<hero>-idle@2x.png` | 8 frames × 8 rows. Heroes only (jess, kate, coupler, breaker, relay) |
| `<hero>-attack@2x.png` | 8 frames × 8 rows. Hit on frame 3 |
| `<zombie>-attack@2x.png` | 8 frames × 8 rows. Claws at a barricade in front of it. Hits on frames 2 and 6 (left swipe, then right swipe) |
| `shadows/<name>-<anim>-shadow@2x.png`, `shadows/1x/` | per-frame cast shadow and contact blob, same layout. Put the shadow pivot on the foot point and draw it under all sprites |
| `anim-v1.json` | cells, pivots, fps, the speed mapping, hit frames, zombie reach/standoff, and an empty `barricade` section to merge into |
| `previews/<name>-<anim>-8dir.gif` | 8-direction treadmill at the JSON fps on a v12 crop |
| `previews/map-path-test-phone-844x476.mp4`, `.gif` | every character walking the v12 road at phone scale, at matched speed |
| `previews/map-path-test-device-crop.mp4` | the same, 1:1 @2x device pixels |
| `previews/zombie-barricade-attack-test.mp4` | the 4 zombies clawing barricade-l1 (state 1) with `fx-barricade-hit` on the hit frames |
| `barricade/` | barricade damage states and FX (a separate set with its own README) |
| `work/` | rig and scripts (sources, not shipped) |

## Layout
- Rows are always `down, up, left, right, downleft, downright, upleft, upright`, and columns are frames 0..7. Every animation loops.
- Pick the row 8-way from the on-screen movement (or facing) vector in 45° sectors.
- Right-facing rows are mirrors of the left ones, so held items swap hands (Relay's radio, Breaker's tablet).
- **Cells differ per sheet.** Some walk cells grew so raised feet don't clip. Always read `cell` and `pivot` from the JSON for each sheet; don't reuse the grounding-v1 values.
- The pivot is the foot point, anchored bottom-centre. It is the same contact point as grounding-v1, so swapping sheets never pops the character.
- Scale: @2x is 2 px per world px for everything, and 1x is 1:1. The zombies were rescaled from the ~4 px/world grounding PNGs to the grounding height rule, so draw them at cell size like the heroes.
- Filtering is the same as grounding-v1: bilinear, no mipmaps, premultiplied alpha.

## Speed ↔ fps (no foot sliding)
Each walk cycle (8 frames) covers a fixed ground distance, `worldPxPerCycle`. It is the same in all 8 directions because the stride was normalised on screen. Feet stay locked to the ground only when:

`fps = moveSpeed(world px/s) × 8 / worldPxPerCycle`

| char | px/cycle | default fps | speed at default |
|---|---|---|---|
| breaker / relay | 30.8 | 12 | 46.2 |
| jess | 29.7 | 12 | 44.6 |
| kate | 28.7 | 12 | 43.1 |
| coupler | 20.5 | 12 | 30.8 |
| drifter | 17.9 | 10 | 22.4 |
| sprinter | 36.4 | 14 | 63.8 |
| howler | 17.2 | 10 | 21.5 |
| bulwark | 21.3 | 9 | 24.0 |

- Drive the walk fps from the unit's actual speed every frame (slows, haste), clamped to 0.75–1.4× default.
- Below about 25% of default speed, switch heroes to idle.
- Advance the frame by accumulated time. Don't reset the cycle on direction changes, so the phase carries over.

## Hit frames
- **Zombie attack:** `hitFrames: [2, 6]`. When the frame index *becomes* 2 or 6: apply barricade damage, spawn `barricade/fx-barricade-hit` at barricade pivot + (0,−38) 1x with x jitter toward the attacker, then wobble and swap state per `barricade/barricade.json`.
  - Stop the zombie at the standoff distance (`standoffWorldPx`, about 64–71 world px from the barricade pivot on side rows for barricade l1) so the claws land on the planks.
  - For vertical and diagonal approaches, stop at a foot-point gap of about 34–40 world px.
- **Hero attack:** `hitFrames: [3]`. Frames 0–2 are anticipation and 4–7 recover. Spawn the projectile or melee damage on frame 3. The muzzle/strike flash is baked into frame 3.

## QC done
- **Foot drift**, measured as planted heel/toe in world space across the stance (@2x px):
  - heroes: down/up 0, sides and diagonals ≤1.4
  - drifter and howler ≤0.55
  - sprinter ≤1.65
  - bulwark ≤1.4
  - That's under 1 world px everywhere.
- Every frame was checked against its cell for clipping: none.
- The previews were reviewed at the JSON fps.

## Known limits
- Hero idle and attack are a spine/torso warp of the grounding-v1 attack pose (wind-up, strike, recoil), not a full limb re-animation. Breaker's down-row attack is the kneel/slam pose.
- Zombie walk arms ride the torso. The arms are rigged only for the claw.
- Source-art inconsistencies are inherited from grounding-v1:
  - bulwark downleft/downright is the armored-brute design
  - howler downleft has greener skin
  - the sprinter's source poses are mid-run
- Minor mesh tearing on the drifter's torn jeans in some frames.
