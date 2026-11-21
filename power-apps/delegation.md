# Delegation-friendly filters for SharePoint

Once a list grows past the data row limit (500 by default, 2,000 max), any filter that can't be delegated to SharePoint quietly works on the first rows only. The app keeps working in testing and starts missing records in production.

## What I stick to with SharePoint

Works on the server side:

- `=`, `<>`, `<`, `>`, `<=`, `>=` on text, number, date and choice (`Status.Value`) columns
- `StartsWith()` on text columns
- `&&` / `||` combining the above
- `Sort` / `SortByColumns` on a single column

Avoid on large lists (not delegated):

- `in` / `exactin` (e.g. `"risk" in Title`)
- `Search()`
- `Len()`, `Lower()`, `Upper()` or any function wrapped around the column itself
- `CountRows()` on the whole list

## Gallery filter

`galControls.Items`

```powerfx
SortByColumns(
    Filter(
        Controls,
        (IsBlank(ddStatus.Selected) || Status.Value = ddStatus.Selected.Value) &&
        (IsBlank(txtSearch.Text) || StartsWith(Title, txtSearch.Text)) &&
        (!chkMine.Value || Owner.Email = User().Email)
    ),
    "DueDate",
    SortOrder.Ascending
)
```

The `IsBlank(...) ||` parts make each filter optional without breaking delegation.

## Dates

Compare against a value calculated outside the filter:

```powerfx
With(
    { cutoff: DateAdd(Today(), 7, TimeUnit.Days) },
    Filter(Controls, DueDate <= cutoff && Status.Value <> "Closed")
)
```

## Checking

- Keep the delegation warnings visible in Studio and treat every yellow triangle as a bug.
- While testing, temporarily set the data row limit (Settings → General) to a small number like 5. Anything that isn't delegated will show wrong results immediately instead of only after the list grows.
