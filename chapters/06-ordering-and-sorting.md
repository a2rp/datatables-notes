# 06. Ordering and custom sorting

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Searching and filtering](./05-searching-and-filtering.md) | [Notes index](../README.md) | [Next: Paging and page length](./07-paging-and-page-length.md) |

## Set the initial order

The `order` option accepts column indexes and directions. Indexes start at zero, so the first column is 0. An order can include more than one column to make ties predictable.

~~~js
const ordersTable = new DataTable("#ordersTable", {
    order: [
        [2, "desc"],
        [0, "asc"],
    ],
});
~~~

This sorts by the third column descending, then by the first column ascending when values in the third column match. Explicit ordering is helpful when a table should open in a meaningful state.

## Change ordering through the API

The API queues a new order. Draw the table to apply it:

~~~js
ordersTable.order([[1, "asc"]]).draw();
~~~

Read the current ordering with `order()` when an application needs to reflect or restore it. Avoid writing to table cell markup to fake an order change. Use the API so the displayed rows, search cache, and order state agree.

## Configure user ordering

Users can activate ordering by selecting a column header. Disable it for a column that has no useful order, such as a row of action buttons:

~~~js
const ordersTable = new DataTable("#ordersTable", {
    columns: [
        { data: "orderNumber" },
        { data: "customer" },
        { data: "status" },
        { data: "actions", orderable: false, searchable: false },
    ],
});
~~~

A non-orderable column can still be ordered through the API if the application explicitly requests it. The setting controls the user's header interaction, not a security rule.

To change the direction cycle, configure `orderSequence` for a column:

~~~js
{
    data: "customer",
    orderSequence: ["asc", "desc"],
}
~~~

Keep the cycle simple. A third direction can clear ordering, but it may not be obvious unless the interface explains the state.

## Use multi-column ordering

A list sorted by family name should usually use given name as a tie-breaker. Multi-column ordering can be set initially or requested from code:

~~~js
const peopleTable = new DataTable("#peopleTable", {
    order: [
        [1, "asc"],
        [0, "asc"],
    ],
});
~~~

Here the second column is the primary order and the first column resolves ties. DataTables can also allow users to add another ordering column through the header interaction. Test the chosen interaction with a keyboard and make the resulting state visible.

## Sort the underlying value, not its display label

A label may not sort in the way users expect. For a DOM table, the `data-order` attribute can provide a machine value while the cell displays readable text:

~~~html
<table id="priorityTable">
    <thead>
        <tr>
            <th scope="col">Task</th>
            <th scope="col">Priority</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Update records</td>
            <td data-order="3">Medium</td>
        </tr>
        <tr>
            <td>Resolve outage</td>
            <td data-order="1">High</td>
        </tr>
        <tr>
            <td>Review copy</td>
            <td data-order="4">Low</td>
        </tr>
    </tbody>
</table>
~~~

~~~js
const priorityTable = new DataTable("#priorityTable", {
    order: [[1, "asc"]],
});
~~~

The numeric values determine the order while the words remain visible. Keep the machine values consistent with the intended ranking. For object data, use a renderer that returns the rank for ordering and the label for display, as shown in chapter 4.

## Order an action column by its related data

If an action column contains buttons, users rarely need to order it. Disable ordering on that column and keep the underlying data fields in their own columns.

~~~js
const accountTable = new DataTable("#accountTable", {
    columns: [
        { data: "accountNumber" },
        { data: "owner" },
        { data: "balance" },
        {
            data: "accountNumber",
            orderable: false,
            searchable: false,
            render(data, type) {
                return type === "display" ? "Open account" : data;
            },
        },
    ],
});
~~~

For production controls, create safe links or buttons from DOM nodes and attach actions using event delegation. This example shows the column setting, not a complete account action.

## Keep formatted values numeric

Currency symbols and grouping separators are for people. Store a number and format it for display so ordering remains numeric.

~~~js
const amountFormatter = new Intl.NumberFormat("en-US", {
    style: "currency",
    currency: "USD",
});

const invoiceTable = new DataTable("#invoiceTable", {
    columns: [
        { data: "invoiceNumber" },
        { data: "customer" },
        {
            data: "amount",
            render(data, type) {
                return type === "display"
                    ? amountFormatter.format(data)
                    : data;
            },
        },
    ],
});
~~~

Avoid sorting values such as "$1,200" as plain text. If the source provides an ISO date, timestamp, or numeric rank, retain that value for the sort operation and format only the display.

## Handle custom values

Before writing a custom ordering plug-in, check whether a built-in type, a renderer, or an orthogonal value solves the problem. Most table columns can be ordered by returning a number or consistently formatted string for the sort operation.

For irregular values, define a clear comparison rule. For example, status labels may need an explicit rank rather than alphabetical order. Keep that rank close to the source data or derive it in a small, tested function.

Ordering should not depend on color, icon appearance, or the current page. The result should remain meaningful when styling is disabled.

## Ordering checklist

- The initial order reflects a useful default view.
- Indexes point to the intended visible columns.
- Ties use an explicit secondary order where needed.
- Action columns do not offer meaningless header sorting.
- Formatted numbers and dates retain machine values for ordering.
- Search and ordering use the intended rendered data types.
- Users can tell which column and direction are active.

## Further reading

- [DataTables order option](https://datatables.net/reference/option/order)
- [DataTables order API](https://datatables.net/reference/api/order())
- [DataTables columns.orderable option](https://datatables.net/reference/option/columns.orderable)
- [DataTables orderSequence option](https://datatables.net/reference/option/columns.orderSequence)
- [DataTables orthogonal data](https://datatables.net/manual/core/data/orthogonal-data)
