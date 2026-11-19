# Rejected approaches (do not do these)

## Do not: Room tags pushed outside their room with leaders (floor plans)

**REJECTED APPROACH – do NOT do this.**

Moving ROOM tags on floor plans out of their room into corridors with leaders to solve clashes. Exception: AREA tags on area plans whose area is too small for the tag may use a leader (ANNO-005).

**Why:** Rejected by the user and reverted by script.

**Do instead:** ANNO-001 / ANNO-004: least-cost move inside the room, no leader; list tiny rooms for manual review. Area plans: ANNO-005.

**Severity:** must

## Do not: Area boundaries from room shells / wall faces

**REJECTED APPROACH – do NOT do this.**

Deriving area boundaries from the room ClosedShell or face-of-wall outlines wrapped around columns.

**Why:** Rejected: boundaries must follow wall centrelines.

**Do instead:** AREA-001 / AREA-002.

**Severity:** must

## Do not: Ungrouping or editing group members

**REJECTED APPROACH – do NOT do this.**

Ungrouping areas/details or editing elements inside groups to apply a change.

**Why:** Rejected ('do not ungroup the areas'); also causes silent rollback.

**Do instead:** CORE-006 / CORE-007.

**Severity:** must

## Do not: Faceted or in-place curved walls

**REJECTED APPROACH – do NOT do this.**

Approximating a curved ramp upstand with many short straight wall segments, or modelling it in-place.

**Why:** Rejected: needs true curved wall elements.

**Do instead:** FAM-003.

**Severity:** must

## Do not: Loose dimension tolerance

**REJECTED APPROACH – do NOT do this.**

Accepting dimensions as clean within 0.5 mm (or any tolerance looser than 6 decimals).

**Why:** Rejected: fractions remained visible in the check views.

**Do instead:** DIM-002.

**Severity:** must

## Do not: Element-driven schedule for a fixed table

**REJECTED APPROACH – do NOT do this.**

Building a requested fixed door-types table as a schedule that pulls rows from model elements.

**Why:** Rejected: the user wanted a manually typed, fixed-row table.

**Do instead:** DOC-002.

**Severity:** must

## Do not: Editing originals in place

**REJECTED APPROACH – do NOT do this.**

Modifying the model's existing templates, filters, types or families directly.

**Why:** Rejected: originals must stay untouched for rollback.

**Do instead:** CORE-004.

**Severity:** must

## Do not: Coordinates entered as relative shifts

**REJECTED APPROACH – do NOT do this.**

Treating user-entered survey point / coordinate values as 'shift by' amounts.

**Why:** Rejected: values the user types are absolute targets; the tool computes the difference.

**Do instead:** Read the current value, compute target - current, apply only the difference, read back.

**Severity:** must

## Never simulate dimensions with lines and text

**REJECTED APPROACH – do NOT do this.**

Do not create lines, curves, ticks, witness graphics, or text notes to imitate dimensions in Revit. Use a native Revit Dimension element only.

**Why:** Graphic substitutes are not associative Revit dimensions and violate the user's documentation standard.

**Do instead:** Create a native Revit Dimension. If the server/API cannot create the required dimension type, report the limitation and do not create substitute graphics.

**Severity:** must
