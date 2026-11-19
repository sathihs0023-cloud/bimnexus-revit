# Guide: Areas, rooms, numbering and door marks

> BIMNEXUS note: written with revit-mcp tool names. Use the matching BIMNEXUS tools; anything that needs custom C# (execute_csharp / send_code_to_revit) has no BIMNEXUS equivalent – do it with BIMNEXUS tools or ask the user. Every model change still needs the user's confirmation.

**Summary:** Area boundaries on wall centrelines from linked walls, area naming from link rooms, terrace assignment by shared edge, room naming from nested links, room/door numbering algorithms.
**Related rules:** AREA-001, AREA-002, AREA-003, CORE-003, CORE-006, PERF-005, REJ-002 (entries bnx-rule-…)

## Area boundaries on wall centrelines

- Boundaries follow **wall centrelines** - never wall faces, never traced around columns.
- `Room.GetBoundarySegments(new SpatialElementBoundaryOptions{ SpatialElementBoundaryLocation = Center })` misses segments contributed by walls in **other links** (STR/BLD). Build loops from the `LocationCurve` of the walls in those links instead (transform each curve with the link's `GetTotalTransform()`), trimmed/extended to their intersections.
- `Room.ClosedShell` gives face-of-wall outlines wrapped around columns - wrong for area boundaries.
- Create lines with `document.Create.NewAreaBoundaryLine(sketchPlane, curve, areaPlanView)`, then place areas (`NewArea(view, uv)`) and read back their area values.
- If writes "succeed" but nothing changes, a dialog may be open in Revit - stop and ask the user.

## Area naming from linked rooms

1. For each area, take its location point, transform it into the room link's coordinates (`GetTotalTransform().Inverse.OfPoint(p)`), and find the room with `room.IsPointInRoom(q)` (z = room level + 1 ft). Only use links whose title matches the agreed link filter.
2. Skip areas in groups (`GroupId.Value > 0`) - renaming grouped areas triggers the group dialog and rolls back the whole run.
3. Max ~2 levels per call; verify with a separate read-back.

## Terraces and other small areas

- Candidate = small area (e.g. < 50 m², excluding MEP/GYM-type names) with a placeholder name (varies per model - survey first).
- Assign it to the unit area with the **longest shared boundary** (≥ ~3 ft); a unit can own several terraces.
- Boolean area parameters (e.g. "is terrace", "is sellable") must be bound to the Areas category - if not, ask the user to add them as project parameters.

## Room names from nested unit links

- Unit links nested inside another link have a null host bounding box. Compute the local bbox from the link document's walls/floors.
- Match each room by inverse-transforming its location point into the link's local coordinates: XY inside the local bbox **and** local Z within a small band above the link's own floor (e.g. -2..+6 ft). World-AABB matching mis-names rooms on non-typical floors (rotated/stacked instances overlap).
- Derive the name from the link type name with a regex (e.g. a 3-character unit code formatted as `1B-A`).
- Report rooms inside no link and links with no room (missing/duplicate rooms) for manual review.

## Room numbering

- Take the floor from the room's **actual level** (parse the ordinal from the level name), not from the old number.
- Order rooms spatially per floor (X then Y with a small tolerance) or along a path seeded by user-selected anchors (greedy nearest-neighbour walk) when the user wants corridor order.
- Write with `room.get_Parameter(BuiltInParameter.ROOM_NUMBER).Set(...)` (works on grouped rooms too).
- Dry-run CSV first; check for duplicates and collisions with existing numbers before applying.

## Door marks from rooms

- Door Mark = number of the room it opens into: `FamilyInstance.ToRoom[phase]`, else `FromRoom[phase]`.
- When a door connects a general space (corridor, lobby, parking, open-to-below) and a specific room, use the specific room.
- Several doors to the same room get suffixes a, b, c (sorted spatially).
- Write `ALL_MODEL_MARK`. Build a QA schedule with room number, door mark, from/to room to let the user check.
