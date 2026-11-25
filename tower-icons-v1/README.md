# Dead Freight tower-icons-v1: build-ring and tray icons + readable placed towers

Fixes Jason's iPhone report (7:34 AM screenshot): the build-ring icons were dark grey guns on dark brown discs and nearly invisible. Nothing has been sent to Builder.

## Icons (`128/`, `256/`, `tower-icons-v1.json`)
Transparent square PNGs, with the content centred in the square.
- **Names:** `<kind>-<style>-<state>.png`. Kind is `sentry`, `mortar` or `barricade`. Style is `ring` (round build-ring button) or `glyph` (no disc, for the tray tower buttons). State is `normal`, `cantafford` or `locked`.
- **Art:** a 3/4 view of the weapon only, painted from the towers-v2 / grounding-v1 L2 designs: Sentry is the twin MG head with shield and ammo belt, Mortar is the riveted tube on its bipod, Barricade is the sandbag, steel and hazard-plank wall. The weapon fills about 75-80% of the frame.
- **Contrast stack:** weapon art with its midtones lifted (gamma 0.74, contrast 1.18), then a #2A1A10 outline, then a worn-cream backplate (#EEE2C6) hugging the silhouette, then a thin #2A1A10 edge.
- **Ring:** a warm mid-tan disc (about #92806A to #685842, lit from the upper left, far lighter than the dark brown UI) inside a riveted rust rim (#B0602E) with a #2A1A10 edge.
- **States:**
  - `normal`: full colour, rust rim, cream backplate.
  - `cantafford`: weapon desaturated to 25% and darkened 18%, a dull red rim (#963428), a greyed backplate, and a red "no salvage" badge at the bottom right. Builder should also turn the cost tag red.
  - `locked`: weapon greyscale at 52% brightness (detail kept, so it still reads as that weapon), a dark steel disc and rim, a dim backplate, and a brass padlock at the bottom right. Drop the separate black lock bubble the game draws now.
- **No cyan:** the icon weapons have no powered gear, so none is used.
- **Which size:** 44-56 CSS px at 3x DPR is 132-168 device px, so use `256/` (or `128/` on 1x/2x screens). At 96 CSS px, use `256/`.
- **JSON:** each icon's files and its opaque bbox at 256.

## Placed towers with readable weapons (`towers-readable/`)
A drop-in replacement for `grounding-v1/towers/`: same filenames, cells, pivots and frame counts, 2x plus `1x/`, and `towers-grounding.json` copied unchanged. Design and size are unchanged; only the tone changed on the weapon half (barrels, gun heads, mortar tubes):
- Pixels above each tower's turret split line get their value lifted (gamma 0.6), a 28% shift toward neutral steel, local contrast (unsharp, 2.5px), and a thin light edge on the upper-left of the silhouette.
- The base below the split is darkened 12%, so the gun separates from the sandbags and crates.
- The dark outlines, cyan lamps, banners and saturated sandbags/fx are masked out, so they keep their colour. Idle and fire frames get the same pass, so the animation stays consistent.
- Barricades are copied through unchanged, since they have no weapon.

## Test sheet: `test-sheet-v1.png` (3000x5497)
1. Before vs after: the live rings cropped from Jason's screenshot, next to the new rings at the same ~52 CSS px.
2. Ring icons on the dark UI panel (cropped from the screenshot) at 44 / 56 / 96 CSS px, rendered at iPhone 3x device px, plus a 1x worst-case row.
3. The same over the v12 map.
4. Tray glyphs on the dark UI at 44 / 56 / 96.
5. Placed towers, map fit to 844x390 landscape: before (grounding-v1) vs after (towers-readable), at 1x CSS (crop shown 3x nearest), the whole map at 844x390, and a 3x device-px crop.

## Scripts (`work/`)
- `build_icons.py`: builds the icons from `keyed/<kind>.png`, the weapon art keyed from `gen/weapons.jpg`.
- `boost_towers.py`: builds `towers-readable/` from `grounding-v1/towers`.
- `test_sheet.py`: builds the test sheet.

## Weak spots
- At 44 px on a 1x screen (worst case), the Sentry and Barricade glyphs read by silhouette and colour, but the fine detail (ammo belt, hazard stripes) blurs. On the iPhone at 3x they're crisp.
- The can't-afford badge is about 10 CSS px at 44, readable as a red dot with a slash. The red rim does most of the work.
- The tower weapon boost is clear at 3x device px. At 1x CSS phone scale, the barrels now read as light steel, but towers are still only about 30 px tall. If they still feel small, the next lever is the code (draw towers at the towers-v2 1x cell size) rather than more tone.
- The icon weapons were a fresh render matched to the L2 designs. They're slightly cleaner and simpler than the in-world sprites, on purpose, for small sizes.
