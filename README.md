# Skyline Art open data

This repository holds the map data behind Skyline Art posters and cards. Each folder in
[`maps/`](maps) is one printed map.

**The data is under the Open Database License 1.0 (ODbL).** The full text is in
[LICENSE-ODbL-1.0.txt](LICENSE-ODbL-1.0.txt). You may copy, share and adapt it, as long as you credit
it, say what you changed, and keep your version under the same licence.

## What it is

For each map there are three files:

- `scene.geojson`: the OpenStreetMap features of the map's square (buildings, building parts, water,
  coastline, parks, forests, railways, grass and fields, piers, ferry routes, roads, bridges and
  lookout viewpoints) with their OpenStreetMap tags (except phone and email tags, which are left out). Every building also carries the height used
  on the poster and where that height came from.
- `alterations.csv`: the buildings whose height came from Overture Maps because OpenStreetMap had
  none. This is the whole difference between OpenStreetMap's data and this database.
- `manifest.json`: when and from which data release the map was made, and the SHA-256 of the
  other two files, so you can check them.

## How it was made

The data is fetched from OpenStreetMap, and where a building has no height or level count there,
the height is filled in from Overture Maps. [METHOD.md](METHOD.md) describes the exact rules.

The posters themselves (the artwork and the code that draws it) are not part of this repository.

## Credits

See [ATTRIBUTION.md](ATTRIBUTION.md).
