# Dimension checks

## Grids are the master reference

Elements move to fit the grids, never the other way round. Correct a grid only after the user approves it. Doors and windows are measured from their host wall, never from grids, and are fixed only after all walls.

**Severity:** must

## Clean at 6 decimals

Dimension compliance means round(value, 6) is a multiple of the module (default 50 mm): residual error < 5e-7 mm. Angles exact to 1e-11 deg, measured with asin(|a x b|).

**Severity:** must

## Full procedure lives in references/guide-dimension-check-compliance.md

Follow the phased procedure (P0 audit -> P1 views -> P2 as-is dimensions -> P3a walls / P3b doors / P3c windows / P3d slabs -> P4 verify) in knowledge/references/guide-dimension-check-compliance.md.

**Severity:** must
