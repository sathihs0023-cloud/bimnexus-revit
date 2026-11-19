---
name: bimnexus-revit-rules
description: "BIMNEXUS Revit working rules and workflow guides: survey before changing, dry-run then apply, never edit inside groups, annotation/tag placement, areas on wall centrelines, dimension checks, families, views/templates, sheets, linked models, performance and safety. Use before any Revit model change, with or without the BIMNEXUS MCP server."
---

# BIMNEXUS Revit rules and guides

57 rules and 9 workflow guides for safe, consistent Revit work with Claude. They are the same entries the
BIMNEXUS MCP server ships as its read-only Knowledge Pack (`list_knowledge` / `get_knowledge`), packaged here so they also
work in Claude Code without Revit running.

## How to use
1. Before changing a model, read `references/rules-core.md` and `references/rules-safe.md`, then the file for the topic.
2. Follow the order: survey the model → test on one view/floor → dry run and show the list → apply by ElementId after the
   user confirms → short report with counts.
3. A rule with **Severity: must** is not optional. If a user request conflicts with a rule, stop and ask.
4. `references/rules-rej.md` lists approaches that were tried and rejected – do not repeat them.
5. Project or office rules the user keeps in their own knowledge folder override these where they say so.

## Rules
| Topic | File | Count | Rules |
|---|---|---|---|
| Annotation | `references/rules-anno.md` | 5 | Room tags stay inside their own room, without leaders; Door tags centred on the opening; Every lift shaft gets a tag; Tag clash resolver = least-cost move inside the room; Area tags in too-small areas: short leader out (area plans only) |
| Areas | `references/rules-area.md` | 4 | Area boundaries on wall centrelines; Build boundary loops from linked wall centrelines; Nested links: match by inverse transform + level band; Non-room areas follow the element outline / property line |
| Core working rules | `references/rules-core.md` | 14 | Survey the model before changing it; Test on one floor / view first; Dry-run, review, then apply by ElementId; Duplicate, never overwrite; Approval gates on multi-phase tasks; Never ungroup or edit inside groups; Grouped annotation types: edit in place and rename _GRP; Keep view scale and link display settings; Fix families in the family, not in the project; Sync is the user's action - remind at every gate; Plan script-based reverts; never rely on Ctrl+Z; Resolve elements by name across sessions; State assumptions on design elements before building; Conflict with a rule: stop and ask for approval |
| Dimension checks | `references/rules-dim.md` | 3 | Grids are the master reference; Clean at 6 decimals; Full procedure lives in references/guide-dimension-check-compliance.md |
| Reporting | `references/rules-doc.md` | 2 | Short final report with counts; Fixed tables stay fixed |
| Families | `references/rules-fam.md` | 3 | Edit families into new files; Reloading an existing family needs the UI; Curved elements are true curves, real system elements |
| New elements | `references/rules-new.md` | 2 | Revision cloud tags sit beside their cloud, no leader; Elevator calculations: occupancy per DBC Table D.5 and round up group population |
| Performance | `references/rules-perf.md` | 6 | Small batches with a remain counter; Collect with category filters; Read first, write after; Light active view before heavy scripts; Split heavy jobs into match and apply passes; Don't copy views one by one between documents |
| Rejected approaches (do not do these) | `references/rules-rej.md` | 9 | Do not: Room tags pushed outside their room with leaders (floor plans); Do not: Area boundaries from room shells / wall faces; Do not: Ungrouping or editing group members; Do not: Faceted or in-place curved walls; Do not: Loose dimension tolerance; Do not: Element-driven schedule for a fixed table; Do not: Editing originals in place; Do not: Coordinates entered as relative shifts; Never simulate dimensions with lines and text |
| Model safety | `references/rules-safe.md` | 6 | Every write gets a separate read-back; Known silent-rollback triggers; After a timeout: PING, then fresh read-back; Revit unresponsive: stop and ask about dialogs; Final sweep for silently reverted batches; Elements owned by another user |
| Views and templates | `references/rules-view.md` | 3 | Create STD templates from a view, not by duplicating the template; Filter order: the top filter wins; Link display limits of the API |

## Workflow guides
- `references/guide-annotation-cleanup.md` – Guide: Plan annotation cleanup
- `references/guide-areas-and-rooms.md` – Guide: Areas, rooms, numbering and door marks
- `references/guide-dimension-check-compliance.md` – Guide: Dimension Check Compliance
- `references/guide-facade-modelling.md` – Guide: Facade modelling (core + finish-skin walls, windows, balconies, railings, grouped bay modules)
- `references/guide-families-and-modelling.md` – Guide: Families and modelling (tags, ramps, curved walls, sloped floors, railings)
- `references/guide-linked-models-and-coordinates.md` – Guide: Linked models, nested links, cloud links and coordinates
- `references/guide-model-operations.md` – Guide: Model operations, performance and recovery
- `references/guide-template-standardisation.md` – Guide: Rolling a visual standard onto plans (templates, filters, dimensions, tags)
- `references/guide-views-and-sheets.md` – Guide: Views, templates, sheets, schedules and legends
- `references/schedules.md` – Fixed-row header schedules – build, format and verify
