# 09. Events and lifecycle

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: The API and table updates](./08-api-and-table-updates.md) | [Notes index](../README.md) | [Next: Ajax loading and JSON](./10-ajax-loading-and-json.md) |

## Why events matter

DataTables events let application code respond to table changes such as ordering, searching, paging, drawing, Ajax requests, and destruction. Use events to update related interface elements or connect application behavior to the table lifecycle.

Do not use a draw event to perform work that can be done once during setup. A draw can happen after search, ordering, paging, data updates, or Ajax completion.

## Attach listeners during initialization

The `on` option registers listeners before initialization events occur. It is useful when an Ajax response or first draw must be observed.

~~~js
const staffTable = new DataTable("#staffTable", {
    ajax: "/api/staff",
    on: {
        xhr(event, settings, json) {
            console.log("Staff response received", json);
        },
        draw() {
            console.log("Staff rows redrawn");
        },
    },
});
~~~

The option uses event names as object keys. DataTables 3 removes these configured listeners when the table is destroyed.

## Listen through the API

The API `on()` method attaches a listener to an existing table:

~~~js
const staffTable = new DataTable("#staffTable");

function reportDraw() {
    const info = staffTable.page.info();
    console.log("Visible rows:", info.end - info.start);
}

staffTable.on("draw", reportDraw);
~~~

A listener added after synchronous initialization does not see the initial draw. Use the initialization `on` option when the first draw matters.

Remove a listener when its work is no longer needed:

~~~js
staffTable.off("draw", reportDraw);
~~~

Keep a reference to the function when it must later be removed. Use `one()` for a handler that should run only for the next matching event.

## Use the right event for the work

- **init:** initialization and configured data loading have completed.
- **draw:** the table has redrawn its visible rows.
- **order:** the order has changed.
- **search:** the search state has changed.
- **page:** the current page has changed.
- **length:** the page length has changed.
- **xhr:** an Ajax response has been received.
- **destroy:** the table is being destroyed.

An event signals that an operation occurred. Use the API to read the resulting search, page, or order state instead of trying to reconstruct it from click events.

## Update a separate status element

A draw listener can keep a custom page summary current:

~~~html
<p id="pageSummary" aria-live="polite"></p>
~~~

~~~js
const pageSummary = document.querySelector("#pageSummary");

function updatePageSummary() {
    const { page, pages, recordsDisplay } = staffTable.page.info();

    pageSummary.textContent = recordsDisplay === 0
        ? "No matching records"
        : "Page " + (page + 1) + " of " + pages;
}

staffTable.on("draw", updatePageSummary);
updatePageSummary();
~~~

The live region should announce useful changes without reading the entire table after every redraw. Keep updates short and avoid moving focus just because the result count changed.

## Delegate events from table controls

DataTables can replace row elements while drawing. A listener attached directly to a button inside one row may disappear when that row is replaced. Event delegation attaches one listener to a stable ancestor and checks which control was activated.

~~~html
<tbody id="staffRows">
    <tr id="staff-104">
        <td>Ada Lovelace</td>
        <td>Engineering</td>
        <td><button type="button" data-action="open">Open</button></td>
    </tr>
</tbody>
~~~

~~~js
const staffRows = document.querySelector("#staffRows");

staffRows.addEventListener("click", (event) => {
    if (!(event.target instanceof Element)) {
        return;
    }

    const button = event.target.closest("[data-action='open']");
    if (!button) {
        return;
    }

    const row = staffTable.row(button.closest("tr"));
    const record = row.data();

    console.log("Open staff record:", record.id);
});
~~~

Delegation keeps working as row elements are redrawn. If an application uses a generated control, build it with DOM methods or safe renderers and use a stable data attribute to identify the action.

## Observe Ajax lifecycle events

Ajax-backed tables can report when a request starts, completes, or fails. Use initialization listeners for the first request and API listeners for later reloads.

~~~js
const auditTable = new DataTable("#auditTable", {
    ajax: "/api/audit",
    on: {
        preXhr() {
            document.querySelector("#loading").hidden = false;
        },
        xhr(event, settings, json) {
            document.querySelector("#loading").hidden = true;

            if (!json) {
                document.querySelector("#loadError").hidden = false;
            }
        },
    },
});
~~~

Keep loading and error messaging in the page. A failed request should not leave users guessing whether the table is empty or still loading. Avoid exposing internal server errors in a public message.

## Avoid work that triggers another draw

A draw listener should usually update an external element, not change the table's data or search. If it changes a table setting and calls draw again, it can create a repeated draw loop.

~~~js
staffTable.on("draw", () => {
    const visibleCount = staffTable.rows({ page: "current" }).count();
    document.querySelector("#visibleCount").textContent =
        String(visibleCount);
});
~~~

Keep event handlers quick. For expensive work, profile the interaction and consider whether the update can be performed less often or on a more specific event.

## Clean up when removing a table

Framework components and single-page navigation can mount and remove table views repeatedly. Keep event handlers inside the owning component and remove application listeners before destroying the table.

~~~js
function mountStaffTable(element) {
    const table = new DataTable(element, {
        pageLength: 10,
    });

    const onDraw = () => {
        console.log("Current page:", table.page.info().page + 1);
    };

    table.on("draw", onDraw);

    return function unmountStaffTable() {
        table.off("draw", onDraw);
        table.destroy();
    };
}
~~~

DataTables 3 removes listeners provided through the initialization `on` option on destroy. Listeners attached later through the API should still be cleaned up by the code that owns them.

## Lifecycle checklist

- Register a listener before initialization when the first event matters.
- Use the event that matches the operation instead of guessing from clicks.
- Keep draw handlers short and prevent redraw loops.
- Delegate events for controls inside rows that DataTables redraws.
- Show loading and error states for Ajax-backed tables.
- Remove application listeners and destroy instances when their view is removed.

## Further reading

- [DataTables events manual](https://datatables.net/manual/core/events)
- [DataTables on API](https://datatables.net/reference/api/on())
- [DataTables initialization on option](https://datatables.net/reference/option/on)
- [DataTables draw event](https://datatables.net/reference/event/draw)
- [DataTables Ajax event](https://datatables.net/reference/event/xhr)
- [DataTables destroy API](https://datatables.net/reference/api/destroy())
