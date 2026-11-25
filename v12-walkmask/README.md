# Map v12 walkability mask (for Builder)

Heroes can currently walk on water. These files say where heroes can and can't go on `farm-map-v12`.

## Files
- `walkmask.png` (1280x720) and `walkmask@2x.png` (2560x1440): **white = walkable, black = blocked**. Pixel-accurate.
- `walkgrid.json`: an 80x45 grid of 16x16 cells, in 1x map pixels. `grid[row][col]`, row-major from the top-left. `1` = blocked, `0` = walkable. A cell is blocked when more than 40% of it is blocked in walkmask.png. Fields: `cellSize`, `width`, `height`, `blockedThreshold`, `grid`.
- `overlay.png`: the blocked area tinted red over the map. `overlay-grid.png` adds the blocked grid cells as yellow squares.
- `work/`: the scripts (`water.py` → `water2.py` → `build.py`). Rerun them in that order to rebuild.

## What's blocked
- **Water:** the two creeks, the right-hand creek that runs under the bottom-right tree, the top-left pond and the bottom-right pond. Water is detected by colour, then:
  - lily pads, rocks and floating debris inside the water are filled in (blocked);
  - reed clumps standing in the water are included, two of them added by hand in the middle creek;
  - speckle is removed (tiny blocked islands dropped, tiny walkable holes filled).
- **Solid footprints**, taken from the map-v12-props sprites:
  - the farmhouse body (house walls; the stacked crates on its left are blocked too);
  - the silo base;
  - the five tree trunks;
  - the truck, tractor and wreck-cart;
  - all crates;
  - the feed barrel, barrel-right and drum-left (their bases);
  - both hay bales;
  - the crossing-signal post.
- **Rail line:** the track and its gravel bed along the top edge (map y < 54), which is off the play area.

## What stays walkable
- The zombie road (the whole sand road).
- The plank boardwalk / bridge that comes down from the top edge (x 932–998) and over the rails, plus its spur to the farmhouse.
- The farmhouse porch deck and the stone yard.
- Grass, dirt and the crop field.
- Fences and bushes: these weren't in the block list. See the uncertainties below.

## How to use it
- **Hero movement:** clamp it to walkable space. Test the next position against `walkmask.png`, or against `walkgrid.json` for a cheaper cell test. If it's blocked, slide along the free axis rather than stopping dead.
- **Pathfinding (A*):** run it on `walkgrid.json` with 8-neighbour moves. Don't let paths cut a diagonal corner between two blocked cells.
- **Tap on a blocked spot:** snap the target to the nearest walkable cell (BFS outward from the tapped cell), then path there.
- **Zombies:** they stay on their road path (spline/waypoints) and ignore this mask. It's for heroes and any free-moving units.
- **Spawns, rally points and drops:** if one lands on a blocked cell, snap it the same way.

## Uncertain / judgment calls
- Edge reeds: reeds on the bank stay walkable. Only reeds clearly in the water are blocked. The creek edges are about ±4px fuzzy.
- Fences are walkable because the brief didn't list them. If heroes shouldn't cross the fence lines (left side, right side, bottom runs, the little farmhouse fence), they can be added as thin blocked lines.
- The grid is coarse. Small blockers that cover less than 40% of a 16px cell vanish from walkgrid.json: the tree-pond trunk, the signal post, some barrel bases. Use walkmask.png when you need exact collision.
- The farmhouse footprint is the house sprite's lower 100px (map y 205–305). That's an approximation of the walls in this 3/4 view.
