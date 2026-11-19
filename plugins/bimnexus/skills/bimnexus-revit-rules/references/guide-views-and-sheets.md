# Guide: Views, templates, sheets, schedules and legends

> BIMNEXUS note: written with revit-mcp tool names. Use the matching BIMNEXUS tools; anything that needs custom C# (execute_csharp / send_code_to_revit) has no BIMNEXUS equivalent – do it with BIMNEXUS tools or ask the user. Every model change still needs the user's confirmation.

**Summary:** Bulk view/sheet renaming, dependent partial plans, STD view templates, sheet duplication, fixed-row schedules, per-scale legends, image export, drafting views from PDF.
**Related rules:** CORE-002, CORE-003, CORE-004, CORE-008, VIEW-001, VIEW-002, VIEW-003, DOC-002, SAFE-005, PERF-001 (entries bnx-rule-…)

## Bulk renaming of views and sheets

1. **Classify first (read-only).** Collect plan views by `ViewType`, skip templates, and classify each name with patterns (overall / partials parent / part / area plan / ceiling). Build the target name from the view's `GenLevel.Name` (strip level suffixes such as "F.F.L." / "S.S.L."). List what is excluded and why (coordination, presentation, staircase/core, `Copy`/`(1)` duplicates, bare level plans).
2. **Write the mapping to a TSV/CSV** in the project's working folder and let the user review it.
3. **Apply in small batches** by ElementId map (~8-15 renames per call on heavy models; one per call if commits roll back silently). Return `done/remain` and loop.
4. **Sheets:** a sheet that hosts exactly one floor plan gets `Sheet.Name = view name` (sheet number unchanged). Keep descriptive names for sheets with several views (area calculation, RCP sets).
5. **Copies on sheets:** if a sheet hosts `... Copy 1` while the renamed original is unplaced, rename the original `... (NOT USED)`, give the copy the standard name, and let the user delete the NOT USED views.
6. **Finish with a sweep** for any remaining old-format names (views and sheets) and re-apply them one by one.

Notes:
- View names reject `: [ ] | ; < > ? \ "` - strip or replace them before `View.Name = ...`, or the call throws.
- The "Title on Sheet" is `BuiltInParameter.VIEW_DESCRIPTION`, independent of the view name.
- Revit refuses duplicate view names: decide the fallback name (e.g. add a qualifier) before applying.
- Views checked out by another user cannot be renamed - report the owner (`WorksharingUtils.GetCheckoutStatus`).

## Dependent partial plans with scope boxes

1. Duplicate the overall plan **without detailing** (`ViewDuplicateOption.Duplicate`) and name it as the PARTIALS parent.
2. Assign the reference plan template.
3. For each part: `parent.Duplicate(ViewDuplicateOption.AsDependent)`, name it `<parent> - PART 0X` (zero-padded), and set the scope box with `get_Parameter(BuiltInParameter.VIEWER_VOLUME_OF_INTEREST_CROP).Set(scopeBoxId)`.
4. Scope boxes are found by name (`OST_VolumeOfInterest`); spaces in their names matter.

## Standardised ("STD") view templates

- `View.Duplicate` fails on templates, and `ElementTransformUtils.CopyElements` loses Custom RVT-link overrides (the API cannot set links to Custom). Create the new template from a view that uses the source template:
  ```csharp
  View nt = viewUsingSource.CreateViewTemplate();   // keeps link settings + filters
  nt.Name = srcName + " - STD";
  nt.SetNonControlledTemplateParameterIds(src.GetNonControlledTemplateParameterIds());
  ```
  Verify the link override count matches the source.
- **Filter order:** the first filter in `GetOrderedFilters()` wins. To reorder, remove and re-add filters (each goes to the bottom).
- Overrides: `OverrideGraphicSettings` (cut/projection pattern + colour, line weight, halftone) via `SetFilterOverrides` / `SetCategoryOverrides`.
- Keep each view's scale: map `_100 -> _100 - STD`, `_200 -> _200 - STD`.
- Switching links to "By Host View" can make linked content disappear (view-range dependent) - test on one view.
- Template quirks to check: stored scale vs. name, and which parameters are non-controlled (scale, view range, link V/G often are).

## Duplicating sheets with their views

- `sheet.Duplicate(SheetDuplicateOption.DuplicateSheetWithViewsAndDetailing)` (note the exact enum name).
- Then renumber (`SheetNumber`), rename views, and set browser parameters (e.g. a "drawing type" parameter) found by scanning `sheet.Parameters` for the right name.
- Copy revisions with `SetAdditionalRevisionIds(reference.GetAdditionalRevisionIds())` and approval names from a reference sheet.

## Fixed-row (manual) schedules

When the user wants a typed table (e.g. a door-types table) rather than element rows:
1. Create a schedule and add a filter that matches nothing: `new ScheduleFilter(fieldId, ScheduleFilterType.Equal, "__NO_ELEMENT_ROWS__")`.
2. `ShowHeaders = false`; build the table in the **header section** (`GetTableData().GetSectionData(SectionType.Header)`), `InsertColumn`/`InsertRow`, `SetCellText`.
3. Match the user's formatting (font, bold headings, grey fills, row heights). Revit 2024 blocks some `TableCellStyle` background writes through the API - report those as a UI step.

## Legends at several scales

A legend's scale clashes with sheets at other scales. Keep the legend and make one copy per scale:
- `Duplicate(ViewDuplicateOption.WithDetailing)`, set the scale, and rescale the geometry by `k = newScale / oldScale` around a fixed origin.
- FilledRegion: create a new one from the scaled `CurveLoop`s, then delete the old one (more reliable than editing boundaries).
- TextNote: `ElementTransformUtils.MoveElement(newPos - oldPos)`.
- Swap viewports: remember `GetBoxCenter()` and the viewport type, delete, `Viewport.Create`, `ChangeTypeId`.

## Image export for checks

```csharp
var o = new ImageExportOptions{ ExportRange = ExportRange.SetOfViews, FilePath = @"<folder>\chk",
  ZoomType = ZoomFitType.FitToPage, PixelSize = 2400, HLRandWFViewsFileType = ImageFileType.PNG,
  ShadowViewsFileType = ImageFileType.PNG };
o.SetViewsAndSheets(new List<ElementId>{ id });
document.ExportImage(o);
```
- Revit appends ` - Sheet - <number> - <name>.png` (or ` - Drafting View - <name>`).
- Schedules must be placed on a temporary sheet (`ScheduleSheetInstance.Create`) to export; delete the sheet in a separate call.
- 2400 px is enough for checks; 4800 px on heavy sheets adds load and preceded a crash.

## Drafting views rebuilt from a PDF detail sheet

Pipeline that worked: extract vectors/fills/text/images from the PDF (PyMuPDF + Shapely), map pen weights to the project's line styles, text sizes to text types and fills to filled-region types, write one data file per detail, then build with C# in batches of 2-3 views per call:
- `ViewDrafting.Create` only after checking the name doesn't already exist (re-runs after a timeout create duplicates).
- `FilledRegion.Create` then `SetLineStyleId(<Invisible lines>)` - look the id up with `FilledRegion.GetValidLineStyleIdsForFilledRegion`.
- Curves: `NurbSpline.CreateCurve(3, knots, points)` with a fallback to short line segments.
- Very short hatch dots: lengthen to Revit's minimum line length so the texture survives.
- Images: `ImageType.Create` + `ImageInstance.Create` with `ImagePlacementOptions(point, BoxPlacement.Center)`.
- New text/fill types get a ` - DETAIL` suffix and transparent backgrounds so leaders stay visible.
- Sync immediately after the rebuild: an unsynced crash lost the whole first run.
