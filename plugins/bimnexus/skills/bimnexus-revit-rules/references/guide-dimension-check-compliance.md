# Guide: Dimension Check Compliance

> BIMNEXUS note: written with revit-mcp tool names. Use the matching BIMNEXUS tools; anything that needs custom C# (execute_csharp / send_code_to_revit) has no BIMNEXUS equivalent – do it with BIMNEXUS tools or ask the user. Every model change still needs the user's confirmation.

**Summary:** Phased procedure to snap columns, walls, doors, windows and slab edges to clean grid offsets, with check views and 6-decimal dimensions.
**Related rules:** DIM-001, DIM-002, DIM-003, CORE-005 (entries bnx-rule-…)

## Overview

**Purpose:** a general tool to make every element in a Revit model sit at clean, fraction-free dimensions from its reference (grid or host wall), and to prove it with dedicated check views.
**Applies to:** any Revit model · **Engine:** revit-mcp `send_code_to_revit` (C#) / `dimension_check_compliance` tool
**Status:** living document. Rule changes go through the knowledge base (rules.json / Notion), then this file.

---

## 0. Configuration (set per company or project before running)

| Key | Default | Notes |
|-----|---------|-------|
| Snap module | **50 mm** | Every checked offset must be a multiple of this. |
| Check dimension type | `<Company>_Check` dimension type (6-decimal display) | Create it if missing: linear, 6-decimal rounding, distinct colour. |
| Check view template | `<Company>_CHECK VIEWS_100` | Created by the tool (see §7). |
| Browser category parameter / value | e.g. `View-Category = USER WORKING VIEWS` | Use the model's own browser-organisation parameters. |
| Browser sub-category parameter / value | e.g. `View Type = CHECK VIEWS` | |
| View name pattern | `{Level name} - CHECK VIEW` | Strip level suffixes such as "F.F.L." / "S.S.L.". |
| Output folder for dry-run CSVs | the project's BIM-MCP working folder | |

---

## PHASED EXECUTION SEQUENCE (mandatory)

The task runs in **phases**. Each phase ends at an **approval gate**: Claude reports what the phase will do (or did), and **waits for the user's explicit approval before starting the next phase**. No model change is made in a phase the user has not approved.

| Phase | What happens | Model changes? | Gate |
|-------|--------------|----------------|------|
| **P0 — Audit (read-only)** | Grid audit (§2), element inventory per level, levels that host in-scope elements, existing check views / template / dimension type. | None | Report → user approves P1 |
| **P1 — Views & template** | Create the check view template (§7.3–7.4) and **one check floor plan per level that hosts in-scope elements** (§7.1–7.2). Create the check dimension type if missing (§0). | Views, template, dimension type only. **No element is moved.** | Report view list → user approves P2 |
| **P2 — Dimensions & angles (as-is audit)** | In every check view: place the §8 dimensions on the **current, unmodified** geometry and run the **angle check** (§4). Output: dimension values (6 decimals), non-clean dimensions, angle deviations, elements that could not be dimensioned (§8.7). | Dimensions only. **No element is moved.** | Report failures → user approves P3 |
| **P3 — Implement changes, element by element** | Fix elements in this order, each sub-step with its own dry-run CSV and approval: **P3a Walls** (incl. curtain walls: rotation → across → ends), **P3b Doors**, **P3c Windows**, optional **P3d Slab sketch lines**. After each sub-step re-read every dimension, add dimensions that were impossible in P2, and report. | Yes — per approved sub-step only | One approval per sub-step |
| **P4 — Verification** | §9 final audit of every dimension value and count; exceptions report. | None | Final report; user saves / syncs |

Gate rules:
1. Before each gate, write the dry-run CSV (§9.1) and a short summary: counts, max shift, max rotation, skipped/exception elements.
2. Approval is per phase / sub-step. Approval of one sub-step never implies the next.
3. If a call times out or Revit stops responding, stop, ping (§10.2), re-verify, and report before continuing — never push the next batch blind.

---

## 1. Scope — categories

| # | Category | What is checked |
|---|----------|-----------------|
| 1.1 | **Structural Columns** | Position (both local axes) and rotation, measured to the nearest grid. |
| 1.2 | **Walls** (basic and stacked) | Rotation, across-wall face offset, and wall-end offset to the nearest grid. |
| 1.3 | **Curtain Walls** | As walls: rotation, location-line offset across the wall, end offset. Mullions/panels follow the wall. |
| 1.4 | **Slab sketch lines** (boundary sketch lines only) | Each straight line exactly parallel/perpendicular to a grid, at a clean offset from the nearest parallel grid. Arcs: endpoints only. Optional P3d. |
| 1.5 | **Doors** | Measured **from the host wall**, never from grids (§6). |
| 1.6 | **Windows** | Measured **from the host wall**, never from grids (§6). |
| 1.7 | Grids: **host-model grids only**. Grids are the master reference: elements move to fit the grids, never the other way round (except §2.3). | |
| 1.8 | Skip and report: in-place / generic families with no usable geometry, **pinned elements, elements inside groups**, curved (arc) walls. | |

## 2. Grid systems

| # | Rule |
|---|------|
| 2.1 | Group linear grids by exact direction (angle mod 180°). A model may have orthogonal, rotated, radial and arc grid sets. |
| 2.2 | Two grids are **parallel only if their angle difference is < 1e-6°**. |
| 2.3 | If a grid is off-parallel from its set by a sub-micron amount, **ask the user** before correcting it; if approved, rotate it about its intersection with its primary/arc grid. |
| 2.4 | "X / Y" means the element's **own two local axes**, not global X/Y. |
| 2.5 | A radial grid passes within 1 mm of an arc grid's centre. |

## 3. Processing order inside P3 (mandatory)

1. **Walls & curtain walls:** rotation + across offset together, iterate until converged (joins drag neighbours), then wall ends (§5.8).
2. **Doors**, only after all walls are fixed (§6).
3. **Windows**, only after all walls are fixed.
4. (Optional) **Slab sketch lines** (§5.9).

## 4. Rotation — angle check

| # | Rule |
|---|------|
| 4.1 | Every column, wall, curtain wall and straight slab sketch line must be **exactly** parallel or perpendicular to a grid set (tolerance 1e-11°). |
| 4.2 | Target angle = exact direction of the nearest grid set within 0.5°, read from a grid (never typed). |
| 4.3 | Elements in a radial/arc zone align to the **nearest radial grid** within 2.5° (report each). Lines > 0.5° off every non-radial set (e.g. 45° chamfers) are **not rotated** — report them. |
| 4.4 | Rotate columns about their location point, walls about their midpoint. Report rotations > 0.01°. |
| 4.5 | **Measure deviation with `asin(|a × b|)`, never `acos(a · b)`.** |
| 4.6 | P2 angle report: every element's deviation (deg, 3 s.f.); flag > 1e-11°. Deviations ≥ 0.01° also get an angular dimension (run last, one per call — §10.7). |

## 5. Position — snap to the clean module

| # | Rule |
|---|------|
| 5.1 | Reference per axis = **nearest parallel grid** whose extent covers the element (±3 m); else nearest parallel grid anywhere (marked `*`). |
| 5.2 | Measure the **element face nearest the grid**, not the centre. For walls compute faces in the wall's own frame, then apply after rotation. |
| 5.3 | Target: near-face offset = **nearest multiple of the module** (max shift = module ÷ 2). |
| 5.4 | If the grid passes through the element and its centre is within 1 mm of it, **centre it exactly** on the grid. Report these. |
| 5.5 | Arc grids: offset along the radius on the element centreline `s_arc = √(R² − lat²)`; report that this axis has no dimension. |
| 5.6 | Precision target: **error < 5e-7 mm** (clean at 6 decimals). |
| 5.7 | **Revit ignores moves smaller than ~1e-5 mm.** Use a detour: location curve 10 mm off then to the target; `LocationPoint.Point = p + (3,3,0)` then `= target`. |
| 5.8 | **Wall ends:** only **true planar end faces**. Prefer snapping a **free (unjoined) end**; if both ends are joined and none is clean, report. Move the **location-curve end point** (with detour), never the whole wall. |
| 5.9 | **Slab sketch lines:** edit with `CurveElement.SetGeometryCurve(newLine, true)` (`Sketch.GetAllElements()`); **SketchEditScope cannot start inside the revit-mcp transaction**. Recompute corners from neighbouring lines so loops stay closed; freeze lines adjacent to arcs. Max 3 floors per call. Never change slab type, thickness or level. |

## 6. Doors & windows (host-wall referenced)

| # | Rule |
|---|------|
| 6.1 | Run **only after every wall is fixed**. |
| 6.2 | Reference = the **host wall**: from the **nearest wall end face or intersecting wall face** to the **nearest jamb**. |
| 6.3 | Snap that offset to the module by moving the insert along the wall only. Never change host, sill, width or type. |
| 6.4 | If the width isn't a multiple of the module, snap the near jamb and report the far jamb. |
| 6.5 | **Record insert positions before** wall-end edits and restore them after (`FindInserts`). |

## 7. Check views & template (P1)

| # | Rule |
|---|------|
| 7.1 | One floor plan per level that hosts in-scope elements, named with the §0 pattern. |
| 7.2 | Set the §0 browser category / sub-category parameters. |
| 7.3 | Template `<Company>_CHECK VIEWS_100`: 1:100, Medium. **Visible:** Structural Columns, Walls, Curtain Panels, Curtain Wall Mullions, Floors, Doors, Windows, Grids, Dimensions. **Hidden:** everything else, including **RVT Links** and imported CAD. |
| 7.4 | View range: Top +2300, Cut +1200, Bottom −3500, Depth −3500 (adjust to storey height). |
| 7.5 | Dimension each element in the view whose cut plane passes through it; otherwise the nearest view that shows it. |

## 8. Dimensions (P2, refreshed after each P3 sub-step)

| # | Rule |
|---|------|
| 8.1 | Use the §0 check dimension type (6 decimals). |
| 8.2 | **One dimension per element per direction**, **2 references**: nearest grid → nearest face. |
| 8.3 | Doors/windows: one dimension along the host wall, wall end/intersecting face → nearest jamb. |
| 8.4 | If the nearest face lies on the grid (< 1 mm), dimension to the opposite face. |
| 8.5 | Wall "along" dimensions use true end faces only. |
| 8.6 | Slab sketch lines: dimension the slab **edge face** to the nearest parallel grid. |
| 8.7 | Family-instance faces: `GetSymbolGeometry()` faces as `UniqueId + ":0:INSTANCE:" + stableRep`. Revit deletes dimensions to non-parallel faces — list them in P2 as "not dimensionable until rotated". |
| 8.8 | Placement 600 mm beyond the face; walls at ±30 % of length. |
| 8.9 | Dimensions are associative: re-read after each P3 sub-step, don't recreate. |

## 9. Verification (mandatory)

| # | Rule |
|---|------|
| 9.1 | Dry-run first (CSV written by the script with `System.IO`), then apply, then **audit with a fresh read-back**. |
| 9.2 | Final check: read **every dimension value**. Pass = `round(value, 6)` is a multiple of the module (centred cases from §5.4 reported). |
| 9.3 | Confirm the dimension count per view; Revit deletes invalid dimensions on regeneration. |
| 9.4 | Confirm door/window positions after wall-end edits. |
| 9.5 | Report: elements moved/rotated (max shift), skipped elements, axes without dimension, grid corrections. |

## 10. revit-mcp execution rules

| # | Rule |
|---|------|
| 10.1 | Variable is `document`. No own `Transaction`; no `SketchEditScope`. Avoid LINQ on Revit 2024; fully qualify `System.Text.StringBuilder` and `System.IO`. |
| 10.2 | ~60 s timeout, but Revit may still commit; the NEXT call may return the stale result. Send `PING-n` until the matching reply returns, then re-verify. |
| 10.3 | Batch sizes: walls ≤ 10–20 per call (≤ 1–3 when many check dimensions exist); dimensioning ≤ ~90 elements per call; slab sketches ≤ 3 floors per call. No `Regenerate()` in loops. |
| 10.4 | **Silent rollback:** re-measure after each batch and re-apply missing elements in small ID-listed groups. |
| 10.5 | Revit unresponsive > 3 min: **stop**, ask the user to check for an open dialog (no Ctrl+Z). |
| 10.6 | One Ctrl+Z reverts a whole run. **Every gate asks the user to Sync with Central first.** |
| 10.7 | `AngularDimension.Create` on slab edges and dimensioning right after heavy wall edits have hung Revit: run angular dimensions last, one per call, after a Sync. |
| 10.8 | Moving elements carrying many check dimensions is slow and has crashed Revit. Before P3 offer `clear_dimensions` (values saved in the P2 CSV), run P3 without dimensions, then re-run the P2 `dim_*` actions as P4. |

## 11. MCP tool — `dimension_check_compliance`

Say **"Dimension Check Compliance"** and Claude runs this tool, one `action` per step; every write action needs `confirm=true`, set only after the user approved that phase and synced.

| Phase | Tool calls (in order) |
|-------|------------------------|
| — | `rules` · `ping` (after any timeout) |
| **P0 Audit** | `audit`; grid correction only on approval: `fix_grid` |
| **P1 Views** | `create_views` |
| **P2** | `dim_walls` (loop `start`) → `dim_curtain_walls` → `dim_inserts` doors, windows → `dim_slabs` → `angle_report` → `audit_dimensions` tag=`P2_before` |
| **P3a Walls** | `walls_across` dry_run → apply (repeat until needFix=0) → `wall_ends` dry_run → apply |
| **P3b Doors / P3c Windows** | `inserts` dry_run → apply |
| **P4 Verify** | `audit_dimensions` tag=`P4_after` (+ `dim_*` for newly dimensionable elements) |

All names (module, dimension type, template, view suffix, browser parameters, output folder) are parameters, so the tool works on any company's model. Company defaults are in the private rules.
