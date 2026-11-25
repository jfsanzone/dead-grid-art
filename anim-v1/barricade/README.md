# Barricade: damage states and hit FX (anim-v1)

Everything for the barricade lives in this folder. `barricade.json` is the machine-readable spec.

## Drop-in contract
- Cell is 208x200 @2x (104x100 1x). Pivot / foot point is (104,176) @2x, or (52,88) at 1x, with the anchor at bottom-centre. The scale matches towers-v2 and grounding-v1 `barricade-l<t>@2x.png` exactly.
- Shadows are in `shadows/` and `shadows/1x/`, cell 284x128 @2x. Pivot (104,60) goes on the barricade pivot. They use the same cast-shadow and contact-blob treatment as grounding-v1. Draw the shadow under the sprite and don't wobble it.
- Style: #2A1A10 outline, graded to #221808 like grounding-v1, with cel shading, light from the upper left, and no cyan.

## States (`barricade-l<1-3>-s<0-3>@2x.png`, 1x in `1x/`)
| HP %   | state | look |
|--------|-------|------|
| > 75   | 0 intact  | current grounding-v1 art (identical pixels) |
| > 40   | 1 damaged | cracked planks, a loose or spilled sandbag, splintered edges (grounding-v1 "damaged") |
| > 0    | 2 heavy   | missing planks and gaps, slumped torn sandbags, sand spill. L3 shows the breached hood (grounding-v1 "broken") |
| 0      | 3 broken  | NEW: a low, flattened rubble and splinter pile about half the height of state 0. Clearly passable. Each level keeps its cue: L1 has the car door, L2 the hazard stripes and tire, L3 the hood, wire and orange banner |

Visible height above the pivot @2x is in `levels.*.states.*.visibleHeight2x` (L1 121/125/109/63, L2 126/128/109/61, L3 167/167/147/70).

## FX (normal blending, no additive)
- `fx-barricade-hit@2x.png`: 6 frames at 12 fps (0.5 s), cell 96x96 @2x, pivot (48,48) = impact point. It shows a pale wood-chip flash, 5 splinters arcing out under gravity, sand grains and a dust puff that drifts up. Spawn it at barricade pivot + (0,-38) 1x, with ±14 px x jitter toward the attacker. Play it on every damage tick (rate-limit to about 1 live instance per barricade).
- `fx-barricade-collapse@2x.png`: 8 frames at 14 fps (0.57 s), cell 256x232 @2x, pivot (128,190) = barricade pivot. One-shot on 2→3. Draw it above the barricade. **Swap the sprite from state 2 to state 3 on frame 2**, when the dust cloud covers the wall. Add an optional 2 px 1x screen shake.

## Hit wobble (code)
- Offset the sprite only, on the x axis. Amplitude is 2 px 1x (4 px @2x), or 3 px 1x in state 2.
- Formula: `x = A*sin(2π*2.5*t/0.16)*(1-t/0.16)^2` for t in 0..0.16 s, plus a 1 px downward squash for the first 0.05 s.
- No wobble in state 3. If hits arrive within 0.06 s, top up the amplitude instead of restarting.

## Contact / QC
- `contact-sheet-phone-v1.png`: all 4 states × 3 levels on farm-map-v12 at iPhone landscape 844x390 pt, plus the road strip at real device pixels (3x), with zombies and hit frames.
- `contact-states@2x.png`, `contact-collapse-sequence@2x.png`, `contact-hit-sequence@2x.png`.
- Sources and scripts are in `work/`: gen/barricade-rubble.jpg, states.py, fx.py, contact.py. These import the towers-v2 / grounding-v1 helpers read-only.
