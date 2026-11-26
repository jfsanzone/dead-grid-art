# towers-aim-v1: aiming turret art (base + 8-direction head)

**Status:**
- Sentry L1–L3 are done (stage 1, handed to Builder, unchanged since).
- Mortar L1–L3 are done (stage 2).
- That covers every aiming tower in towers-v2. The only other tower there is the Barricade, which has no weapon, so it doesn't aim and keeps its towers-v2 / anim-v1 damage-state art.

## Which art this replaces
Builder's towers are the **towers-v2** set (`ingame/towers-v2/`): single north-facing stills plus idle and fire sheets. Two later sets use exactly the same files, cells and pivots:
- `grounding-v1/towers/` is a graded copy with AO and cast shadows.
- `tower-icons-v1/towers-readable/` is that graded copy with the weapons brightened for phone readability.

`anim-v1/` contains hero and zombie animation plus barricade damage states, and no turret art.

This set is finished through the same pipeline: towers-v2 cel finish with a #2A1A10 2 px outline and rim, then the grounding-v1 grade, then the towers-readable weapon boost (sentry heads only; the mortar heads skip the boost, see below). So it matches the newest look.

## Files (per tower `<kind>-l<t>`)
| file | what |
|---|---|
| `<kind>-l<t>-base@2x.png` (+ `1x/`) | Static base: sandbags, mount and turntable plate, no weapon. **Same cell, scale and anchor as towers-v2** (sentry 128x208 @2x, anchor (64,202); mortar 160x208 @2x, anchor (80,202); 1x is half). |
| `<kind>-l<t>-head@2x.png` (+ `1x/`) | Head sheet with 8 frames left to right in the order **N, NE, E, SE, S, SW, W, NW**. Sentry cell 192x208 @2x, anchor (96,202). Mortar cell 160x208 @2x, anchor (80,202), the same as its base. 1x is half. |
| `heads/<kind>-l<t>-head-<DIR>@2x.png` (+ `heads/1x/`) | The same 8 frames as single files. |
| `towers-aim-v1.json` | Paths, anchors, pivots and muzzle offsets (see below). |
| `contact-sheet-sentry.png`, `contact-sheet-mortar.png` | Each level aiming in all 8 directions on its base over map-v12 grass, at @2x and at 1x, next to the current tower. |

**Why the sentry head cell is wider:** the head cell is 192 px wide, not 128. Side-on (E/W) barrels reach about 70 px from the pivot at @2x and would be clipped in a 128 px cell. It shares the base's ground anchor, so drawing both at the tower position lines them up. Nothing moves.

## Drawing
1. Draw the base at the tower's world position with its anchor.
2. Draw the head frame at the **same world position** with the head's anchor. All 8 frames are pre-registered, so no per-direction offsets are needed.
3. Pick the frame with `i = round(angleDeg/45) mod 8`. The angle is measured clockwise from screen-up: `angleDeg = atan2(dx, -dy)` with screen y pointing down. 0 = N (up-screen), 2 = E, 4 = S (toward the camera).
4. Don't rotate the sprite any more. If you want smoother aim between the 45° steps, you can add a small residual rotation of up to ±22.5° about the head **pivot**. That keeps the base still, and the lighting error is small.
5. **Muzzle flash / projectile spawn:** use head pivot + `muzzle2x[dir]` (or `muzzle1x`). Twin guns list two muzzles, so alternate between them. In side views the twins overlap, so both points are nearly equal.
6. **Recoil (suggested):** offset the head 2 px @2x / 1 px 1x opposite the aim direction for about 60 ms. The fx-v1 muzzle flashes work at the muzzle points.
7. **Shadow:** keep the grounding-v1 tower shadow under the base (`grounding-v1/shadows/towers/<kind>-l<t>-shadow@2x.png`, same anchor).

## JSON format (`towers-aim-v1.json`)
```
towers.<kind>.l<t>:
  base: { file, file1x, cell2x, anchor2x, cell1x, anchor1x }
  head: { sheet, sheet1x, frames: 8, order: [N..NW], cell2x, anchor2x, cell1x, anchor1x,
          pivot2x, pivot1x,              // head rotation axis (turntable hub) inside the head cell
          files{dir}, files1x{dir},      // single-frame files
          muzzle2x{dir}: [[dx,dy],...],  // muzzle tip offsets from pivot, px, screen y down
          muzzle1x{dir}: [[dx,dy],...] }
  mountInBaseCell2x: [x,y]               // the same hub point in base-cell coords
  clipped{dir}: false                    // QC: no opaque pixels on any cell edge
mortar only, inside head:
  launch2x{dir}: [[dx,dy],...]           // shell spawn = centre of the tube mouth, offset from pivot (L3: two tubes)
  launch1x{dir}: [[dx,dy],...]
  tubeElevationDeg: 60, lob: true
  (muzzle2x/muzzle1x hold the same values as launch*, so muzzle-reading code still works)
```

## Mortar (stage 2)
**What it is.** A short, fat tube on a yoke (L1 is on a two-wheeled wooden cart with a rope lashing; L2 is one big tube with an iron band; L3 is twin tubes with rope and iron bands on a steel cradle). Each level sits on a turntable plate in its towers-v2 base: L1 has crates and a shell bucket, L2 has sandbags, a stone block and an ammo crate, and L3 has the hazard-striped ring, ladder, banner, amber beacon and cyan lamp. The tube is fixed at **60° elevation** and turns with the head in 8 directions.

**How it was made.** The image generator couldn't turn a mortar consistently (it kept drawing the same pose), so each head is a small 3D model, rendered in cel style with the towers-v2 palette, #2A1A10 ink and the same 2 px outline and grounding grade. That keeps all 8 frames exactly on-model, and the launch points are exact (projected from the model's mouth centre, not guessed from pixels). The towers-readable "weapon boost" is **not** applied to the mortar heads. That boost caused the silvery cast. Instead the iron is a warm, rust-flecked gunmetal that sits with the brown bases.

**Launch point.** Spawn the shell at head pivot + `launch2x[dir]` (or `launch1x`). That is the centre of the tube mouth. For L3, alternate between the two tubes. For N, the mouth faces away, so the point is at the top of the tube. For S, the mouth faces the camera, so the point is in the middle of the visible bore.

**Should the tube also show elevation? Recommendation: no. Keep the tube fixed at 60° and do the lob in code.**
- At phone size the tube is about 20–28 px wide @2x. A 45°→75° change moves the mouth only about 6–10 px, and it barely reads. It would also triple the frames (24 per level instead of 8).
- Players read range from the shell arc and the impact, not from the tube angle. Spawn at the launch point and fly a parabola to the target in screen space: x and y lerp linearly, with an extra upward offset of `4·h·t·(1−t)`, where apex height `h` is about 0.35 × distance (clamp 40–140 px @2x) and flight time is about 0.6–0.9 s, scaled with distance. Scale the shell sprite up by about 15% at the apex for depth. Put the fx-v1 muzzle puff at the launch point and drop the shadow dot on the ground line.
- **Recoil (suggested):** push the head 2 px @2x straight down (not opposite the aim) for about 80 ms. A mortar kicks into its bed.
- If you do want elevation later, it's cheap: the heads come from a 3D model (`work/mortar3d.py`, `TILT`), so a 45° and a 75° set can be re-rendered with matching launch points without redrawing anything.

