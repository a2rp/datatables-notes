# 16. Integrating and running in production

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Security, performance, and debugging](./15-security-performance-and-debugging.md) | [Notes index](../README.md) | End of core notes |

## Install only the packages the table needs

DataTables 3 can run without jQuery. For an NPM project, the default styling package includes the core library:

~~~sh
npm install datatables.net-dt
~~~

Import the constructor and stylesheet from the application entry module:

~~~js
import DataTable from "datatables.net-dt";
import "datatables.net-dt/css/dataTables.dataTables.css";
~~~

Each extension is installed and imported separately. Include one only when the application uses it, and choose its matching styling package.

~~~sh
npm install datatables.net-responsive-dt
~~~

~~~js
import "datatables.net-responsive-dt";
~~~

Keep the DataTables core and extension versions compatible. Use the official package list or download builder when selecting several extensions or a styling integration.

## Initialize after the table exists

The table element must be in the document before DataTables initializes it. In a page with a single table, a module can initialize after the HTML is parsed:

~~~html
<table id="ordersTable">
    <caption>Recent orders</caption>
    <thead>
        <tr>
            <th scope="col">Order</th>
            <th scope="col">Customer</th>
            <th scope="col">Status</th>
        </tr>
    </thead>
    <tbody></tbody>
</table>
~~~

~~~js
import DataTable from "datatables.net-dt";

const ordersTable = new DataTable("#ordersTable", {
    ajax: "/api/orders",
    columns: [
        { data: "orderNumber", render: DataTable.render.text() },
        { data: "customerName", render: DataTable.render.text() },
        { data: "status", render: DataTable.render.text() },
    ],
    pageLength: 25,
});
~~~

The server response and the configured column names must agree. If the response shape is different from the default, configure `ajax.dataSrc` as covered in the Ajax chapter.

## Coordinate the table with a page lifecycle

Single-page applications can mount and remove views without a full page load. Initialize after the view inserts its table, and destroy the instance before the application removes or replaces that table element.

~~~js
let ordersTable;

export function mountOrdersTable() {
    const element = document.querySelector("#ordersTable");

    if (!element || DataTable.isDataTable(element)) {
        return;
    }

    ordersTable = new DataTable(element, {
        ajax: "/api/orders",
        pageLength: 25,
    });
}

export function unmountOrdersTable() {
    if (!ordersTable) {
        return;
    }

    ordersTable.destroy();
    ordersTable = undefined;
}
~~~

The application should have one owner for the table's rows and DOM. Avoid changing table rows directly while DataTables controls them. Send data changes through the API or reload the table, then let DataTables draw the result.

Destroy the table when its view is removed or when the table structure must be rebuilt. For a normal data refresh, keep the instance and use `ajax.reload()` or the row APIs.

## Keep shared settings small and explicit

A small factory can keep common settings consistent without hiding page-specific behavior:

~~~js
const baseTableOptions = {
    pageLength: 25,
    processing: true,
    deferRender: true,
};

export function createOrdersTable(selector) {
    return new DataTable(selector, {
        ...baseTableOptions,
        ajax: "/api/orders",
        columns: [
            { data: "orderNumber", render: DataTable.render.text() },
            { data: "customerName", render: DataTable.render.text() },
            { data: "status", render: DataTable.render.text() },
        ],
    });
}
~~~

Keep configuration near the table when the behavior differs. A shared helper should make the setup easier to understand, not make every option difficult to trace.

## Verify the complete request path

A table is only as reliable as the path from the page to its data source. Before release, check:

- The production page loads the intended JavaScript, CSS, and extension packages.
- The API route is correct for the deployment base path.
- The response status, JSON shape, and field names match the table configuration.
- Empty results and server errors produce understandable messages.
- Server-side search and order use validated fields and access checks.
- Requests do not expose tokens or private details in query strings.
- Large responses and slow queries have been measured with production-like data.
- A refresh preserves the current page only when that is useful to the user.
- Keyboard focus, responsive layout, and screen-reader output work with the production theme.
- Logs and error reports omit private table values.

Use browser developer tools to inspect the built page and the data request. A development server can hide problems with missing assets, incorrect base paths, or production-only API routes.

## Keep deployment versions repeatable

Pin dependency versions through the package manifest and lockfile, then use the project's clean installation command in its build environment. Update the lockfile when changing DataTables or an extension, and verify the generated application after the update.

Use the same DataTables styling framework for core and extensions. Mixing styling integrations can produce controls that look inconsistent or omit expected styles.

## Production checklist

- Core, theme, and extension packages are installed and compatible.
- The stylesheet is loaded once and matches the selected theme.
- Initialization occurs after the table element is present.
- Page removal cleans up the table instance and its event handlers.
- Data updates use the DataTables API.
- Untrusted text is rendered safely.
- Server-side requests are validated and authorized.
- Empty, loading, and error states are clear.
- Keyboard, zoom, narrow-screen, and assistive technology checks are complete.
- The production build and real API route have been checked.

## Further reading

- [Installation](https://datatables.net/manual/core/installation)
- [NPM packages](https://datatables.net/download/npm)
- [DataTables API](https://datatables.net/manual/core/api)
- [Ajax data source](https://datatables.net/manual/ajax)
- [Server-side processing](https://datatables.net/manual/server-side)
- [DataTables security manual](https://datatables.net/manual/core/security)