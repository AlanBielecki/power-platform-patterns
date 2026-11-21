# power-platform-patterns

Notes and snippets I keep coming back to when building Power Apps, Power Automate flows and Copilot Studio agents on top of SharePoint.

Most of these came out of replacing Excel-based processes with apps: forms that people actually fill in correctly, data that doesn't get overwritten by accident, and lists that keep working after they pass 2,000 items. The code is generic (no company data), with made-up list and column names.

## Contents

**Power Apps**
- [Form validation and saving with feedback](power-apps/form-validation.md)
- [Updating a record without wiping existing values](power-apps/keep-existing-values.md)
- [Delegation-friendly filters for SharePoint](power-apps/delegation.md)

**Power Automate**
- [Excel to SharePoint sync with data quality checks](power-automate/excel-to-sharepoint-sync.md)

**Copilot Studio**
- [Agent design notes: scope, data and human review](copilot-studio/agent-design-notes.md)

## Conventions used

- Controls: `txt` text input, `dd` dropdown, `cmb` combo box, `gal` gallery, `btn` button, `frm` form
- Variables: `var` global, `loc` context, `col` collection
- SharePoint list in the examples: `Controls` with columns `Title`, `Owner` (person), `Status` (choice), `Hours` (number), `Comment` (text), `DueDate` (date)
