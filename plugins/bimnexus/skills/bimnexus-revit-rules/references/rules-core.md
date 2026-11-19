# Core working rules

## Survey the model before changing it

Before any write, read the model's own conventions: links and their names, naming patterns, parameter bindings, existing types/templates/filters, grouped elements. Survey every building separately; never assume one model matches another.

**Why:** Buildings differ (link names, placeholder names, parameter bindings). Runs based on assumptions failed or had to be redone.

**Severity:** must

## Test on one floor / view first

Apply any rollout to a single test scope first (e.g. one floor's OVERALL + PART views), show the result, and wait for the user's OK before pushing to all views or buildings.

**Why:** The user reviews visually before scale-out; mistakes stay small.

**Severity:** must

## Dry-run, review, then apply by ElementId

For bulk edits: (1) dry-run pass that computes the changes and writes a CSV/TSV to the project's BIM-MCP working folder, (2) user review, (3) apply pass driven by an explicit ElementId -> value map, (4) separate read-back verification.

**Why:** Lets the user check the mapping before the model changes, and makes the apply pass fast and repeatable.

**Severity:** must

## Duplicate, never overwrite

Templates, filters, line/fill patterns, dimension types and families are duplicated, modified and then applied. New names get the suffix ' - STD' (or a new family name). The original stays untouched so the change can be rolled back.

**Why:** Keeps every change reversible and protects elements used elsewhere.

**Severity:** must

## Approval gates on multi-phase tasks

Heavy tasks run in phases (audit -> setup -> as-is check -> changes per element type -> verification). Report at the end of each phase and wait for explicit approval. Approval of one phase never implies the next.

**Why:** The user controls every model change; a crash costs at most one phase.

**Severity:** must

## Never ungroup or edit inside groups

Skip elements with GroupId > 0 (areas, dimensions, tags, model elements) and report them. Never ungroup to make a change.

**Why:** Editing group members makes Revit roll back the whole transaction silently ('Changes to groups are allowed only in group edit mode'); 12 floors of edits were lost once.

**Severity:** must

## Grouped annotation types: edit in place and rename _GRP

When a dimension/annotation type is used inside groups, modify that type in place to the standard values and rename it with the suffix '_GRP' instead of swapping elements inside the groups.

**Why:** Changes the look without touching group definitions.

**Severity:** must

## Keep view scale and link display settings

When replacing or duplicating view templates keep each view's scale (_100 -> _100 - STD, _200 -> _200 - STD) and keep the template's Custom RVT-link display settings. Losing link settings is a failure even if everything else is right.

**Why:** Sheets break when scales change; linked models are the content of the plans.

**Severity:** must

## Fix families in the family, not in the project

Layout problems that come from a family (label positions, text types, shown fields) are fixed by editing the family, not worked around in the project.

**Severity:** must

## Sync is the user's action - remind at every gate

The script runner cannot save or sync. At every approval gate and after every large batch, ask the user to Sync with Central (or save) before continuing.

**Why:** Revit crashed twice before a sync and all work was lost.

**Severity:** must

## Plan script-based reverts; never rely on Ctrl+Z

> BIMNEXUS note: written with revit-mcp tool names. Use the matching BIMNEXUS tools; anything that needs custom C# (execute_csharp / send_code_to_revit) has no BIMNEXUS equivalent – do it with BIMNEXUS tools or ask the user. Every model change still needs the user's confirmation.

Before bulk moves or type swaps, store the original values (positions, types, parameter values) in a backup file so they can be restored by script. After a sync the Undo stack is gone.

**Severity:** must

## Resolve elements by name across sessions

> BIMNEXUS note: written with revit-mcp tool names. Use the matching BIMNEXUS tools; anything that needs custom C# (execute_csharp / send_code_to_revit) has no BIMNEXUS equivalent – do it with BIMNEXUS tools or ask the user. Every model change still needs the user's confirmation.

Look up templates, types, views and sheets by name at the start of each run. Treat stored ElementIds as hints only; IDs of new elements can change after a sync.

**Severity:** must

## State assumptions on design elements before building

When asked to model a design element (ramp, stair, railing, wall system) with missing parameters, list the assumed values (slope, widths, heights, code basis such as Dubai Building Code) and ask for confirmation before or right after building.

**Why:** Ramp test used assumed 1:12 slope, widths and railing height that the user never specified.

**Severity:** should

## Conflict with a rule: stop and ask for approval

If a request conflicts with an approved rule or a rejected approach, stop before any model change, name the rule ID and the conflict, and ask the user: 'This may conflict with your model standards (<rule ID>). Do you approve continuing anyway - yes or no?'. Continue only on an explicit yes; otherwise offer a compliant alternative. Record approved exceptions in the final report.

**Why:** The BIM lead decides when a standard may be broken; Claude must never break one silently.

**Severity:** must
