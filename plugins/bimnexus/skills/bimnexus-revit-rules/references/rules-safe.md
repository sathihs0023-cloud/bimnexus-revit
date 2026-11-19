# Model safety

## Every write gets a separate read-back

> BIMNEXUS note: written with revit-mcp tool names. Use the matching BIMNEXUS tools; anything that needs custom C# (execute_csharp / send_code_to_revit) has no BIMNEXUS equivalent – do it with BIMNEXUS tools or ask the user. Every model change still needs the user's confirmation.

After every write call, run a separate read-only call that re-measures the changed elements. Never report work as done without that read-back; report rolled-back or skipped items with counts.

**Why:** The runner can report success while Revit rolls the transaction back at commit.

**Severity:** must

## Known silent-rollback triggers

> BIMNEXUS note: written with revit-mcp tool names. Use the matching BIMNEXUS tools; anything that needs custom C# (execute_csharp / send_code_to_revit) has no BIMNEXUS equivalent – do it with BIMNEXUS tools or ask the user. Every model change still needs the user's confirmation.

Expect silent rollback when touching: elements inside groups, system types (e.g. 'Linear Dimension Style'), leaderless tags placed outside their room, read-only parameters, elements checked out by another user, or while a dialog is open in Revit. When changes don't stick, attach a one-shot FailuresProcessing logger in the same call and read the failure text from the log file.

**Severity:** must

## After a timeout: PING, then fresh read-back

Calls time out at ~60 s. A timed-out run may roll back OR still commit, and the next call can return the stale result of the timed-out one. Send `return "PING-n";` until the matching reply returns, then re-measure before any further write. Never push the next batch blind.

**Severity:** must

## Revit unresponsive: stop and ask about dialogs

If Revit does not answer for more than ~3 minutes or results contradict the last write, stop writing and ask the user to check for an open dialog. Do not ask them to press Ctrl+Z; cancel only that dialog.

**Severity:** must

## Final sweep for silently reverted batches

One Ctrl+Z reverts a whole script run, and a batch can roll back after returning OK. Finish every bulk job with a full count/sweep of the old state (e.g. old-format names) and re-apply missing ids one by one.

**Severity:** must

## Elements owned by another user

In workshared models, edits to elements checked out by a colleague fail. Report them with the owner's name and ask the user to have them synced/relinquished; never force.

**Severity:** should
