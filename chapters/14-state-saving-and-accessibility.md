# 14. State saving and accessibility

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Buttons, export, and layout](./13-buttons-export-and-layout.md) | [Notes index](../README.md) | [Next: Security, performance, and debugging](./15-security-performance-and-debugging.md) |

## Save the state users need

State saving can restore a table's page, page length, ordering, search, and column visibility after a reload. Enable it with `stateSave`:

~~~js
const ordersTable = new DataTable("#ordersTable", {
    stateSave: true,
});
~~~

By default, DataTables keeps state for two hours in `localStorage`. Set `stateDuration` in seconds to choose a different lifetime. A value of `0` keeps state without an expiry, while `-1` uses `sessionStorage` for the current browser session.

~~~js
const ordersTable = new DataTable("#ordersTable", {
    stateSave: true,
    stateDuration: 60 * 60 * 24,
});
~~~

The example keeps state for one day. Use a shorter period for a shared computer, or use `-1` when state should disappear with the browser session. State is scoped to the browser, so it does not automatically follow a user to another device.

## Keep private search terms out of saved state

Search text can contain names, account identifiers, or other details users did not expect to persist. Clear it before DataTables stores its state:

~~~js
const ordersTable = new DataTable("#ordersTable", {
    stateSave: true,
    stateDuration: -1,

    stateSaveParams(settings, data) {
        data.search.search = "";

        for (const column of data.columns) {
            column.search.search = "";
        }
    },
});
~~~

This example leaves paging, ordering, and column visibility available while omitting the global and per-column search terms. If an application has different privacy needs, decide explicitly which fields may be saved.

Browser storage is not an authentication or authorization boundary. Never use saved table state to decide which rows a user may access. Apply access rules on the server for every request.

## Change where state is stored when needed

The built-in storage is useful for a single browser and a non-sensitive table. An application can provide `stateSaveCallback` and `stateLoadCallback` to store state elsewhere, such as a server account setting.

Before saving state on a server, associate it with the authenticated user and the specific table. Validate loaded values, apply normal access checks, and handle an empty or expired response. Do not accept a client-submitted state object as proof that a user can view records.

## Give the table a meaningful structure

A table caption explains what the data represents. Column headers identify each field, and row headers identify a record when that relationship is useful.

~~~html
<table id="ordersTable">
    <caption>Recent orders</caption>
    <thead>
        <tr>
            <th scope="col">Order number</th>
            <th scope="col">Customer</th>
            <th scope="col">Status</th>
            <th scope="col">Total</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <th scope="row">ORD-1042</th>
            <td>Riya Shah</td>
            <td>Processing</td>
            <td>Rs 2,450</td>
        </tr>
    </tbody>
</table>
~~~

Use real table elements for tabular data. Avoid replacing table semantics with generic `div` elements or adding ARIA roles that conflict with the native table structure. If the caption is visually hidden, use a tested visually-hidden style instead of removing it from the accessibility tree.

## Make controls understandable without a mouse

DataTables adds keyboard focus to its search, ordering, paging, and other controls by default. Its `tabIndex` option defaults to `0`. Keep the controls in the normal tab order unless testing shows a specific reason to change it.

~~~js
const ordersTable = new DataTable("#ordersTable", {
    tabIndex: 0,
    language: {
        search: "Search orders:",
        emptyTable: "No orders are available.",
        zeroRecords: "No orders match the current search.",
    },
});
~~~

Use Tab to move through controls and Enter or Space to activate them. Check that focus is visible against the page background. Do not add every table cell to the tab order; that can make keyboard navigation long and difficult.

~~~css
.dt-container :focus-visible {
    outline: 3px solid #ff6b35;
    outline-offset: 2px;
}
~~~

The example sets a visible focus outline for controls inside the DataTables container. Adapt the color to the site's contrast requirements, and do not remove the browser outline without providing another clear indicator.

## Label custom filters

If the page uses an application-owned filter, give it a visible label and connect the label to the input. Placeholder text alone is not a label.

~~~html
<label for="ordersCustomerFilter">Filter by customer</label>
<input id="ordersCustomerFilter" type="search">
~~~

~~~js
const ordersTable = new DataTable("#ordersTable");

document
    .querySelector("#ordersCustomerFilter")
    .addEventListener("input", (event) => {
        ordersTable
            .column(1)
            .search(event.currentTarget.value)
            .draw();
    });
~~~

If a table redraw changes an important result count, provide a concise status message. Do not make the entire table a live region, since that can cause large amounts of content to be announced repeatedly.

~~~html
<p id="ordersStatus" role="status" aria-live="polite"></p>
~~~

~~~js
const ordersTable = new DataTable("#ordersTable");
const ordersStatus = document.querySelector("#ordersStatus");

function updateOrdersStatus() {
    const page = ordersTable.page.info();
    ordersStatus.textContent =
        `${page.recordsDisplay} matching orders.`;
}

ordersTable.on("draw", updateOrdersStatus);
updateOrdersStatus();
~~~

A polite status is useful when the result count changes and the built-in information text does not meet the page's needs. Avoid announcing the same update through two different live regions.

## Check responsive and extension behavior

Responsive details, fixed headers, scrolling, column visibility, and custom controls can change focus order or how data is announced. Test the actual combination used by the application rather than assuming that an accessible core table guarantees every extension view is accessible.

For each table, check that:

- A screen reader announces the caption and the correct row and column headers.
- Search, sorting, paging, and custom filters have clear names.
- Every control works using only the keyboard, with a visible focus indicator.
- Sorting changes are understandable without relying on color or an icon alone.
- Hidden responsive values remain available and associated with their labels.
- Focus remains predictable after searching, paging, and redrawing the table.
- Empty, loading, and error states are explained in text.
- The table remains usable at high zoom and on a narrow screen.

Use a keyboard pass and at least one screen reader supported by the application. Include people who rely on assistive technology in usability checks when possible.

## State and accessibility checklist

- Save only state that helps the user return to their work.
- Choose a storage lifetime that fits the device and data.
- Clear search values before saving when they may contain private details.
- Enforce record access on the server, independently of browser state.
- Use a caption and correctly scoped headers.
- Keep keyboard focus visible and controls named.
- Test the exact responsive and extension configuration with assistive technology.

## Further reading

- [State saving](https://datatables.net/reference/option/stateSave)
- [State duration](https://datatables.net/reference/option/stateDuration)
- [State save parameters](https://datatables.net/reference/option/stateSaveParams)
- [Keyboard tab index](https://datatables.net/reference/option/tabIndex)
- [DataTables manual](https://datatables.net/manual/)
- [HTML table accessibility (MDN)](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Structuring_content/Table_accessibility)
