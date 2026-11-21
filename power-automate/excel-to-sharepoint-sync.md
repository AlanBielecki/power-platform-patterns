# Excel to SharePoint sync with data quality checks

A flow that takes rows from an Excel table (exported from another system) and creates or updates items in a SharePoint list. The goal was to make the import repeatable and safe to run twice, without overwriting good data with empty cells.

## Flow outline

```
Recurrence / manual trigger
└─ List rows present in a table            (Excel Online, table: tblImport)
└─ Filter array: valid rows                (key present, numbers parse)
└─ Filter array: rejected rows             (everything else)
└─ Apply to each valid row                 (concurrency: 1)
   ├─ Get items                            (SharePoint, filter on key, top 1)
   └─ Condition: item exists?
      ├─ yes → Update item                 (keep existing values when cell is empty)
      └─ no  → Create item
└─ Create HTML table from rejected rows
└─ Send an email (V2) to the process owner (only if rejected rows exist)
```

## Valid rows

Filter array, `From`: `body('List_rows_present_in_a_table')?['value']`

Advanced mode condition:

```
@and(
    not(empty(trim(coalesce(item()?['ControlID'], '')))),
    or(
        empty(item()?['Hours']),
        isFloat(string(item()?['Hours']))
    )
)
```

The rejected-rows filter is the same expression wrapped in `not(...)`. Keeping both means nothing gets lost silently: every row either goes in or ends up in the email.

## Look up the existing item

Get items, Filter Query:

```
ControlID eq '@{trim(items('Apply_to_each')?['ControlID'])}'
```

Top Count: `1`. Index the `ControlID` column in SharePoint so this stays fast on big lists.

Condition:

```
@greater(length(body('Get_items')?['value']), 0)
```

## Update without overwriting

In Update item, each field uses the Excel value only if it's not empty:

Hours:

```
@{if(
    empty(items('Apply_to_each')?['Hours']),
    first(body('Get_items')?['value'])?['Hours'],
    float(items('Apply_to_each')?['Hours'])
)}
```

Comment:

```
@{coalesce(
    if(empty(trim(coalesce(items('Apply_to_each')?['Comment'], ''))), null, trim(items('Apply_to_each')?['Comment'])),
    first(body('Get_items')?['value'])?['Comment']
)}
```

## Things that bit me

- **Concurrency.** With parallel branches on, two rows with the same key can both decide "item doesn't exist" and create duplicates. Concurrency control on the loop set to 1 fixes it.
- **Excel dates** come in as serial numbers (e.g. `45672`) unless the action's DateTime Format is set to ISO 8601.
- **Trailing spaces in keys.** `trim()` on both sides of every lookup.
- **Run twice to test.** A sync is only correct if the second run with the same file changes nothing.
