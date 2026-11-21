# Updating a record without wiping existing values

A common problem when an app or a sync writes to SharePoint: someone leaves a field empty and the save overwrites a value that was already there. In a capacity planning app this meant losing hours that had been entered earlier.

The fix is to only send values that are actually filled in, and fall back to what is already stored.

## Single record

```powerfx
With(
    { current: LookUp(Controls, ID = galControls.Selected.ID) },
    Patch(
        Controls,
        current,
        {
            Hours: Coalesce(Value(txtHours.Text), current.Hours),
            Comment: Coalesce(Trim(txtComment.Text), current.Comment),
            DueDate: Coalesce(dpDue.SelectedDate, current.DueDate)
        }
    )
)
```

`Coalesce` returns the first argument that is not blank and not an empty string, so:

| Input | Stored before | Saved |
|---|---|---|
| `12` | `8` | `12` |
| *(empty)* | `8` | `8` |
| *(empty)* | *(blank)* | *(blank)* |

`Value("")` returns blank, so an empty number field falls through to the stored value as well.

## When blank is a valid answer

Sometimes the user really wants to clear a field. Then `Coalesce` is wrong, because there is no way to tell "left empty" from "cleared on purpose". I add an explicit checkbox next to the field:

```powerfx
Comment: If(
    chkClearComment.Value,
    Blank(),
    Coalesce(Trim(txtComment.Text), current.Comment)
)
```

## Showing what will change

Before saving, a small summary label avoids surprises:

```powerfx
With(
    { current: LookUp(Controls, ID = galControls.Selected.ID) },
    If(
        !IsBlank(txtHours.Text) && Value(txtHours.Text) <> current.Hours,
        "Hours: " & current.Hours & " → " & txtHours.Text & Char(10),
        ""
    ) &
    If(
        !IsBlank(Trim(txtComment.Text)) && Trim(txtComment.Text) <> current.Comment,
        "Comment will be replaced",
        ""
    )
)
```
