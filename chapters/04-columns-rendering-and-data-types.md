# 04. Columns, rendering, and data types

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Data sources](./03-data-sources.md) | [Notes index](../README.md) | [Next: Searching and filtering](./05-searching-and-filtering.md) |

## Map each visible column to its data

The table header order and the `columns` configuration describe the same sequence. For object data, `columns.data` names the value to read from each row. Keep the raw record intact and format a separate value for presentation.

~~~js
const products = [
    {
        sku: "BK-104",
        name: "Notebook",
        stock: 32,
        price: 4.5,
    },
    {
        sku: "PN-220",
        name: "Pen set",
        stock: 18,
        price: 12,
    },
];

const productsTable = new DataTable("#productsTable", {
    data: products,
    columns: [
        { data: "sku" },
        { data: "name" },
        { data: "stock" },
        { data: "price" },
    ],
});
~~~

The first column reads `sku`, the second reads `name`, and so on. The order in the array must agree with the table's header cells.

## Give columns stable names and behavior

Column options can define names, classes, default content, and whether users can order or search the values.

~~~js
const productsTable = new DataTable("#productsTable", {
    columns: [
        { data: "sku", name: "sku" },
        { data: "name", name: "product-name" },
        { data: "stock", name: "stock", className: "numeric-cell" },
        {
            data: "price",
            name: "price",
            className: "numeric-cell",
            searchable: false,
        },
    ],
});
~~~

Use `name` to identify a column in API code or server-side requests. Use `className` for a styling hook. A column that should not be searchable or orderable can set the corresponding flag to false. Do not disable a control merely to make the interface simpler if users need it to compare or find records.

## Format numbers for display and keep raw values for order

A currency value should look familiar to people, but its raw number should remain available for ordering. The renderer receives the original value and the operation DataTables needs it for.

~~~js
const money = new Intl.NumberFormat("en-US", {
    style: "currency",
    currency: "USD",
});

const productsTable = new DataTable("#productsTable", {
    data: products,
    columns: [
        { data: "sku" },
        { data: "name" },
        { data: "stock" },
        {
            data: "price",
            render(data, type) {
                if (type === "display") {
                    return money.format(data);
                }

                return data;
            },
        },
    ],
});
~~~

The display value is formatted, while ordering and type detection receive the number. This separation is called orthogonal data. A renderer can be called multiple times for display, searching, ordering, and type detection, so it should return the appropriate value for every requested type.

DataTables also includes number renderers for common formatting:

~~~js
{
    data: "price",
    render: DataTable.render.number(null, null, 2, "$"),
}
~~~

The renderer formats the displayed value and retains a usable underlying value for other operations. Use the current API documentation for locale and formatting details.

## Handle dates with a machine-readable value

Dates formatted for people can sort incorrectly as plain text. Keep an ISO date or timestamp as the source value, then format the display.

~~~js
const rows = [
    { name: "Ada Lovelace", joined: "2024-03-14" },
    { name: "Grace Hopper", joined: "2023-11-02" },
];

const peopleTable = new DataTable("#peopleTable", {
    data: rows,
    columns: [
        { data: "name" },
        {
            data: "joined",
            render(data, type) {
                if (type === "display") {
                    return new Intl.DateTimeFormat("en-GB", {
                        dateStyle: "medium",
                    }).format(new Date(data + "T00:00:00"));
                }

                return data;
            },
        },
    ],
});
~~~

The ISO source remains suitable for ordering. A date-only string is turned into a local date by adding a time portion before constructing the Date. Choose a date strategy that matches the time zone meaning of the source field.

## Provide different values for display, search, and ordering

A record can provide ready-made values for different operations. An object renderer describes the default value and any specialized values:

~~~js
const rows = [
    {
        joined: {
            display: "14 Mar 2024",
            sort: 1710374400,
            filter: "14 Mar 2024 2024-03-14",
        },
    },
];

const peopleTable = new DataTable("#peopleTable", {
    data: rows,
    columns: [
        {
            data: "joined",
            render: {
                _: "display",
                display: "display",
                sort: "sort",
                type: "sort",
                filter: "filter",
            },
        },
    ],
});
~~~

The `_` value is the fallback for an operation without a dedicated mapping. This is useful when an API already sends both readable and machine values. Use one clear source format when deriving the extra values in a renderer is simpler.

## Combine values from the whole row

Set `data` to null when a column is calculated from more than one field. The renderer then receives the complete row as its third argument.

~~~js
const invoices = [
    { subtotal: 120, tax: 12 },
    { subtotal: 85, tax: 8.5 },
];

const invoiceTable = new DataTable("#invoiceTable", {
    data: invoices,
    columns: [
        { data: "subtotal" },
        { data: "tax" },
        {
            data: null,
            render(data, type, row) {
                const total = row.subtotal + row.tax;

                if (type === "display") {
                    return money.format(total);
                }

                return total;
            },
        },
    ],
});
~~~

Keep calculations deterministic and inexpensive because rendering can happen multiple times. For a value that is expensive to calculate, compute it once in the data layer or return it from the API.

## Show a fallback for missing values

An API may omit a field or return null. Use `defaultContent` for a fixed fallback:

~~~js
{
    data: "manager",
    defaultContent: "Not assigned",
}
~~~

A renderer can handle several display cases when the output depends on the record:

~~~js
{
    data: "email",
    render(data, type) {
        if (data == null || data === "") {
            return type === "display" ? "No email" : "";
        }

        return data;
    },
}
~~~

Do not turn missing data into a value that misleads users. For numeric values, decide whether missing means zero, not available, or an error. Those cases should not be treated as the same thing by default.

## Render untrusted text safely

Data from an API is still data from outside the current page. Do not concatenate an untrusted string into HTML markup. The text renderer escapes HTML entities before placing the value in a cell:

~~~js
{
    data: "customerName",
    render: DataTable.render.text(),
}
~~~

If a display cell needs an interactive element, create a DOM node and assign untrusted text through `textContent`. Validate any URL before assigning it to `href`. Keep the non-display return value suitable for sorting and searching.

## Keep ordering data numeric

Formatted numbers such as "$1,200" and "900" may be interpreted as text if the column mixes formats. Store the value as a number and use a renderer only for display. Avoid adding visual punctuation into the underlying record.

When a column contains a custom format, inspect its type detection and order behavior. Add a custom ordering plug-in only when the built-in type detection and a simple renderer do not match the data.

## Renderer checklist

- `columns.data` points to the intended field.
- The `columns` array follows the visible header order.
- Display formatting does not replace the original value.
- Renderers handle display, filter, sort, and type requests appropriately.
- Missing values have a deliberate fallback.
- API text is escaped rather than inserted as raw HTML.
- Calculated values are stable and do not do costly work repeatedly.

## Further reading

- [DataTables renderers](https://datatables.net/manual/core/data/renderers)
- [DataTables orthogonal data](https://datatables.net/manual/core/data/orthogonal-data)
- [DataTables columns.data](https://datatables.net/reference/option/columns.data)
- [DataTables columns.render](https://datatables.net/reference/option/columns.render)
- [DataTables built-in render helpers](https://datatables.net/manual/core/data/renderers#Built-in-helpers)
