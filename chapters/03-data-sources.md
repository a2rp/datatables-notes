# 03. Data sources: DOM, arrays, and objects

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: HTML table structure and initialization](./02-html-structure-and-initialization.md) | [Notes index](../README.md) | [Next: Columns, rendering, and data types](./04-columns-rendering-and-data-types.md) |

## Pick a data source that matches the page

DataTables can read rows already present in the HTML, accept rows from a JavaScript array, or load data from an Ajax endpoint. Choose one source of truth for a table. Mixing HTML rows with an initial data array can make it unclear which records should appear.

- Use **DOM data** when the server or page already rendered a small, complete set of rows.
- Use **array data** when JavaScript already holds the records.
- Use **Ajax data** when the browser should request records from an API.
- Use **server-side processing** when searching and paging must happen against a large data set on the server. That request flow is covered in chapter 11.

## Read rows from the DOM

When a table already has body rows, the constructor can read them directly:

~~~html
<table id="peopleTable">
    <thead>
        <tr>
            <th scope="col">Name</th>
            <th scope="col">Department</th>
            <th scope="col">Office</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Ada Lovelace</td>
            <td>Engineering</td>
            <td>London</td>
        </tr>
        <tr>
            <td>Grace Hopper</td>
            <td>Research</td>
            <td>New York</td>
        </tr>
    </tbody>
</table>
~~~

~~~js
const peopleTable = new DataTable("#peopleTable");
~~~

This is straightforward for a short list rendered by the server. DataTables reads the text and markup in the cells. If a cell contains a link or other HTML, think about whether the display markup should also be used for sorting and filtering.

## Supply an array of arrays

An array row stores values in column order. This shape is compact and useful when the data is already tabular, but the meaning of each index can be less obvious in application code.

~~~html
<table id="inventoryTable">
    <thead>
        <tr>
            <th scope="col">SKU</th>
            <th scope="col">Product</th>
            <th scope="col">Quantity</th>
        </tr>
    </thead>
</table>
~~~

~~~js
const inventoryRows = [
    ["BK-104", "Notebook", 32],
    ["PN-220", "Pen set", 18],
    ["FD-510", "Folder", 41],
];

const inventoryTable = new DataTable("#inventoryTable", {
    data: inventoryRows,
});
~~~

Each row must have values in the same order as the header. If the table gains or reorders a column, positional data has to change with it. Use object rows when named fields make the relationship clearer.

## Supply an array of objects

An object row names each value. Use the `columns` option to map those properties to table columns.

~~~html
<table id="staffTable">
    <thead>
        <tr>
            <th scope="col">Name</th>
            <th scope="col">Department</th>
            <th scope="col">Office</th>
        </tr>
    </thead>
</table>
~~~

~~~js
const staff = [
    {
        name: "Ada Lovelace",
        department: "Engineering",
        office: "London",
    },
    {
        name: "Grace Hopper",
        department: "Research",
        office: "New York",
    },
];

const staffTable = new DataTable("#staffTable", {
    data: staff,
    columns: [
        { data: "name" },
        { data: "department" },
        { data: "office" },
    ],
});
~~~

Object properties make each mapping explicit. A column's `data` setting identifies the value to read from each row object. The `columns` array follows the order of the table headers.

A property can be nested when the API response groups related information:

~~~js
const staff = [
    {
        name: "Ada Lovelace",
        role: {
            title: "Engineer",
            team: "Platform",
        },
    },
];

const staffTable = new DataTable("#staffTable", {
    data: staff,
    columns: [
        { data: "name" },
        { data: "role.title" },
        { data: "role.team" },
    ],
});
~~~

Nested paths are convenient, but keep API data shapes understandable. A renderer can be a better choice when a displayed value needs calculation or formatting.

## Load rows from an Ajax endpoint

An Ajax source keeps the initial HTML small and lets the browser request data from an application endpoint.

~~~js
const staffTable = new DataTable("#staffTable", {
    ajax: "/api/staff",
    columns: [
        { data: "name" },
        { data: "department" },
        { data: "office" },
    ],
});
~~~

By default, DataTables looks for an array in the JSON response's `data` property. A response with named fields can look like this:

~~~json
{
    "data": [
        {
            "name": "Ada Lovelace",
            "department": "Engineering",
            "office": "London"
        },
        {
            "name": "Grace Hopper",
            "department": "Research",
            "office": "New York"
        }
    ]
}
~~~

If the API returns the rows under another key, set `dataSrc`:

~~~js
const staffTable = new DataTable("#staffTable", {
    ajax: {
        url: "/api/staff",
        dataSrc: "results",
    },
    columns: [
        { data: "name" },
        { data: "department" },
        { data: "office" },
    ],
});
~~~

The matching response would put the array at `results`. Check the Network panel when a table stays empty. Confirm the response status, JSON shape, field names, and any authentication requirements.

## Add data after initialization

For client-side tables, the API can replace or extend rows without rebuilding the table element:

~~~js
const staffTable = new DataTable("#staffTable", {
    data: [],
    columns: [
        { data: "name" },
        { data: "department" },
        { data: "office" },
    ],
});

function showStaff(records) {
    staffTable.clear();
    staffTable.rows.add(records);
    staffTable.draw();
}
~~~

Batch updates, then draw once. Repeatedly drawing after each row adds avoidable work and can reset user context.

For Ajax-backed data, reload from the same endpoint when the server has new records:

~~~js
staffTable.ajax.reload(null, false);
~~~

The second argument keeps the current page where practical. Choose whether to preserve paging based on the action. For example, after changing a filter, returning to the first page may be clearer.

## Keep the source and column count aligned

The table structure, row shape, and column configuration must describe the same number of columns. For object rows, every configured field should exist or have a deliberate fallback. Missing values can otherwise display as undefined or trigger a warning.

Do not initialize with one source and then insert another source into the table's body behind DataTables. Use the API for row updates so its internal ordering and search data stay synchronized with what is displayed.

## Choose between client and server data work

Client-side processing loads all rows needed by the table into the browser. It is simple and supports immediate search and ordering for modest data sets. The browser must still download, parse, and retain those rows.

Server-side processing sends paging, search, and ordering requests to an application endpoint. The server returns only the records for the current draw. It adds backend work and a defined request protocol, but avoids loading the entire data set into the browser.

Do not choose a threshold by habit. Measure response size, render time, user needs, and backend capacity.

## Data source checklist

- Each table has one clear source of truth.
- Array indexes or object fields map to the visible column order.
- The number of data fields agrees with the header and configuration.
- Ajax JSON has the array key expected by `dataSrc`.
- Empty, missing, or invalid values have intentional behavior.
- Updates use the DataTables API and are drawn in batches.
- Data size is appropriate for client-side or server-side processing.

## Further reading

- [DataTables data sources](https://datatables.net/manual/data/)
- [DataTables Ajax loading](https://datatables.net/manual/core/ajax)
- [DataTables columns.data option](https://datatables.net/reference/option/columns.data)
- [DataTables ajax.dataSrc option](https://datatables.net/reference/option/ajax.dataSrc)
