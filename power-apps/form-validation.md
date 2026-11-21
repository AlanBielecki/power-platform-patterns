# Form validation and saving with feedback

Default forms let people submit half-empty records and fail silently when SharePoint rejects something. This is the pattern I use instead: validate first, save with `Patch`, then check `Errors()` before telling the user it worked.

## Disable the button until required fields are filled

`btnSave.DisplayMode`

```powerfx
If(
    IsBlank(Trim(txtTitle.Text)) || IsBlank(cmbOwner.Selected) || IsBlank(ddStatus.Selected),
    DisplayMode.Disabled,
    DisplayMode.Edit
)
```

It's worth also showing *why* the button is disabled. A label under the form:

`lblMissing.Text`

```powerfx
With(
    {
        missingFields: Concat(
            Filter(
                Table(
                    { field: "Title", missing: IsBlank(Trim(txtTitle.Text)) },
                    { field: "Owner", missing: IsBlank(cmbOwner.Selected) },
                    { field: "Status", missing: IsBlank(ddStatus.Selected) }
                ),
                missing
            ),
            field,
            ", "
        )
    },
    If(IsBlank(missingFields), "", "Required: " & missingFields)
)
```

## Save and check the result

`btnSave.OnSelect`

```powerfx
Patch(
    Controls,
    Defaults(Controls),
    {
        Title: Trim(txtTitle.Text),
        Owner: {
            '@odata.type': "#Microsoft.Azure.Connectors.SharePoint.SPListExpandedUser",
            Claims: "i:0#.f|membership|" & cmbOwner.Selected.Mail,
            DisplayName: cmbOwner.Selected.DisplayName,
            Email: cmbOwner.Selected.Mail,
            Department: "",
            JobTitle: "",
            Picture: ""
        },
        Status: { Value: ddStatus.Selected.Value },
        DueDate: dpDue.SelectedDate
    }
);
If(
    IsEmpty(Errors(Controls)),
    Notify("Saved", NotificationType.Success);
    Reset(txtTitle);
    Reset(cmbOwner),
    Notify(
        "Not saved: " & First(Errors(Controls)).Message,
        NotificationType.Error
    )
)
```

Notes:

- `cmbOwner` here uses the Office 365 Users connector (`Office365Users.SearchUser`), which is why the person record has to be built by hand.
- `Trim()` on text inputs saves a surprising amount of cleanup later. Trailing spaces are one of the main reasons the same value ends up in a report twice.
- Checking `Errors()` matters most for permission problems: users who can open the app but don't have contribute rights on the list.
