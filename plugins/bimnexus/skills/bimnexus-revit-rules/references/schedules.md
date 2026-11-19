# Fixed-row header schedules – build, format and verify

## Build
1. ViewSchedule.CreateSchedule(doc, OST_GenericModel). Add field "Comments".
   AddFilter(Equal "__NO_ELEMENT_ROWS__"). ShowTitle = true. Type the table into the HEADER section.
2. Set field.GridColumnWidth = sum of the column widths FIRST, then the header SetColumnWidth for
   each column. If the body width changes afterwards, Revit makes the header columns equal and
   labels get cut off with "...".
3. Unmerge row 0. InsertColumn(LastColumnNumber+1) / InsertRow(LastRowNumber+1). Unmerge all cells.
   Then set widths, row heights, text and styles. Do the merges LAST.
4. After Regenerate(), get a fresh vs.Definition (a stale one throws "object is not valid").

## Hide the 'Comments' heading (hide only – never delete the field)
- Definition.ShowHeaders = false. Revit turns it back to TRUE after header edits or merges, so set it
  again as the FINAL step and read it back.

## Fonts
- TableCellStyle.TextSize is in POINTS: mm × 72 / 25.4. Set FontName, TextSize, IsFontBold and the
  alignment, with the matching override flags.
- The title cell (row 0, col 0) ignores cell overrides. Use vs.TitleTextTypeId: duplicate a text
  type, e.g. "SCHEDULE TITLE - ARIAL 4.5mm BOLD", width factor 1.0.
- User standard: Arial. Headings 4.5 mm bold, body 3 mm regular, TOTAL row bold. All cells centred
  horizontally and vertically. Body rows 7 mm, heading rows 10 mm.

## Merges
- A vertical merge (e.g. FLOORS over two heading rows) draws the grid line through the text. Instead,
  put the text in a single cell in the upper row and set the shared border between the two cells to
  the 'INVISIBLE LINE' style (BorderBottomLineStyle / BorderTopLineStyle).
- GetMergedCell on a non-anchor cell returns the cell itself. That is normal.

## Placement and checks
- ScheduleSheetInstance.Create, then move it so the top-left corner lands exactly. Check its bounding
  box against the other schedules and viewports on the sheet. Re-stack with 6 mm gaps when sizes change.
- When replacing a live schedule, delete only its sheet instance and keep the schedule view.
- Values are typed. Recompute from Areas in the Rentable scheme: GFA / BUA / LEASABLE Yes-No
  parameters; NSA = LEASABLE = Yes. Refresh them when the area plans change.
- User preference: use native schedules for fixed tables, NOT drafting views with lines and text.

## Reading font sizes from a reference DWF
- A DWF is a zip. The .w2d contains (FontExtension 'Arial') plus binary font opcode 0x06
  (mask, then height as int32). mm = units × PlotInfo scale (≈0.021167).
- The DWF height is the font em height. For Arial, divide by ≈1.4 to get the Revit text height.
