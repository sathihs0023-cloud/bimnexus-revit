# Guide: Model operations, performance and recovery

> BIMNEXUS note: written with revit-mcp tool names. Use the matching BIMNEXUS tools; anything that needs custom C# (execute_csharp / send_code_to_revit) has no BIMNEXUS equivalent – do it with BIMNEXUS tools or ask the user. Every model change still needs the user's confirmation.

**Summary:** Curating view libraries (detach + delete, not copy), bulk deletes and purge, crashes and unsynced work, heavy sheets, multiple Revit sessions on port 8080.
**Related rules:** PERF-004, PERF-006, CORE-010, SAFE-004, SAFE-006, PLUG-001 (entries bnx-rule-…)

## Extracting a subset of a large view library

Copying views into a new document (`ElementTransformUtils.CopyElements`) opens every view and never releases graphics resources: after ~100 views Revit hits its graphics resource limit ("A required resource was unavailable") and crashes, closing all documents.
Instead:
1. The user detaches the source (discard or keep worksets as they prefer) and **Save As** the new file.
2. Delete everything not on the keep-list: sheets, schedules, other views, unused templates - `document.Delete(ids)` in batches of ~40 with a `Stopwatch` limit (~12-20 s per call), returning `remain`.
3. Remove unwanted parameters (`ParameterElement`) and purge unused elements; verify the final counts.
4. Rename views back to their library names if needed (sheet number + title, from `ViewSheet.GetAllPlacedViews()` and sheet parameters).

## Crashes and lost work

- The runner cannot save or sync: unsynced work is lost on a crash. Ask the user to sync at every milestone and right after any rebuild.
- After a crash, first check what survived (by name) and rebuild only what is missing, in smaller batches.
- Diagnose hangs/crashes from the OS: `Get-Process Revit`, the Revit journal files under `%LOCALAPPDATA%\Autodesk\Revit\Autodesk Revit <year>\Journals`, and the MCP log.

## Performance

- Heavy sheets with many links hang on "Generating graphics": ask the user to open a light view (schedule/legend) and close sheet tabs first.
- Associative dimensions on elements you move force regeneration on every move - clear check dimensions before bulk moves and re-create them after.
- Self-limit long loops with `System.Diagnostics.Stopwatch` and return `remain=N`.

## Several Revit sessions

Only one Revit session can own port 8080; the MCP server talks to whichever started its service first. Check ownership with
`Get-NetTCPConnection -LocalPort 8080 -State Listen | Select OwningProcess` and match it to `Get-Process Revit`.

## Worksharing

- Elements checked out by a colleague can't be edited: report the owner and ask for a sync/relinquish.
- After a sync the Undo stack is cleared and new elements may get new ids - resolve by name.
