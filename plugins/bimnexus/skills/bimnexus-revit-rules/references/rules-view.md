# Views and templates

## Create STD templates from a view, not by duplicating the template

View.Duplicate fails on templates and CopyElements loses Custom link overrides. Create the new template with CreateViewTemplate() from a view that uses the source template, then copy the non-controlled parameter ids and verify the link override count.

**Severity:** must

## Filter order: the top filter wins

In View.GetOrderedFilters() the first filter wins. To reorder, remove and re-add filters (each re-added filter goes to the bottom).

**Severity:** must

## Link display limits of the API

The API cannot set RVT-link display to Custom or edit a link's colour scheme / Custom display settings; these are UI steps for the user (or computer use with permission). Switching links to 'By Host View' can make linked content disappear - test on one view first.

**Severity:** should
