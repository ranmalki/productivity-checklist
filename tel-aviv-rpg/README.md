# TLV — a Tel Aviv walkabout (prototype)

A top-down pixel-art RPG prototype set in a Tel Aviv-shaped city block grid.
One self-contained `index.html`, no build step, no image or audio assets — the
city, the sprites and the sounds are all generated in the browser at load.

**Play:** open `tel-aviv-rpg/index.html`, or `/tel-aviv-rpg/` on the deployed site.

```
WASD / arrows   walk            E / space   talk, read, pick up
Shift           jog             M           mute       R   reset save
```

Touch devices get a thumbstick and an **E** button automatically.

## What's in it

- **A generated city.** 128 × 96 tiles: a road grid of 20 × 16-tile blocks, sea and
  beach and promenade down the west edge, blocks subdivided into buildings,
  courtyards, one square with a fountain and one empty lot with a graffiti wall.
  Seeded — the same city every time.
- **Rooftops.** Every building gets a coherent roof: water tanks with solar
  collectors, A/C condensers, laundry lines, potted plants, a satellite dish, a
  stairwell head-house, sometimes a tiled terrace with a table on it.
- **Street level.** Red-and-white curbs, ficus trees on the pavement, zebra
  crossings, cars parked along the kerb, bollards, scooters, café tables,
  recycling bins, shuttered shopfronts with tags on them.
- **A day/night cycle.** ~12 real minutes per in-game day. Dawn, flat noon,
  golden hour, dusk, night — the streetlamps punch warm holes in the dark and the
  windows light up along the facades.
- **A quest chain.** Hummus → a lost cat → a record → the sunset. Four steps,
  five speaking parts, XP and levels, and twelve coins to find on the pavement.
- **Persistence.** Progress saves to `localStorage` under `tlv_rpg_v1`. It does
  not touch the hub app's `ttg_v2` key. `R` wipes it.

## Art direction

Built against a set of pixel-art references of Tel Aviv — Bauhaus blocks from
above, a square with a fountain at golden hour, a café street at night, graffiti
shutters on Allenby, a corner flower shop, the promenade and the sea. The palette
(`P` at the top of the script) is lifted from them: cream and sand walls, warm
grey asphalt, ficus green, the red-white kerb, the specific blue of the water.

## How it fits together

Everything lives in one `<script>` in `index.html`, in reading order:

| Section | What it does |
|---|---|
| config / palette / rng | tile enum, colours, seeded `mulberry32`, `hash2` for per-tile variation |
| world | `baseTile()` lays the road grid; `buildBlock()` subdivides block interiors into buildings, plaza or lot; `dressStreets()` / `dressSeafront()` scatter props |
| tiles | `paintTile()` draws each 16 × 16 tile type procedurally |
| bake | the whole map is painted once into an offscreen 2048 × 1536 canvas; `bakeBuilding()` adds roof clutter and shadows |
| sprites | `sprite()` pre-renders each prop into small canvases; `makeChar()` builds a 4-direction × 4-frame character sheet from a palette |
| cast / quests | NPCs, dialogue trees keyed off `S.flags`, quest list rendering |
| render | camera, y-sorted draw list, lighting pass (tint + `destination-out` lamp pools + additive glow), minimap |
| update | input, tile + prop collision, clock, interaction targeting, save |

## Known rough edges

- Traffic is parked, not moving; NPCs stand still.
- Collision is circle-vs-tile at the feet, so you can clip a tree corner at a jog.
- The block interiors are subdivided by a simple recursive split — a few blocks
  come out sparser than the references.
- No interiors: shops are facades, not rooms.

## Next, if it continues

1. Moving traffic and pedestrians on the road grid (path along the lane tiles).
2. Interiors for the record shop and the falafel stand (a room swap on the door tile).
3. Named streets and a signpost system, so directions in dialogue are followable.
4. Weather: the sea breeze on the trees, a winter rain pass over the same palette.
