# Guide: Families and modelling (tags, ramps, curved walls, sloped floors, railings)

> BIMNEXUS note: written with revit-mcp tool names. Use the matching BIMNEXUS tools; anything that needs custom C# (execute_csharp / send_code_to_revit) has no BIMNEXUS equivalent – do it with BIMNEXUS tools or ask the user. Every model change still needs the user's confirmation.

**Summary:** Editing and reloading families safely, true curved/sloped upstand walls via mass + FaceWall, sloped floors and railings by API.
**Related rules:** FAM-001, FAM-002, FAM-003, CORE-009, CORE-013, REJ-004 (entries bnx-rule-…)

## Editing families (tags and others)

1. `Document fam = document.EditFamily(family)`.
2. In the family doc open your **own** Transaction (allowed there): duplicate or adjust `TextElementType`s, move labels with `ElementTransformUtils.MoveElement`, toggle shown parameters.
3. `fam.SaveAs(<project Families_STD folder>\<NEW name>.rfa)` - never overwrite the original - then `fam.Close(false)`.
4. Load: a new family name may load with `document.LoadFamily(path, out f)`; if it fails with "document must not be modifiable", or when overwriting an existing family, use the UI: Insert > Load Family > "Overwrite the existing version" (answer Yes to make elements editable in workshared models).
5. Verify by script: type count, label positions, parameter values.

## Curved and sloped upstand walls (e.g. ramp edges)

The user wants **true curved wall elements**, not many straight segments and not in-place models.
1. Read the selected floor's top-face boundary edges; separate curved and straight segments; chain them end to end (tolerance ~10 mm), ordered bottom to top.
2. For curved runs: build a solid **mass loft** family from the edge point chains (cross-sections every few points), load it, place it.
3. Create walls with `FaceWall.Create` on the mass faces - check `FaceWall.IsValidFaceReferenceForFaceWall(document, reference)` first; typically the largest face of each solid is valid. Keep the mass in the project as the host.
4. For straight sloped runs: `Wall.Create` with a profile (sloped top).
5. Tag everything created with a `Comments` value (e.g. "RAMP UPSTAND 1000") so it can be selected or removed later.

## Sloped floors and railings

- `Floor.Create(document, loops, typeId, levelId, false, slopeArrowLine, slope)` - the arrow line direction sets the slope direction; offset with `FLOOR_HEIGHTABOVELEVEL_PARAM`.
- Call `document.Regenerate()` once before reading bounding boxes to verify Z values.
- `Autodesk.Revit.DB.Architecture.Railing.Create(document, curveLoop, railingTypeId, levelId)` (fully qualify `Railing`); set `HostId` in a try/catch - hosting can fail on some configurations.
- The API cannot create native Ramp elements: use sloped floors + railings and say so.
- State any assumed design values (slope, run length, widths, landing depth, rail height, code basis) and ask the user to confirm.
