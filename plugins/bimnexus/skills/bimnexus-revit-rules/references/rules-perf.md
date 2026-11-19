# Performance

## Small batches with a remain counter

Keep batches small and loop with `remain=N`: 1-3 views per call; ~8-15 renames per call on heavy models; ~300-400 parameter changes; walls with many associative dimensions 1-3 per call; drafting views with full geometry 2-3 per call.

**Why:** Larger batches timed out and rolled back.

**Severity:** must

## Collect with category filters

> BIMNEXUS note: written with revit-mcp tool names. Use the matching BIMNEXUS tools; anything that needs custom C# (execute_csharp / send_code_to_revit) has no BIMNEXUS equivalent – do it with BIMNEXUS tools or ask the user. Every model change still needs the user's confirmation.

Use OfCategory / ElementMulticategoryFilter (+ BoundingBoxIntersectsFilter for links). Never scan a view with a bare FilteredElementCollector(doc, viewId) on models with many links.

**Severity:** must

## Read first, write after

> BIMNEXUS note: written with revit-mcp tool names. Use the matching BIMNEXUS tools; anything that needs custom C# (execute_csharp / send_code_to_revit) has no BIMNEXUS equivalent – do it with BIMNEXUS tools or ask the user. Every model change still needs the user's confirmation.

Collect all positions/bounding boxes first, then do all moves. Do not call get_BoundingBox / geometry reads after a move inside a loop, and avoid document.Regenerate() in loops.

**Severity:** must

## Light active view before heavy scripts

Ask the user to switch the active view to a schedule or legend and close sheet tabs before heavy scripts; heavy sheets with many links hang on 'Generating graphics'.

**Severity:** must

## Split heavy jobs into match and apply passes

For heavy write jobs, run a read-only matching pass that outputs an Id:value list, then a fast apply pass that sets values directly from that list without collectors.

**Severity:** must

## Don't copy views one by one between documents

Copying views between documents through the API exhausts Revit's graphics resources (~100 views -> crash). To extract a subset of a library, Save-As a copy of the whole source model and delete what is not needed.

**Severity:** should
