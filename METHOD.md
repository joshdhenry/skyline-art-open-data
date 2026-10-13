# Method

This describes what the tool does, step by step. Names in `code style` are constants in the tool's
source; the value after `=` is what they were when this file was written, and a test checks that
they still match.

## 1. OpenStreetMap data

The tool asks the Overpass API for each layer inside the map's square. It asks for geometry and
tags only (`out geom`), never for edit metadata, so the files carry no user names, edit times or
changeset numbers. The server's data timestamp is recorded as `osm_data_timestamp` in the manifest.

The scene holds these layers: buildings, building parts, water, rivers, coastline, parks,
forests, land cover (grass, fields and similar), railways, piers, ferry routes, roads, bridges,
bridge areas, stadiums and lookout viewpoints. Elements are kept whole, with their OpenStreetMap
tags, and are written in layer order and then by OpenStreetMap id.

The area is the map's square. For a viewpoint poster, which shows land out to the horizon, the
buildings, building parts, water, coastline, parks, forests and land cover are taken over the
wider box the poster shows; the manifest's `bbox` and the map id cover that whole box.

## 2. The height of a building

For each building the tool uses the first of these that applies:

1. The `height` tag, when it parses to a length above zero. `height_source` is `osm`.
2. The `building:levels` tag times `FLOOR_HEIGHT_M` = `3` metres per level, plus the roof height
   when tagged. `height_source` is `osm`.
3. The Overture height described below, only for a building that has no building parts.
   `height_source` is `overture`.
4. An assumed height of `DEFAULT_BUILDING_HEIGHT_M` = `10` metres. `height_source` is `default`.
   This is a guess, not data, and is labelled as such.

Building parts (`building:part`) never take Overture heights. They use rules 1, 2 and 4.

## 3. Where the Overture height comes from

Overture's building features come from a pinned release, `OVERTURE_RELEASE` = `2026-09-23.1`
(the manifest records the release actually used).

**Matching is by identifier, not by shape.** An Overture building lists its `sources`. The tool
looks at sources whose `dataset` is `OVERTURE_OSM_DATASET` = `OpenStreetMap`. Each such source
has a `record_id` like `w123@4`. A `record_id` that matches the pattern `OVERTURE_RECORD_ID_PATTERN` = `/^([wr])(\d+)@\d+$/` names an OpenStreetMap way (`w`) or relation (`r`); the number after `@` is a
version and is ignored. Records for nodes, or that do not match, are skipped. No overlap or
distance test between outlines is made, so there is no threshold.

The height used is Overture's own `height` field, in metres, for features that have one, rounded
to `OVERTURE_HEIGHT_DECIMALS` = `2` decimal places. A height that rounds to zero or less is ignored.

**Several candidates.** If more than one Overture feature cites the same OpenStreetMap element,
the tallest rounded height is used. If heights are equal, the feature with the lowest Overture id
(compared as text) is used. The scene names that feature in `overture_id` and its sources in
`overture_sources`.

**If Overture has nothing,** or the Overture lookup is unavailable or switched off, the building
falls to rule 4 in section 2.

## 4. What is copied from Overture

`overture_id` is Overture's feature `id`. `overture_sources` lists every entry of the feature's
`sources` with the fields `dataset`, `license`, `property` and `record_id`, exactly as Overture
gives them; nothing is reconstructed. `overture_license` is the distinct, non-empty `license`
values of those entries, sorted and joined with `; `, so CC BY 4.0 sources can be found.

## 5. Alterations

`alterations.csv` has one row for each building with `height_source` = `overture`: its
`osm_id`, `overture_id`, `overture_source` (distinct dataset names), `overture_license` and
`height_m`. Those heights, applied to OpenStreetMap's untagged buildings, are the whole
difference between OpenStreetMap's data and this database.

## 6. File size and precision

Coordinates are written as received from OpenStreetMap. If `scene.geojson` would be larger than
`SCENE_SIZE_WARNING_BYTES` = `50000000` bytes, coordinates are rounded to `COORD_DECIMALS` = `5`
decimal places (about 1.1 m) and the manifest says so in `coordinate_decimals`. A scene larger than
`SCENE_SIZE_LIMIT_BYTES` = `100000000` bytes even then is not published. The limits come from
GitHub's file size rules.

## 7. Map ids and repeatability

A map id is the first `MAP_ID_LENGTH` = `8` characters of the SHA-256 of the map's square
(west, south, east, north, each written to `MAP_ID_BOUNDING_BOX_DECIMALS` = `7` decimal places)
and the render date. The same square on the same date gives the same id and the same files, and
the manifest's `scene_sha256` and `alterations_sha256` verify them.

A published map folder is never changed or removed.

## 8. Not included

Terrain and elevation data, the poster artwork and the tool's settings are not part of this
database.
