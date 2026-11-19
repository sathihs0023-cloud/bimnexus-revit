# Families

## Edit families into new files

document.EditFamily -> own Transaction in the family doc -> SaveAs under a NEW name in the project's Families_STD folder -> Close(false). Never overwrite the original .rfa.

**Severity:** must

## Reloading an existing family needs the UI

> BIMNEXUS note: written with revit-mcp tool names. Use the matching BIMNEXUS tools; anything that needs custom C# (execute_csharp / send_code_to_revit) has no BIMNEXUS equivalent – do it with BIMNEXUS tools or ask the user. Every model change still needs the user's confirmation.

LoadFamily from inside the runner can fail with 'document must not be modifiable'. New family names may load by API; overwriting an existing family needs Insert > Load Family > 'Overwrite the existing version' in the UI. Verify by script afterwards.

**Severity:** must

## Curved elements are true curves, real system elements

Curved walls/upstands follow the real curve as true curved wall elements (not many short straight segments, not model in-place). For sloped curved upstands use a mass + Wall by Face; tag created elements with a Comments value so they can be selected later.

**Severity:** must
