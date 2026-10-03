# 15. Security, performance, and debugging

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: State saving and accessibility](./14-state-saving-and-accessibility.md) | [Notes index](../README.md) | [Next: Integrating and running in production](./16-integration-and-production-patterns.md) |

## Treat table data as untrusted

Rows can come from a database, an API, a file, or user input. A value that looks like text can contain HTML. If a renderer inserts that value as markup, it can run in the browser.

Use the built-in text renderer for untrusted string columns:

~~~js
const customersTable = new DataTable("#customersTable", {
    data: customers,
    columns: [
        {
            data: "name",
            render: DataTable.render.text(),
        },
        {
            data: "email",
            render: DataTable.render.text(),
        },
    ],
});
~~~

The text renderer escapes HTML for display. It is a useful default for names, labels, and other plain text. Use a purpose-built, reviewed approach when a cell must contain a link or other markup. Do not concatenate untrusted values into HTML attributes or markup.

Encoding output protects the browser display. It does not replace server validation, database parameterization, authorization checks, or safe handling in other output contexts.

## Validate server-side table requests

In server-side mode, the browser sends paging, search, and ordering information to the server. Treat every request field as input that can be changed by a caller.

On the backend:

- Allowlist sortable column names instead of using a client-supplied name in a query.
- Validate page size, offsets, search length, and direction.
- Use parameterized database queries for values.
- Apply the signed-in user's access rules before counting or returning rows.
- Return only fields that the user is allowed to see.
- Escape untrusted values when they are later displayed.

The column index in a DataTables request is not a permission check. Keep authorization and query construction in application code on the server.

## Keep the browser work proportional to the view

DataTables creates row and cell elements when it displays data. For Ajax and JavaScript data sources, deferred rendering creates nodes as they are needed for a draw. It is enabled by default in current DataTables versions and has the most benefit when paging is active.

~~~js
const productsTable = new DataTable("#productsTable", {
    ajax: "/api/products",
    deferRender: true,
    pageLength: 25,
});
~~~

Deferred rendering reduces the number of DOM nodes created up front. It does not reduce the amount of JSON downloaded or remove all rows from browser memory. For a data set that is too large to send to the browser, use server-side processing and filter, order, and page records in the backend.

When deferred rendering is active, a row that has not been drawn may not have a DOM node yet. Prefer delegated events or DataTables events over attaching a separate listener to every row.

~~~js
const productsTable = new DataTable("#productsTable", {
    ajax: "/api/products",
});

productsTable.on("click", "tbody button[data-action='open']", function (event) {
    const row = productsTable.row(event.target.closest("tr")).data();
    openProduct(row.id);
});
~~~

The API event handler is delegated through the table, so it also works for rows created on later draws. Keep the action button's accessible label clear in the row markup.

## Avoid unnecessary work during draws

Search, ordering, paging, and data updates can cause a draw. Keep render functions fast and avoid repeating expensive formatting or network work for each cell.

Add a group of rows in one API operation and draw once:

~~~js
productsTable
    .rows.add(newProducts)
    .draw(false);
~~~

The `false` keeps the current page when the table is redrawn. Use `ajax.reload(null, false)` when refreshing Ajax data and preserving the user's page is useful.

~~~js
productsTable.ajax.reload(null, false);
~~~

If a table is still slow, measure the slow part. The cause could be a large response, an expensive database query, a custom renderer, too many DOM nodes, or an extension. Reduce unnecessary columns and payload fields, choose a suitable page length, and move large data sets to server-side processing when it fits the application.

## Initialize once and change data through the API

A common console warning is that a table cannot be reinitialized. It occurs when code calls the constructor again for a table that is already active.

~~~js
const tableElement = document.querySelector("#ordersTable");

if (!DataTable.isDataTable(tableElement)) {
    new DataTable(tableElement, {
        pageLength: 25,
    });
}
~~~

If only the rows changed, use the existing instance's API. If a configuration option must change and there is no API for it, destroy and recreate deliberately. Destroying and rebuilding a table has extra cost, so do not use it for ordinary data refreshes.

## Debug from the first failing layer

When a table is blank or behaves unexpectedly, find the first layer that is wrong:

1. Check the browser console for JavaScript errors and DataTables warnings.
2. Inspect the table HTML and confirm each body row matches the header column count.
3. For Ajax, inspect the request URL, response status, JSON shape, and field names in the Network panel.
4. Confirm the configured `dataSrc` matches the response structure.
5. Check whether the table has already been initialized.
6. Reduce custom renderers and extensions until the failing behavior is isolated.
7. Try a small known data set, then add the real request and options back one at a time.

For an Ajax response shaped like `{ "data": [...] }`, the default source is `data`. If the records are nested under another property, configure `ajax.dataSrc` to that property.

~~~js
const ordersTable = new DataTable("#ordersTable", {
    ajax: {
        url: "/api/orders",
        dataSrc: "results.orders",
    },
});
~~~

Use `dataSrc` that matches the actual response, not the shape you expect the server to return. A successful HTTP status can still contain invalid JSON or the wrong property.

## Protect diagnostic data

Console messages and network responses often include customer names, account numbers, or tokens. Remove private values before sharing a screenshot, request, or diagnostic report. Keep authentication tokens out of URLs and logs.

Do not expose production records while debugging. Reproduce a problem with a small sanitized response whenever possible.

## Security, performance, and debugging checklist

- Render untrusted plain text with `DataTable.render.text()`.
- Keep authorization, validation, and safe query construction on the server.
- Use deferred rendering and paging for browser-side data where appropriate.
- Use server-side processing when the browser should not receive the complete data set.
- Delegate handlers for rows created after initialization.
- Update data through the API and draw once after a batch.
- Initialize each table once and use its API for routine updates.
- Inspect the console and network response before changing several options at once.
- Remove private data from diagnostics before sharing them.

## Further reading

- [DataTables security manual](https://datatables.net/manual/core/security)
- [Data renderers](https://datatables.net/manual/core/data/renderers)
- [Deferred rendering](https://datatables.net/reference/option/deferRender)
- [Check whether a table is initialized](https://datatables.net/reference/api/DataTable.isDataTable%28%29)
- [Cannot reinitialize warning](https://datatables.net/manual/tech-notes/3)
- [Server-side processing](https://datatables.net/manual/server-side)