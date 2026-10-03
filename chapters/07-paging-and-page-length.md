# 07. Paging and page length

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Ordering and custom sorting](./06-ordering-and-sorting.md) | [Notes index](../README.md) | [Next: The API and table updates](./08-api-and-table-updates.md) |

## What paging does

Paging shows a portion of a table at a time. With client-side processing, all configured rows are still held in the browser; paging changes how many are displayed, not how many were downloaded.

Paging helps people scan long lists and can reduce the amount of table markup drawn at once. It is not a security boundary. Do not send records to the browser if the user is not allowed to receive them.

## Set the first page size

Use `pageLength` for the initial number of rows and `lengthMenu` for the choices offered to the user:

~~~js
const ordersTable = new DataTable("#ordersTable", {
    pageLength: 25,
    lengthMenu: [10, 25, 50, 100],
});
~~~

Choose a default that balances scanning with the amount of information users need to see. Very large page sizes can make dense tables difficult to read.

A length of -1 means show all rows. Use it only when the client-side data set is small enough and users have a clear reason to view everything at once.

## Arrange the page controls

DataTables 3 uses the `layout` option to place the page-size selector, search box, information text, and paging controls around the table:

~~~js
const ordersTable = new DataTable("#ordersTable", {
    layout: {
        topStart: "pageLength",
        topEnd: "search",
        bottomStart: "info",
        bottomEnd: "paging",
    },
});
~~~

Keep related controls in predictable places. On narrow screens, check that controls wrap without causing horizontal scrolling and that each input has an accessible name.

## Navigate with the API

The page API uses zero-based indexes. The third page is index 2:

~~~js
ordersTable.page(2).draw("page");
~~~

Move to the next or previous page through the same API:

~~~js
ordersTable.page("next").draw("page");
ordersTable.page("previous").draw("page");
~~~

A request past the first or final page does not create another page of records. Check the current page count before building custom navigation around these methods.

## Understand draw behavior

API changes are queued so several settings can be changed before a redraw. The draw mode controls how much work is repeated and whether the current page is kept.

~~~js
ordersTable
    .page.len(50)
    .order([[1, "asc"]])
    .draw(false);
~~~

This changes the page length and order, then redraws while keeping the current page when possible. If the new result set has fewer pages, the current page may move to the last available page.

The page-only draw mode is useful after changing the page through the API. It redraws the current page without recalculating ordering and search. Use a full draw after changing filters or data so the result set is recalculated.

## Inspect paging information

The `page.info()` API returns the current page and record counts:

~~~js
const info = ordersTable.page.info();

console.log({
    pageIndex: info.page,
    pageCount: info.pages,
    pageLength: info.length,
    recordsDisplayed: info.end - info.start,
    recordsTotal: info.recordsTotal,
});
~~~

Use this information to show custom status text or disable external navigation controls at the first and last pages. Prefer the built-in controls when they meet the design needs.

## Keep custom controls synchronized

A custom pager needs to reflect table state after every page change, search, ordering change, and data update. DataTables emits a draw event after the displayed result has changed.

~~~js
const pageStatus = document.querySelector("#pageStatus");

ordersTable.on("draw", () => {
    const { page, pages } = ordersTable.page.info();
    pageStatus.textContent = pages === 0
        ? "No pages"
        : "Page " + (page + 1) + " of " + pages;
});
~~~

The visible page number is one-based for people even though the API index is zero-based. Make custom controls keyboard-operable and give each control a clear label. Chapter 9 covers event lifecycle details.

## Search and paging

A new search usually moves the table to the first page because later pages may no longer exist after filtering. This avoids showing an empty page when matching records are found near the start of the result set.

When filtering from application code, queue the search and then perform a full draw:

~~~js
ordersTable.search("processing").draw();
~~~

If the user changes page length or ordering and the product should preserve the current page, use the draw mode that keeps paging where possible. Test the choice with a filter that removes most rows.

## Client-side and server-side page counts

For a client-side table, DataTables already has the loaded rows and computes page counts in the browser. For server-side processing, the endpoint returns both the requested page of rows and the total counts needed to describe the result set.

A server response must report the number of records before filtering and after filtering. If those values are wrong, the pager may show a misleading page count even when the returned rows look correct.

## Paging checklist

- The default page length fits the density and purpose of the table.
- The length choices are useful and do not encourage loading excessive data.
- Page indexes are translated correctly between zero-based API values and one-based labels.
- Custom controls update after each draw.
- Search and ordering changes do not leave users on an empty page.
- Paging is not used to restrict access to records.
- Server-side page totals come from correct count queries.

## Further reading

- [DataTables pageLength option](https://datatables.net/reference/option/pageLength)
- [DataTables lengthMenu option](https://datatables.net/reference/option/lengthMenu)
- [DataTables paging API](https://datatables.net/reference/api/page())
- [DataTables page information API](https://datatables.net/reference/api/page.info())
- [DataTables draw API](https://datatables.net/reference/api/draw())
- [DataTables layout option](https://datatables.net/reference/option/layout)
