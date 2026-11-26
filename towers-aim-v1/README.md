# towers-aim-v1: aiming turret art (base + 8-direction head)

**Status:**
- Sentry L1–L3 are done (stage 1).
- Mortar L1–L3 are next.
- The Barricade has no weapon, so it doesn't aim and keeps its towers-v2 / anim-v1 damage-state art.

## Which art this replaces
Builder's towers are the **towers-v2** set (`ingame/towers-v2/`): single north-facing stills plus idle and fire sheets. Two later sets use exactly the same files, cells and pivots:
- `grounding-v1/towers/` is a graded copy with AO and cast shadows.
- `tower-icons-v1/towers-readable/` is that graded copy with the weapons brightened for phone readability.

`anim-v1/` contains hero and zombie animation plus barricade damage states, and no turret art.

This set is finished through the same pipeline: towers-v2 cel finish with a #2A1A10 2 px outline and rim, then the grounding-v1 grade, then the towers-readable weapon boost. So it matches the newest look.

## Files (per tower `<kind>-l<t>`)
| file | what |
|---|---|
| `<kind>-l<t>-base@2x.png` (+ `1x/`) | Static base: sandbags, mount and turntable plate, no weapon. **Same cell, scale and anchor as towers-v2** (sentry 128x208 @2x, anchor (64,202); 1x 64x104, anchor (32,101)). |
| `<kind>-l<t>-head@2x.png` (+ `1x/`) | Head sheet with 8 frames left to right in the order **N, NE, E, SE, S, SW, W, NW**. Cell 192x208 @2x (96x104 at 1x), anchor (96,202). |
| `heads/<kind>-l<t>-head-<DIR>@2x.png` (+ `heads/1x/`) | The same 8 frames as single files. |
| `towers-aim-v1.json` | Paths, anchors, pivots and muzzle offsets (see below). |
| `contact-sheet-sentry.png` | Each level aiming in all 8 directions on its base over map-v12 grass, at @2x and at 1x, next to the current tower. |

**Why the head cell is wider:** the head cell is 192 px wide, not 128. Side-on (E/W) barrels reach about 70 px from the pivot at @2x and would be clipped in a 128 px cell. It shares the base's ground anchor, so drawing both at the tower position lines them up. Nothing moves.

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
```
