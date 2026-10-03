# 05. Searching and filtering

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Columns, rendering, and data types](./04-columns-rendering-and-data-types.md) | [Notes index](../README.md) | [Next: Ordering and custom sorting](./06-ordering-and-sorting.md) |

## Use the built-in search

The built-in search control filters the rows held by a client-side table. DataTables smart search supports multiple words, quoted phrases, and negative terms. It ignores letter case by default.

For example, searching for `new york` can match a row containing both words even when they are not next to each other. Quoting a phrase asks for that phrase together. A term with a leading minus excludes rows containing that term.

Search uses the values DataTables prepared for filtering. If a column has a renderer, return an appropriate value for the `filter` operation as well as the display value.

## Search from application code

The API queues a search change. Call `draw()` to apply the search and update the visible rows.

~~~js
const staffTable = new DataTable("#staffTable");

staffTable.search("London").draw();
~~~

The global search can be configured with search options:

~~~js
staffTable
    .search("Engineer", {
        caseInsensitive: false,
        smart: true,
    })
    .draw();
~~~

Use regular expressions only when the application needs them. Treating user input as a regular expression can surprise users and can make search behavior harder to explain. The default string search safely treats regular expression characters as ordinary text.

## Search a single column

Use `column().search()` when a filter belongs to a particular field.

~~~js
staffTable.column(1).search("Engineering").draw();
~~~

Column indexes are zero-based. This searches column 1, the second visible data column. A search on one column is a useful starting point for department, status, or location filters.

A select control can perform an exact match:

~~~html
<label for="departmentFilter">Department</label>
<select id="departmentFilter">
    <option value="">All departments</option>
    <option value="Engineering">Engineering</option>
    <option value="Research">Research</option>
</select>
~~~

~~~js
const departmentFilter = document.querySelector("#departmentFilter");

departmentFilter.addEventListener("change", () => {
    staffTable
        .column(1)
        .search(departmentFilter.value, { exact: true })
        .draw();
});
~~~

An empty string clears that column's search. The explicit label helps users understand what the select controls.

## Search across selected columns in DataTables 3

DataTables 3 allows one search term to be checked across a selected set of columns. A row matches if the term matches a selected column.

~~~js
staffTable
    .columns([1, 2])
    .search("London")
    .draw();
~~~

Here, the search term is checked against both selected columns as one grouped search. This is an OR across the selected columns. It does not apply a separate copy of the term to each selected column as older versions did.

If the application needs simultaneous independent filters, set each column separately:

~~~js
staffTable
    .column(1)
    .search("Engineering")
    .column(2)
    .search("London")
    .draw();
~~~

This keeps rows that satisfy both column filters. The difference is useful when migrating from older examples: use `columns().search()` for a grouped search over selected fields, and use separate singular column searches for independent conditions.

## Limit the built-in search input

The built-in search input can be restricted to chosen columns without disabling those columns for every other search:

~~~js
const staffTable = new DataTable("#staffTable", {
    layout: {
        topEnd: {
            search: {
                columns: [0, 1],
            },
        },
    },
});
~~~

The `search.columns` option controls the columns used by that interface. The `columns.searchable` setting is broader: it disables search for a column altogether. Choose based on whether the field should be searchable through any interface or only excluded from the main search box.

## Keep separate filters with fixed search

A fixed search has a name and can remain active alongside the user's regular search. It is useful when an application filter should not overwrite the text entered into the built-in search.

~~~js
staffTable.search.fixed("active-records", (searchText, row) => {
    return row.active === true;
});

staffTable.draw();
~~~

A named fixed search can be replaced or removed independently:

~~~js
staffTable.search.fixed("active-records", null).draw();
~~~

Use a stable name owned by the feature that applies the condition. Remove or update the fixed filter when that feature changes. Fixed searches are client-side logic; server-side processing requires the application endpoint to apply equivalent conditions.

## Clear search conditions

Clear the global and column searches when the user asks to reset the table:

~~~js
staffTable.search("");
staffTable.columns().search("");
staffTable.search.fixed("active-records", null);
staffTable.draw();
~~~

If the interface has several named fixed searches, clear each one that belongs to the current reset action. Keep the UI controls synchronized with the API state.

## Search formatted values correctly

Search uses a column's filter data. A renderer can provide text that is easier to find than the display alone:

~~~js
{
    data: "phone",
    render(data, type) {
        if (type === "display") {
            return data.replace(/(\d{3})(\d{3})(\d+)/, "$1-$2-$3");
        }

        if (type === "filter") {
            return data.replaceAll("-", "");
        }

        return data;
    },
}
~~~

This example displays a formatted phone number while allowing a user to search for digits without hyphens. For this to work, the source value should use a consistent format.

Test search against the value users see and the raw values they may know. Include expected abbreviations or alternate names in the filter data only when they are meaningful and maintained.

## Add a custom input

A custom input gives the application control over its label and placement:

~~~html
<label for="staffSearch">Search staff</label>
<input id="staffSearch" type="search" />
~~~

~~~js
const staffSearch = document.querySelector("#staffSearch");

staffSearch.addEventListener("input", () => {
    staffTable.search(staffSearch.value).draw();
});
~~~

For a large client-side table, drawing on every keystroke may do unnecessary work. Consider a short debounce when profiling shows the input feels slow. Always keep the search response understandable and let the user clear the field.

## Search and server-side processing

With `serverSide: true`, the browser sends search terms to the endpoint. The server is responsible for validating and applying those terms. Client search options such as exact matching do not automatically define backend query behavior.

Do not concatenate search text into a SQL statement. Use parameterized queries, restrict searchable fields to an explicit list, and escape wildcard syntax according to the database and search rules. The full request and response flow is covered in chapter 11.

## Search checklist

- The right fields are searchable through each search control.
- Exact select filters use exact matching where it fits.
- Column indexes agree with the visible column order.
- Independent filters use separate column searches.
- Grouped search over several columns accounts for DataTables 3 behavior.
- Custom filters are cleared when the user resets the table.
- Filter values are useful for the actual rendered data.
- Server search is validated and parameterized by the backend.

## Further reading

- [DataTables searching](https://datatables.net/manual/core/search)
- [DataTables search API](https://datatables.net/reference/api/search())
- [DataTables column search API](https://datatables.net/reference/api/column().search())
- [DataTables grouped column search API](https://datatables.net/reference/api/columns().search())
- [DataTables 3 grouped search changes](https://datatables.net/releases/3/new#Grouped-column-search)
