# BIMNEXUS plugin for Claude Code

The `bimnexus` plugin adds these skills to Claude Code:

| Skill | What it does |
|---|---|
| bimnexus-revit-rules | 57 Revit working rules + 9 workflow guides (same as the BIMNEXUS MCP Knowledge Pack) |
| dubai-building-code-architecture / -technical / -villas | Dubai Building Code 2021 study notes for compliance checks |
| task-observer | Learns from your corrections and proposes skill updates (CC-BY-4.0, Eoghan Henn) |
| claude-automation-recommender | Recommends hooks, skills and MCP servers for a project (Apache-2.0, Anthropic) |

Install in Claude Code:

```
/plugin marketplace add sathihs0023-cloud/bimnexus-revit
/plugin install bimnexus@bimnexus
```

Update later with `/plugin marketplace update bimnexus`. The plugin works in Claude Code only; in the Claude desktop chat
use the BIMNEXUS MCP server (installed by the BIMNEXUS setup, Revit 2023–2027), which serves the same rules through
`list_knowledge` / `get_knowledge`.

Recommended (not bundled): the revit-bim skills – https://github.com/Utopia5327/revit-bim (MIT).
