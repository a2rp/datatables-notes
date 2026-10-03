# 10. Ajax loading and JSON

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Events and lifecycle](./09-events-and-lifecycle.md) | [Notes index](../README.md) | [Next: Server-side processing](./11-server-side-processing.md) |

## Load rows from an endpoint

The `ajax` option tells DataTables where to request client-side rows. The endpoint returns JSON containing an array of objects or arrays. By default, DataTables reads that array from the response's `data` property.

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

A matching response is:

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

Each object field must agree with the corresponding `columns.data` value. Use the Network panel to inspect the actual JSON response when a table is empty.

## Point to a different JSON property

If the endpoint uses a different property name, configure `dataSrc`:

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

A dotted path can read an array nested inside an object, such as `payload.records`. For an endpoint that returns a JSON array at the root, use an empty string:

~~~js
const staffTable = new DataTable("#staffTable", {
    ajax: {
        url: "/api/staff",
        dataSrc: "",
    },
    columns: [
        { data: "name" },
        { data: "department" },
        { data: "office" },
    ],
});
~~~

## Transform a response when needed

A `dataSrc` function can map an API response to the row array that DataTables expects:

~~~js
const staffTable = new DataTable("#staffTable", {
    ajax: {
        url: "/api/staff",
        dataSrc(response) {
            return response.payload.records;
        },
    },
    columns: [
        { data: "name" },
        { data: "department" },
        { data: "office" },
    ],
});
~~~

Keep the response mapping small and explicit. If many pages need a different shape, it may be clearer to make the API return a consistent data contract.

Do not override the Ajax success callback. DataTables uses it internally to process the response. Use `dataSrc` to select or transform client-side row data, and use lifecycle events to observe the request.

## Add request parameters

The `ajax.data` option can add application filters to a request. A function can read current UI values each time the table requests data.

~~~js
const staffTable = new DataTable("#staffTable", {
    ajax: {
        url: "/api/staff",
        data(request) {
            request.department =
                document.querySelector("#departmentFilter").value;
        },
    },
    columns: [
        { data: "name" },
        { data: "department" },
        { data: "office" },
    ],
});
~~~

The function receives the request data object. Add fields to it when DataTables parameters already need to be preserved. Returning a replacement object may discard other request parameters. For server-side processing, the request already contains paging, ordering, and search information, so preserve it when adding application filters.

## Choose the request method and body

Use the request method expected by the endpoint. A simple POST can be configured like this:

~~~js
const staffTable = new DataTable("#staffTable", {
    ajax: {
        url: "/api/staff/search",
        type: "POST",
        data(request) {
            request.department =
                document.querySelector("#departmentFilter").value;
        },
    },
});
~~~

The server must validate the submitted values and check authorization. Never treat a browser-supplied department, user ID, sort field, or page index as trusted.

For cookie-authenticated endpoints, keep the request within the application's credential policy. For token-based access, configure the request header through the Ajax settings supported by the installed version and avoid storing long-lived secrets in public client code.

## Show loading and error states

Enable the processing indicator when requests or table work may take noticeable time. Application-specific loading and error elements can provide more useful messages.

~~~html
<p id="loadError" role="status" hidden>
    Staff data could not be loaded. Try again later.
</p>
~~~

~~~js
const loadError = document.querySelector("#loadError");

const staffTable = new DataTable("#staffTable", {
    processing: true,
    ajax: "/api/staff",
    on: {
        xhr(event, settings, json) {
            loadError.hidden = Boolean(json);
        },
        "dt-error"() {
            loadError.hidden = false;
        },
    },
});
~~~

Do not display internal stack traces or sensitive response details to the user. Record diagnostic information in a controlled logging system and show a short recovery message.

## Reload the configured endpoint

When the server has new records, call `ajax.reload()` instead of constructing a second request and manually replacing table rows:

~~~js
refreshButton.addEventListener("click", () => {
    staffTable.ajax.reload(null, false);
});
~~~

The second argument preserves the current page when possible. Reset to the first page when a changed filter could make the current page invalid:

~~~js
departmentFilter.addEventListener("change", () => {
    staffTable.ajax.reload();
});
~~~

Keep the visible filter and the request parameters synchronized so the data reflects the control state.

## Handle empty and malformed responses

An empty result should be a valid response with an empty data array:

~~~json
{
    "data": []
}
~~~

A response with the wrong top-level property or the wrong row shape will not populate the table correctly. Confirm the HTTP status, JSON parse, array location, field names, and content type.

Keep API errors distinct from an empty search result. A failed request should display an error state, while a successful request returning no matching rows should use the table's empty-state message.

## Know when client-side Ajax is not enough

With client-side Ajax, the endpoint returns all rows that DataTables will search and order in the browser. This can work well for a moderate data set.

When the data is very large, loading every row can use significant network, memory, and rendering time. Server-side processing requests only the page and applies search and ordering at the endpoint. Its request and response protocol is covered in the next chapter.

## Ajax checklist

- The endpoint returns valid JSON with an array of rows.
- `dataSrc` matches the response shape.
- Every object field agrees with the column mapping.
- Request parameters are added without losing required data.
- The endpoint validates filters and enforces authorization.
- Loading, empty, and error states are distinct.
- A refresh reuses the configured Ajax source.
- Large data sets are evaluated for server-side processing.

## Further reading

- [DataTables Ajax manual](https://datatables.net/manual/core/ajax)
- [DataTables ajax option](https://datatables.net/reference/option/ajax)
- [DataTables ajax.dataSrc option](https://datatables.net/reference/option/ajax.dataSrc)
- [DataTables ajax.data option](https://datatables.net/reference/option/ajax.data)
- [DataTables ajax.reload API](https://datatables.net/reference/api/ajax.reload())
- [DataTables Ajax and server-side processing](https://datatables.net/manual/server-side)
