# 11. Server-side processing

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Ajax loading and JSON](./10-ajax-loading-and-json.md) | [Notes index](../README.md) | [Next: Responsive tables and extensions](./12-responsive-tables-and-extensions.md) |

## Move data work to the server

Client-side processing loads all rows into the browser, then searches, orders, and pages them locally. Server-side processing asks the endpoint for each draw of the table. It is useful when transferring and processing the full data set in the browser is too slow or too large.

With server-side processing, the application owns the query behavior. The browser sends paging, ordering, and search parameters. The endpoint validates those parameters, applies authorization, queries the data store, and returns the exact response shape DataTables expects.

## Enable server-side mode

Set `serverSide` to true and provide an Ajax endpoint. The processing indicator tells users that a request is being handled.

~~~js
const ordersTable = new DataTable("#ordersTable", {
    processing: true,
    serverSide: true,
    ajax: "/api/orders",
    columns: [
        { data: "orderNumber", name: "order_number" },
        { data: "customerName", name: "customer_name" },
        { data: "status", name: "status" },
        { data: "createdAt", name: "created_at" },
    ],
});
~~~

Each search, order, or page change can make a new request. The endpoint must apply those operations to the full authorized result set, then return only the requested rows.

## Understand the request

The request includes a draw counter, a starting row, and a page length. It also includes global search text, ordering instructions, and column metadata.

- `draw` identifies the sequence of the request and response.
- `start` is the zero-based offset of the first requested row.
- `length` is the requested page size.
- `search[value]` is the global search string.
- `order[i]` contains requested column indexes and directions.
- `columns[i]` describes the configured columns and their search and order flags.

The exact representation depends on the HTTP method and framework parser. Inspect a real request in the Network panel before implementing the endpoint parser.

## Return the required response

The endpoint returns the draw counter, the total authorized record count before filtering, the filtered count before paging, and the rows for the requested page.

~~~json
{
    "draw": 4,
    "recordsTotal": 1200,
    "recordsFiltered": 37,
    "data": [
        {
            "orderNumber": "SO-1048",
            "customerName": "Ravi Shah",
            "status": "Shipped",
            "createdAt": "2026-09-21T10:30:00Z"
        }
    ]
}
~~~

- `recordsTotal` counts records in the user's authorized scope before the current search.
- `recordsFiltered` counts matching records before the page limit is applied.
- `data` contains only the requested page.
- `draw` must be parsed as an integer before it is returned.

If the counts are inaccurate, the page controls and information text will be inaccurate too. A successful search with no matches should return zero filtered records and an empty data array.

## Build the backend query safely

Never trust a column index, order direction, length, search string, or tenant identifier from the browser. Bind values through the database driver's parameterized query interface. SQL identifiers such as an order-by column cannot usually be bound as ordinary values, so map requested indexes to a server-owned allowlist.

~~~js
const orderableColumns = [
    "order_number",
    "customer_name",
    "status",
    "created_at",
];

const requestedIndex = Number(request.order?.[0]?.column);
const sortColumn = orderableColumns[requestedIndex] ?? "created_at";
const sortDirection =
    request.order?.[0]?.dir === "asc" ? "ASC" : "DESC";

const pageSize = Math.min(
    Math.max(Number(request.length) || 25, 1),
    100,
);
const offset = Math.max(Number(request.start) || 0, 0);

const sql =
    "SELECT order_number, customer_name, status, created_at " +
    "FROM orders " +
    "WHERE tenant_id = $1 " +
    "ORDER BY " + sortColumn + " " + sortDirection + " " +
    "LIMIT $2 OFFSET $3";

const result = await database.query(sql, [
    authenticatedUser.tenantId,
    pageSize,
    offset,
]);
~~~

The column names come from a fixed server-side list, and the direction is reduced to one of two known strings. User and tenant scope come from authenticated server state. The page size is capped to prevent an oversized request. Placeholder syntax varies by database driver.

Search text must also be passed as a bound value. If the query uses wildcard search, handle wildcard characters according to the database so they do not change the intended matching rule.

## Count before filtering and after filtering

The endpoint needs two counts: the authorized base count and the count after the requested filters. Both counts must use the same access scope as the row query.

~~~sql
SELECT COUNT(*)
FROM orders
WHERE tenant_id = $1;
~~~

~~~sql
SELECT COUNT(*)
FROM orders
WHERE tenant_id = $1
  AND (
      order_number ILIKE $2
      OR customer_name ILIKE $2
      OR status ILIKE $2
  );
~~~

Use bound parameters for both the tenant and search value. The second count is not the number of rows returned for this page. It counts every record matching the filters before `LIMIT` and `OFFSET`.

For large tables, index the fields used by common filters and ordering. Compare the query plan and response time under realistic data volume.

## Apply the same filters to count and row queries

A common error is to apply a condition to the row query but forget to apply it to `recordsFiltered`, or vice versa. Build a shared filter description and use it consistently for both operations.

If the interface has a status select or date range, send it as an additional request parameter:

~~~js
const ordersTable = new DataTable("#ordersTable", {
    processing: true,
    serverSide: true,
    ajax: {
        url: "/api/orders",
        data(request) {
            request.status =
                document.querySelector("#statusFilter").value;
            request.fromDate =
                document.querySelector("#fromDate").value;
        },
    },
    columns: [
        { data: "orderNumber", name: "order_number" },
        { data: "customerName", name: "customer_name" },
        { data: "status", name: "status" },
        { data: "createdAt", name: "created_at" },
    ],
});
~~~

Validate dates and status values on the server. If a filter changes, reload the table and reset to the first page when keeping the old offset would skip matching rows.

## Treat draw as a sequence number

Ajax responses can arrive out of order. DataTables uses `draw` to associate a response with the request that produced it. Parse the received value as an integer before returning it. Do not reflect an arbitrary string from the request into HTML or JSON output.

## Handle search and ordering deliberately

The request identifies ordering columns by index and direction. Translate only indexes that the server has explicitly allowed. Ignore unsupported order instructions or return a controlled validation error.

Global search should only inspect fields intended for searching. Column search requires its own allowlist. Decide whether your server implements multi-word, exact, case-sensitive, or regex behavior. The browser's smart search settings do not automatically create those semantics in the backend.

For DataTables 3, a grouped search across selected columns is a single search over those fields. If several independent filters must all apply, represent them as separate validated conditions on the server.

## Authorization is part of every query

Apply authorization before computing counts or returning rows. A user who can access one tenant must not learn another tenant's record count through the table information element.

Never rely on hidden columns, client-side filters, paging, or disabled search controls for access control. The endpoint must enforce permissions for every request.

## When server-side mode is not useful

For a small or moderate data set, client-side processing can be simpler and avoids a request on every interaction. Server-side mode adds query, count, validation, indexing, and error-handling work. Measure actual data volume and response behavior before choosing it.

Server-side processing reduces rows transferred per draw, but it does not make an inefficient database query fast. Profile count and page queries separately.

## Server-side checklist

- The request parameters are parsed from the actual HTTP request format.
- Draw, offset, and page length are validated and bounded.
- Sort indexes map through a server-owned allowlist.
- Search and filter values are parameterized.
- Authorization scopes both counts and page rows.
- `recordsTotal` and `recordsFiltered` use the right filters.
- The endpoint returns only the requested rows.
- Response fields match the DataTables contract.
- Query plans and response times are measured with realistic data.

## Further reading

- [DataTables server-side processing manual](https://datatables.net/manual/server-side)
- [DataTables server-side processing option](https://datatables.net/reference/option/serverSide)
- [DataTables server-side protocol](https://datatables.net/manual/server-side#Sent-parameters)
- [DataTables Ajax option](https://datatables.net/reference/option/ajax)
