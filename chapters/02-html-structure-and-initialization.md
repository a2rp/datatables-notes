# 02. HTML table structure and initialization

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Getting started and setup](./01-getting-started-and-setup.md) | [Notes index](../README.md) | [Next: Data sources](./03-data-sources.md) |

## Start with a real HTML table

DataTables enhances a table element. The structure should already describe the information clearly. Use a caption, a header row, and a body so users can understand the data before any search, order, or paging controls are added.

~~~html
<table id="ordersTable">
    <caption>Recent customer orders</caption>
    <thead>
        <tr>
            <th scope="col">Order</th>
            <th scope="col">Customer</th>
            <th scope="col">Status</th>
            <th scope="col">Total</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>1048</td>
            <td>Ravi Shah</td>
            <td>Shipped</td>
            <td>125.50</td>
        </tr>
        <tr>
            <td>1049</td>
            <td>Meera Das</td>
            <td>Processing</td>
            <td>84.00</td>
        </tr>
    </tbody>
</table>
~~~

The table ID is the selector used by the constructor. IDs must be unique in a page. A class selector is useful when several tables share the same configuration, but be intentional about initializing every match.

## Understand each table section

- `caption` names the subject of the table.
- `thead` groups the header rows.
- `tbody` groups the data rows.
- `tfoot` can provide footer labels or summary cells.

Use `th` for header cells and `td` for ordinary data cells. For a simple table, `scope="col"` tells assistive technology that a header describes its column. Complex headers may need a deliberate structure and explicit header associations.

A table is a two-dimensional data structure. Keep each body row aligned with the same set of columns. Do not put `colspan` or `rowspan` in the body rows used by DataTables. Header and footer cells can use spans, but each actual data column still needs a clear header.

## Initialize a table with DOM data

For an HTML table that already has rows, a small constructor is enough:

~~~js
const ordersTable = new DataTable("#ordersTable");
~~~

DataTables reads the rows and builds its controls. The table API can be stored when code needs to search or update it later:

~~~js
const ordersTable = new DataTable("#ordersTable", {
    pageLength: 10,
    order: [[0, "desc"]],
});

ordersTable.search("Processing").draw();
~~~

Use zero-based column indexes in options. Index 0 means the first column. In this example, the order number sorts descending, and the search is applied across searchable columns.

## Configure columns deliberately

The `columns` option provides configuration for every column in the table. If it is supplied, its array must match the number of header columns.

~~~js
const ordersTable = new DataTable("#ordersTable", {
    columns: [
        { name: "order", orderable: true },
        { name: "customer", searchable: true },
        { name: "status", searchable: true },
        { name: "total", className: "numeric-cell" },
    ],
});
~~~

A column can have a stable name for application code, a class for styling, and flags to control whether users can search or order it. Configure only the behavior that differs from the defaults.

Use `columnDefs` when a rule applies to selected column indexes:

~~~js
const ordersTable = new DataTable("#ordersTable", {
    columnDefs: [
        {
            targets: [2],
            orderable: false,
        },
        {
            targets: [3],
            className: "numeric-cell",
        },
    ],
});
~~~

Avoid defining the same property in both `columns` and `columnDefs` without understanding which definition has priority. Keep column configuration in one place when the mapping is simple.

## Add a footer for summaries

A footer can contain a label aligned with the table columns. It is also a useful place for calculated totals or column filters, but those controls should be created after the table is initialized.

~~~html
<table id="salesTable">
    <caption>Sales by product</caption>
    <thead>
        <tr>
            <th scope="col">Product</th>
            <th scope="col">Units</th>
            <th scope="col">Revenue</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Notebook</td>
            <td>12</td>
            <td>96.00</td>
        </tr>
        <tr>
            <td>Pen set</td>
            <td>8</td>
            <td>40.00</td>
        </tr>
    </tbody>
    <tfoot>
        <tr>
            <th scope="row">Visible total</th>
            <td></td>
            <td></td>
        </tr>
    </tfoot>
</table>
~~~

The footer cells should line up with the columns. DataTables supports footer cells with spans, but simple one-cell-per-column footers are easier to work with and announce.

## Work with grouped headers

A multi-row header can group related columns. The leaf header cells still define the underlying data columns.

~~~html
<table id="teamTable">
    <caption>Team member contact details</caption>
    <thead>
        <tr>
            <th rowspan="2" scope="col">Name</th>
            <th colspan="2" scope="colgroup">Contact</th>
        </tr>
        <tr>
            <th scope="col">Email</th>
            <th scope="col">Office</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Ada Lovelace</td>
            <td>ada@example.test</td>
            <td>London</td>
        </tr>
    </tbody>
</table>
~~~

For complex table headers, check the browser accessibility tree and test how header associations are announced. The visual grouping alone does not explain every relationship to assistive technology.

## Keep initialization in one place

Create one function or module responsible for initializing the table. This prevents duplicate initialization when a page component mounts more than once.

~~~js
function createOrdersTable() {
    const element = document.querySelector("#ordersTable");

    if (!element) {
        return null;
    }

    return new DataTable(element, {
        pageLength: 10,
    });
}

const ordersTable = createOrdersTable();
~~~

Do not call the constructor for a selector that may already have an active instance. When the application only needs to retrieve an existing instance, use the version's documented retrieval method instead of constructing another one.

## Common structure problems

- **Header and row cells do not line up:** count the leaf header cells and compare each body row.
- **A body row spans multiple columns:** remove the span and represent the information in ordinary cells.
- **The wrong field is ordered:** check the zero-based column index and the actual header order.
- **A caption is missing:** add a concise caption that explains the table's subject.
- **Headers announce poorly:** verify scope and header associations with assistive technology.
- **A warning says the table is already initialized:** centralize the constructor call and retain the instance.
- **A styling selector affects another table:** scope it to a unique table or component class.

## Structure checklist

- There is one unique selector for each separately configured table.
- The caption and headers describe the data.
- Header cells use `th` and body values use `td`.
- Each body row has a consistent number of cells.
- Column options line up with the actual leaf columns.
- Initialization happens once after the table element exists.
- Grouped headers are checked for accessible header relationships.

## Further reading

- [DataTables HTML table structure](https://datatables.net/manual/tech-notes/4)
- [DataTables columns option](https://datatables.net/reference/option/columns)
- [DataTables columnDefs option](https://datatables.net/reference/option/columnDefs)
- [MDN: table element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/table)
- [MDN: table header cell](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/th)
