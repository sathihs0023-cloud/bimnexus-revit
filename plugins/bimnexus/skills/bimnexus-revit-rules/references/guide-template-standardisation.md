# Guide: Rolling a visual standard onto plans (templates, filters, dimensions, tags)

> BIMNEXUS note: written with revit-mcp tool names. Use the matching BIMNEXUS tools; anything that needs custom C# (execute_csharp / send_code_to_revit) has no BIMNEXUS equivalent – do it with BIMNEXUS tools or ask the user. Every model change still needs the user's confirmation.

**Summary:** Method to capture a reference view's visual standard and apply it to other buildings - STD templates, filters in order, dimension types, tag families, test floor, rollout.
**Related rules:** CORE-002, CORE-004, CORE-007, CORE-008, CORE-009, VIEW-001, VIEW-002, FAM-001 (entries bnx-rule-…)

## Steps

1. **Capture the reference**: ask which model/view/unit is the reference (marked-up photo welcome). Read its template (filters + overrides, category overrides, detail level, link display), the dimension type values (`LookupParameter` by display name) and the tag families' text types.
2. **Survey the target**: plan templates in use and their view counts by scale, existing filters (reuse their rules), dimension types in use and which sit in groups, tag types in use split by host vs. linked rooms, existing fill/line patterns.
3. **Create STD templates** from views (`CreateViewTemplate`) - keep Custom link settings and scales (see views-and-sheets).
4. **Filters and overrides** on the STD templates only - create missing filters by duplicating; set the required order (first wins).
5. **Dimension type**: duplicate the closest type and set values by display name, guarding `p != null && !p.IsReadOnly`. Swap free dimensions; skip spot dimensions, system types, red check/markup types and anything in groups. Types used in groups: edit in place and rename `_GRP`.
6. **Tag families**: edit in the family and save under a new name; load (UI if overwriting).
7. **Apply to the test floor** (overall + parts), show the user, wait for OK.
8. **Roll out** 1-3 views per call, ~300-400 changes per call, `remain` loop, skipping grouped elements.
9. **Annotation cleanup** (see annotation-cleanup).
10. **Report**: templates created + link settings kept, filters/types created, views switched, items skipped, family files saved, user actions (sync, reload in other models). Write backups (old type values, old tag positions) before changing anything.

## Known issues

- A finish-wall filter with a wide width rule can hide hatches of thin block walls in linked interior models - narrow the rule or reorder.
- After a sync, template ids change - resolve by name.
- Link display set to "By Host View" can hide linked content - test first; Custom link display/colour schemes are UI-only.
