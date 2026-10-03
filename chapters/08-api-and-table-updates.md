# 08. The API and table updates

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Paging and page length](./07-paging-and-page-length.md) | [Notes index](../README.md) | [Next: Events and lifecycle](./09-events-and-lifecycle.md) |

## Keep the API instance

The constructor returns an API instance for the table. Store it when application code needs to search, navigate, read rows, or update data.

~~~js
const staffTable = new DataTable("#staffTable", {
    pageLength: 10,
});
~~~

The API has methods for tables, rows, columns, cells, and core behavior. Methods can be chained when they return the API instance. Read the reference for each method's return value, because some methods return data or DOM nodes instead.

## Read row data

Use the row or rows API to get data from the current table:

~~~js
const firstRow = staffTable.row(0).data();
const allRows = staffTable.rows().data().toArray();

console.log(firstRow);
console.log(allRows);
~~~

The row selector can use an index, node, or selector depending on what the application has available. Prefer a stable row ID for records that can move when users sort or filter.

~~~js
const selectedRecord = staffTable.row("#staff-104").data();
~~~

To inspect only rows that match the current search, use a selector modifier:

~~~js
const matchingRows = staffTable
    .rows({ search: "applied" })
    .data()
    .toArray();
~~~

This is useful for an export or a summary of the currently filtered set. Keep the user's search and page state in mind when interpreting the returned data.

## Add rows

Add one row with `row.add()` and several with `rows.add()`:

~~~js
staffTable.row.add({
    id: "staff-108",
    name: "Dorothy Vaughan",
    department: "Computing",
    office: "Hampton",
}).draw(false);
~~~

~~~js
const newRecords = [
    {
        id: "staff-109",
        name: "Annie Easley",
        department: "Computing",
        office: "Cleveland",
    },
    {
        id: "staff-110",
        name: "Katherine Johnson",
        department: "Mathematics",
        office: "Hampton",
    },
];

staffTable.rows.add(newRecords).draw(false);
~~~

Batch changes and draw once. This avoids recalculating and redrawing the table after each individual row.

## Replace or edit data

Set a row's data through the row API, then draw:

~~~js
const updatedRecord = {
    id: "staff-104",
    name: "Ada Lovelace",
    department: "Engineering",
    office: "London",
};

staffTable
    .row("#staff-104")
    .data(updatedRecord)
    .draw(false);
~~~

A cell can be changed directly when just one value changes:

~~~js
staffTable
    .cell("#staff-104", 2)
    .data("Platform")
    .draw(false);
~~~

Cell selectors can be based on a row node and column index. A row selector based on a stable ID makes the target easier to identify than a display position.

## Remove rows

Remove a row through the API so DataTables updates its data and caches:

~~~js
staffTable
    .row("#staff-104")
    .remove()
    .draw(false);
~~~

Remove several rows by selector when the application has a clear selection rule:

~~~js
staffTable
    .rows(".expired-record")
    .remove()
    .draw();
~~~

Do not remove only a table row from the DOM with a browser API while leaving DataTables unaware of the change. Use the DataTables API for records it manages.

## Replace the complete data set

For a client-side table, clear and add the new records, then draw once:

~~~js
function replaceStaff(records) {
    staffTable.clear();
    staffTable.rows.add(records);
    staffTable.draw();
}
~~~

The draw recalculates search, ordering, and paging using the new data. If the data comes from the configured Ajax source, prefer the Ajax reload API instead of making a second request yourself.

## Refresh Ajax data

Reload the current Ajax source when the server has changed records:

~~~js
staffTable.ajax.reload();
~~~

To keep the current page when possible, pass a callback and the paging reset option:

~~~js
staffTable.ajax.reload(null, false);
~~~

Reset to the first page when a previous page may no longer be meaningful, such as after changing a filter. Preserve the page when a user edits a record and should remain in the same part of the list.

## Draw only when the change is ready

Many API methods queue work. This lets the application set filters, order, and page length before a single draw.

~~~js
staffTable.search("Hampton");
staffTable.order([[0, "asc"]]);
staffTable.page.len(25);
staffTable.draw();
~~~

Choose the draw mode based on the operation. A full draw recalculates ordering and filtering. A page-only draw is for a page navigation that does not change the result set. Use a full draw after data or search conditions change.

## Destroy only when configuration must be rebuilt

The API can destroy a table, but that is a lifecycle operation. Use it when the table's column structure or initialization settings must fundamentally change, not for routine data updates.

~~~js
staffTable.destroy();

const rebuiltTable = new DataTable("#staffTable", {
    pageLength: 25,
    columns: [
        { data: "name" },
        { data: "department" },
        { data: "office" },
    ],
});
~~~

Before rebuilding, remove event listeners owned by the application and restore the original markup if needed. A normal refresh should use `clear()`, `rows.add()`, or `ajax.reload()`.

## Check for an existing instance

When an application component can mount more than once, check whether the table already has an instance before initializing it:

~~~js
const tableElement = document.querySelector("#staffTable");

if (!DataTable.isDataTable(tableElement)) {
    new DataTable(tableElement);
}
~~~

The cleaner design is usually to initialize in one lifecycle location and keep the instance there. The check is useful at integration boundaries where several scripts may own the same page.

## API update checklist

- Keep the API instance returned by the constructor.
- Identify rows by stable IDs when display order can change.
- Use API methods for every row, cell, and data update.
- Batch related changes before drawing once.
- Use selector modifiers intentionally when reading filtered rows.
- Reload the configured Ajax source for server-owned data.
- Destroy and rebuild only when configuration or columns must change.
- Remove application listeners when the table is destroyed.

## Further reading

- [DataTables API manual](https://datatables.net/manual/core/api)
- [DataTables row.add API](https://datatables.net/reference/api/row.add())
- [DataTables rows.add API](https://datatables.net/reference/api/rows.add())
- [DataTables clear API](https://datatables.net/reference/api/clear())
- [DataTables ajax.reload API](https://datatables.net/reference/api/ajax.reload())
- [DataTables isDataTable API](https://datatables.net/reference/api/DataTable.isDataTable())
