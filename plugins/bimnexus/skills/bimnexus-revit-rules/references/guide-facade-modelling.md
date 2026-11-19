# Guide: Facade modelling (core + finish-skin walls, windows, balconies, railings, grouped bay modules)

> BIMNEXUS note: written with revit-mcp tool names. Use the matching BIMNEXUS tools; anything that needs custom C# (execute_csharp / send_code_to_revit) has no BIMNEXUS equivalent – do it with BIMNEXUS tools or ask the user. Every model change still needs the user's confirmation.

**Summary:** How a wall-built facade is modelled (core walls + thin finish walls, windows hosted in the core, balcony edge band + railing, nested bay-module groups), how to survey it, and how to change or generate bays without ungrouping.
**Related rules:** CORE-001, CORE-002, CORE-004, CORE-006, CORE-013, SAFE-001, SAFE-002, DIM-001, REJ-003 (entries bnx-rule-…)

## Survey first (read-only)

Before changing a facade, map how it is built (CORE-001). One read-only pass, grouped counts only:
1. Walls by `WallType.Kind` (Basic / Curtain / Stacked), type name, grouped or not (`GroupId.Value > 0`), `WALL_KEY_REF_PARAM` (location line), base offset, top constraint + top offset, height.
2. Model groups: top-level groups (`GroupId.Value <= 0` - a top-level group's GroupId is -1, not 0) per level; for each type the direct member categories and the nested group types. Typical pattern: one **floor master group** per floor (or per set of identical floors) nesting repeated **bay module groups**.
3. Windows/doors: family:type, host wall type, sill/head (`INSTANCE_SILL_HEIGHT_PARAM`, `INSTANCE_HEAD_HEIGHT_PARAM`), mirrored, grouped.
4. Railings: type, base offset ("Base Offset"), "Offset from Path", host (often none), path segments from `GetPath()`.
5. Curtain walls: type grid layout/spacing, panel type, mullion types per grid (`CurtainGrid.GetMullionIds()`), height/base, distance to the nearest parallel basic wall.
6. Level heights (floor-to-floor), because module top offsets differ on taller floors.

Reading notes: stacked walls expose internal "System Panel : Wall" panels through the API - they are not curtain panels. Some statistics tools list types under the wrong category; count with direct collectors.

## Core walls + finish-skin walls

A common way to colour facades without curtain systems:
- **Core wall** (block/AAC/CMU, one structure layer, often no material) carries the openings. Location line usually *Core Face: Exterior*. External walls sit on the structural slab (base below FFL by the floor build-up) and stop under the slab/beam above (top = level above minus slab depth).
- **Finish walls** are thin (about 20 mm) single-layer walls with the paint material, drawn tight against the core face (check: a facing core face is coplanar, 0 mm gap). They are pieced around openings - jamb returns, piers, a head band over the openings, a slab-edge band - and usually have wall joins **disallowed at both ends** so Revit does not clean them into the core.
- Windows/doors are hosted **only in the core wall**; the finish walls are not cut. Changing an opening therefore also means trimming or moving the finish pieces around it.
- A head band that starts on the window head line is a recurring recipe: keep window heads on that line when you resize.
- Corner (L-shaped) windows hosted in one wall cut the perpendicular wall through the wall join; breaking that join loses the second leg's opening.

## Balconies and railings

- Balcony = floor finish (tile) from the wall face to the edge + a thin slab-edge finish wall along the balcony outline (e.g. 600 high, from below the slab to just above FFL, unconnected height) + one railing.
- The railing is usually **unhosted**, drawn on the same outline about 10 mm inside the band, base offset = top of the band.
- On podium terraces the railing often stands on an **upstand wall** (block, centreline location) drawn on the same line (path offset 0), base offset = top of the upstand.
- Check consistency: path offsets and base offsets often drift between bays; report them, don't silently normalise.

## Podium facades with louver curtain walls

Horizontal louvers are curtain walls with **no vertical grid** and a **horizontal grid at a fixed distance** (e.g. 150 mm), **empty panels**, and the horizontal mullions as blades (rectangular mullions; depth = blade depth, an offset mullion type for alternating blades). They stand a short distance (25-75 mm) in front of a cladding wall (stone/aluminium finish, location line *Finish Face: Interior* against the backing wall), which sits on a block or dummy-concrete backing wall. Group them per facade stretch.

## Changing a grouped facade (no ungrouping)

The unit of change is the **bay module group type**: change it once and every floor that nests it updates.
- A script **cannot** change a grouped member outside group edit mode. Revit raises the error *"Changes to groups are allowed only in group edit mode ... You may use the Ungroup option"* and shows a modal dialog that blocks the API (calls then time out). The user must click **Cancel** (never Ungroup). Do not retry while the dialog is open.
- Working pattern without ungrouping (CORE-006, REJ-003):
  1. Prepare by script: new types (duplicate, never overwrite the original - CORE-004), the list of module group types to change, which other facades/floors also nest each module type, and the planned change per element.
  2. The user opens **Edit Group** on one module (for a nested module: edit the master group, then the nested one).
  3. While that edit session is open, the script changes only that module's members (swap type, move along the host wall), then reads back.
  4. The user clicks **Finish**; read back again on another floor that nests the same module to confirm propagation.
  5. Repeat per module. Elements that sit loose in a master group (not in a module) are handled in the master group's edit session.
- Alternative that needs no group edit: change a **type** in place (e.g. door width). It updates every instance on every facade at once and nothing moves; use it only when the user accepts a global change.
- Test on one facade/floor first and show before/after images (CORE-002).

## Resizing balcony doors with a minimum solid wall

Typical brief: set balcony sliding doors to a fixed width (e.g. 2200) and keep a minimum solid wall (e.g. 400 mm) between each door edge and the face of the room's side (perpendicular) inside wall.
1. For each door: host wall line, door centre along it, current width (type `WINDOW_WIDTH`), host `Orientation` = exterior.
2. Side walls: core walls on the same level, perpendicular to the host (|dot| < 0.05), one end within host thickness + 100 mm of the host line and running into the interior, nearest face within about 3 m of the door edge. Measure from the door edge to that face.
3. With the new width centred: both gaps >= minimum -> keep the centre; else shift along the host so both gaps hold; if the clear bay is narrower than width + 2 x minimum -> leave the door and report it.
4. Check the host wall is long enough for the new width; a wider insert must cut the adjoining wall through the join, or the host must be extended.
5. List the finish pieces (jambs, returns, head band) that now overlap the wider opening.
6. Read back width, centre, both gaps per door; export the same views before and after.

## Before/after images

Create dedicated test views (prefix e.g. "FACADE TEST - "): a duplicated elevation cropped to the facade, a section through one changed bay (scale 1:50, shallow far clip), a floor plan cropped to the bays (1:50). Hide the clutter in those views only: levels and grids from links, planting/entourage, section/elevation marks; hide heavy reference geometry by workset (`SetWorksetVisibility`) - `HideElements` refuses some groups. Export with `ImageExportOptions` (FitToPage, 2400-3000 px), one or two views per call to stay under the timeout, and the same views and settings before and after.
