# Annotation

## Room tags stay inside their own room, without leaders

Tags must be readable inside their own room and must not overlap dimensions, other tags, text, walls, columns, doors or stairs (furniture overlap is tolerated). If a room is too small, keep the least-bad spot inside it and list it for manual review.

**Severity:** must

## Door tags centred on the opening

The centre of a door tag sits exactly on the door's insertion point (midpoint of the opening), with no leader. Report tags whose door is missing/unloaded.

**Severity:** must

## Every lift shaft gets a tag

Every lift shaft room visible in a plan (host or linked) gets a room tag of the standard unit type.

**Severity:** must

## Tag clash resolver = least-cost move inside the room

Resolve tag clashes by scoring candidate positions inside the tagged room: weighted overlap (annotation 1000, hard model elements 100, furniture 5) plus a distance penalty; move only if cost improves. Order: leaders off -> lift tags -> room-tag resolver -> door-tag centring -> read-back.

**Severity:** must

## Area tags in too-small areas: short leader out (area plans only)

On area plans (BUA/GFA/rentable area-calculation plans), first move a clashing area tag least-cost inside its area. Only if the area is too small for the tag (narrow facade terraces, small core rooms): turn the leader on, keep the tag horizontal, put the text in clear space outside the area (white space beyond the facade for perimeter areas, the nearest free area for core rooms) without overlapping any annotation, and put the leader end in the middle of the area. Leader short, single, straight, horizontal preferred. Clean tags are never moved; duplicate tags on one area are reported, not moved. Room tags on floor plans stay under ANNO-001 / REJ-001.

**Why:** Narrow terraces and core rooms on small-scale area plans cannot hold the tag text inside the boundary; demonstrated by the BIM lead in a screen recording of a BUA area calculation plan.

**Severity:** must
