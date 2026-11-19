# Areas

## Area boundaries on wall centrelines

Area boundary lines go on wall centrelines only - never traced around columns or along wall faces. New areas take their names from the matching room names, and area parameters follow the criteria of the existing areas.

**Severity:** must

## Build boundary loops from linked wall centrelines

Room.GetBoundarySegments does not include segments from walls modelled in other links (STR/BLD). Build closed loops from the LocationCurve centrelines of walls in those links.

**Severity:** should

## Nested links: match by inverse transform + level band

get_BoundingBox(null) is null for nested link instances. Compute the local bbox from the link document, transform with GetTotalTransform(), and match points by inverse-transforming them into link coordinates with a Z band anchored to the link's own floor - never by world AABB (rotated/stacked instances overlap).

**Severity:** must

## Non-room areas follow the element outline / property line

Exception to AREA-001: areas for things that are not rooms (canopies, shading sails, plot coverage) follow the element's plan outline or the property line, not wall centrelines. Merge touching outlines of one category into one area.

**Why:** Lead accepted outline-based boundaries for canopy/shading areas and the property line for plot coverage (noted as a user exception to AREA-001).

**Severity:** should
