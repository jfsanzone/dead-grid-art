# Map v12 walk-behind props (for Builder)

`farm-map-v12` split into a **base map with the tall props removed** plus **28 prop sprites**, so units can walk behind trees, the silo, the farmhouse, vehicles, barrels, crates and fences.

## Files
- `farm-map-v12-base.jpg` (1280x720) and `farm-map-v12-base@2x.jpg` (2560x1440): v12 with the props painted out.
- `props/1x/<name>.png` and `props/<name>@2x.png`: transparent prop sprites. The edges are anti-aliased, and wherever the cut edge wasn't already dark ink they get a soft dark-brown (#221808) outline to match the map's line work.
- `props.json`: position, size and sortY for every prop.
- `verify.json`, `verify/recomposite@2x.png`, `verify/diff_x4.png`: check that base plus props reproduces the original v12.
- `test/walk-behind-test@2x.png`: heroes and drifters walking behind and in front of the silo, tree-pond and farmhouse. `test/walk-behind-full-1x.png` is the whole map with the same units.
- `work/`: the scripts (cut.py makes the masks, build.py makes the base, props, JSON and verify, testcomp.py makes the test). Rebuild with `PYTHONPATH=/workspace/pylib-ocvc python3 build.py all` from inside `work/`.

## Draw order (every frame)
1. Base map (`farm-map-v12-base`).
2. Ground layer: decals, path/blood FX, tower bases.
3. Shadows: all unit and tower shadows (the grounding-v1 shadow sheets).
4. **Units and props together, sorted by sortY, lowest first.** A unit's sortY is its foot y. A prop's sortY is the `sortY` in props.json. On a tie, draw the prop first.
5. FX: projectiles, muzzle flashes, hit sparks, UI.

Props never move, so you can pre-sort them once and merge-insert the units each frame.

## props.json
```json
{"name": "tree-pond", "kind": "tree", "x": 760, "y": 49, "w": 76, "h": 71, "sortY": 114, "anchor": {"x": 38.0, "y": 65}, "file": "props/1x/tree-pond.png", "file2x": "props/tree-pond@2x.png"}
```
- All values are in **1x map pixels**. For the @2x art, multiply by 2.
- `x`,`y` = top-left where the sprite goes on the map. Drawing it there reproduces v12 exactly.
- `sortY` = the prop's ground/foot line (map y).
- `anchor` = the same foot point relative to the sprite's top-left (x = centre, y = sortY - y), if you'd rather position by pivot.

| prop | kind | x,y | w x h | sortY |
|---|---|---|---|---|
| tree-pond | tree | 760,49 | 76x71 | 114 |
| tree-tr | tree | 1174,0 | 106x122 | 116 |
| tree-tl | tree | 0,0 | 102x142 | 136 |
| crossing-signal | signal | 1019,59 | 43x83 | 140 |
| haybale-1 | hay | 77,95 | 60x58 | 150 |
| fence-farmhouse | fence | 1058,131 | 36x53 | 178 |
| silo | building | 164,57 | 54x123 | 180 |
| bushes-left | bush | 0,133 | 78x122 | 250 |
| haybale-2 | hay | 133,218 | 34x32 | 250 |
| farmhouse-crates | crate | 1034,182 | 49x73 | 252 |
| bushes-right | bush | 1220,145 | 60x150 | 290 |
| feed-barrel | barrel | 223,228 | 40x66 | 292 |
| fence-left-top | fence | 3,267 | 79x39 | 300 |
| farmhouse | building | 1047,109 | 170x199 | 305 |
| truck | vehicle | 83,268 | 126x81 | 344 |
| wreck-cart | vehicle | 1093,388 | 76x54 | 444 |
| crate-3 | crate | 1143,451 | 43x46 | 493 |
| crate-1 | crate | 421,511 | 31x25 | 540 |
| barrel-right | barrel | 1038,517 | 38x72 | 578 |
| fence-right | fence | 1194,373 | 86x219 | 585 |
| crate-2 | crate | 399,544 | 38x37 | 586 |
| tractor | vehicle | 57,491 | 125x106 | 592 |
| drum-left | barrel | 502,550 | 34x39 | 596 |
| fence-left | fence | 6,451 | 46x157 | 612 |
| fence-bottom-l | fence | 118,666 | 529x40 | 698 |
| fence-bottom-r | fence | 643,666 | 464x42 | 698 |
| tree-bl | tree | 0,553 | 117x158 | 705 |
| tree-br | tree | 1098,518 | 182x193 | 705 |

## Verification
Recomposite (base + all props at their positions) compared with the original v12 @2x:
- PSNR 34.61 dB.
- Inside the props: mean abs diff 0.04 (essentially identical).
- Outside the props and their 6px edge band: max diff 0 (untouched).
- Edge band (6px at 2x around each cut): mean 13.81, p99 80. This is the AA/outline seam, and it only shows as a thin edge.

## Known limitations / weak spots
- **The base map under the props isn't meant to be seen.** The inpaint (class-based texture fill: grass, dirt, sand, water, plus a periodic copy for the rail sleepers) holds up at gameplay zoom, but if you hide a prop you'll see:
  - the farmhouse footprint: a patchy dirt/grass fill, with the porch deck and vine lattice left floating (they're flat or behind the house and stay in the base);
  - tree-tr, tree-tl, bushes-left and bushes-right: soft, slightly blurry grass;
  - tree-br: the creek under the canopy is filled as grass, so the creek is narrower than in v12;
  - the silo spot: soft grass.
- **One sortY per prop.** This works well for upright props. For long or diagonal ones (the truck, the bottom fence runs, wreck-cart), a unit walking along the long side can sort wrong at the ends. If that shows up in play, split the prop or give it a sloped sort line.
- Some props carry a little painted contact shadow (tractor, truck, crate bottoms), so a unit standing right at the base line draws under that small strip. Hardly noticeable.
- The top-left and top-right tree sprites include a few rail-sleeper pixels behind their branches. These match the base exactly and the rails aren't walkable, so it doesn't matter.
- Kept in the base as flat ground detail: reeds, rocks, tyres, the lily pads, the boardwalk/bridge (no tall posts), the farmhouse porch deck (flat with steps, no porch roof) and the trellis.
- The bottom fence runs (fence-bottom-l/r) sit just below the playable area. They're included anyway so units clipping the edge go behind them.
