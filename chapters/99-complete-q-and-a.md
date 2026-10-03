# 99. Complete questions and answers

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: All code samples](./98-all-code-samples.md) | [Notes index](../README.md) | End of notes |

These answers review the main choices and patterns covered in the DataTables study notes. Use the linked core chapter when you need the full explanation and examples.

## 01. Getting started and setup

### 1. Do I need jQuery for DataTables?

DataTables 3 does not require jQuery. Older DataTables versions did, so check the version and integration used by an existing project before changing imports.

### 2. What files does a basic table need?

It needs the DataTables JavaScript library and a matching stylesheet. The page also needs a valid HTML table with a header and one body.

### 3. Should I install the CDN build or an NPM package?

Use NPM when the application already has a package manager and build step. A CDN can be useful for a small static page, but pin a version and include the matching CSS.

### 4. What does the styling package do?

The styling package connects DataTables controls to a visual system, such as the default DataTables theme or Bootstrap. Core and extensions should use compatible styling packages.

### 5. Where should initialization code run?

Run it after the table element exists in the document. In an application with route-driven views, initialize when the view mounts.

## 02. HTML structure and initialization

### 6. Which table elements should I include?

Use a table with a caption when it needs context, a `thead` for column headings, and a single `tbody` for records. Each body row should have the same number of cells as the header.

### 7. Why should table headers use `th`?

A `th` element marks a cell as a header. Correct column and row headers help browsers and assistive technology explain how each data cell relates to the table.

### 8. Can I put `rowspan` or `colspan` in the body?

DataTables supports spans in table headers and footers, but body rows should have a consistent column count. Put grouped headings in the header.

### 9. What causes a table to initialize twice?

Initialization code may run again after navigation, refresh, or component remount. Check whether the table is already initialized and update it through its existing API instance.

## 03. Data sources

### 10. When should I use an existing DOM table?

Use DOM data when the server or page has already rendered a modest set of rows. DataTables reads the table cells as its starting data.

### 11. When should I use JavaScript arrays or objects?

Use JavaScript data when the application already holds records in memory or receives them from an API. Objects are convenient when each field has a descriptive name.

### 12. What is the difference between array and object rows?

Array rows use positions, so a value such as index `2` depends on column order. Object rows use named fields, which are easier to understand and maintain when a schema has several columns.

### 13. How do I load data from JSON?

Configure the Ajax source and ensure `dataSrc` points to the array of records. The default expects a `data` property unless the response mapping is changed.

## 04. Columns, rendering, and data types

### 14. What does `columns.data` do?

It tells DataTables which field or position supplies a column's value. For object rows, use the property's name, such as `customerName`.

### 15. What does `columns.render` do?

It formats or transforms a value for display, filtering, ordering, or other data types. Keep the underlying value accurate and return suitable values for the operations the table needs.

### 16. What is orthogonal data?

It is the use of different representations of one value for different purposes. For example, a date can display in a friendly format while sorting by a sortable timestamp.

### 17. How should I display untrusted text?

Use `DataTable.render.text()` for plain strings. It escapes HTML so text from a record is not interpreted as markup when displayed.

## 05. Searching and filtering

### 18. What does the built-in search field search?

It searches the searchable columns in the current table data. Use a per-column search when a filter should apply to one specific field.

### 19. How do I add a filter for one column?

Call `column(index).search(value).draw()` from the filter control. Give that control a clear label and keep the field-to-column mapping stable.

### 20. How can I make a search exact?

Use the current search options supported by the installed DataTables version to set exact matching. Check the version-specific API before copying options from an older example.

### 21. Why can a search return unexpected matches?

The table may search rendered or normalized values, the column may not match the assumed index, or an older filter may still be active. Inspect the configured columns and clear filters through the API.

## 06. Ordering and custom sorting

### 22. How do I set the initial order?

Use the `order` option with the column index and direction, such as ascending or descending. Confirm the index still points to the intended column after columns are added or moved.

### 23. How do I allow ordering on some columns only?

Set `orderable` for each relevant column or define it through column definitions. Keep action buttons and other non-data columns out of ordering when it would confuse users.

### 24. Why does a date or currency sort incorrectly?

The displayed text may not sort in chronological or numeric order. Supply a suitable data type or a render value that gives DataTables a consistent ordering value.

### 25. When is custom ordering necessary?

Use it when built-in type detection cannot represent the application's real ordering rule. Prefer a standard data representation when possible because it is easier to maintain.

## 07. Paging and page length

### 26. What does page length control?

It controls how many rows are shown on each page. Choose a value that keeps the table usable without making the page unnecessarily long.

### 27. How do I change pages through the API?

Use the paging API to set the page, then call `draw()` to update the display. Validate page numbers when they come from application controls.

### 28. What does `draw(false)` preserve?

It redraws the table while keeping the current paging position where possible. Use it after updates when returning the user to the first page would be disruptive.

### 29. Should I remove paging for a small table?

It can make sense when the full result is consistently short. Reconsider the choice if the data can grow or if users need predictable page performance.

## 08. API and table updates

### 30. How do I get a table instance?

Store the instance returned by `new DataTable(...)` and reuse it for API operations. Avoid creating another instance just to search, reload, or update rows.

### 31. How do I add multiple records?

Pass the collection to `rows.add()` and draw once after the batch. A single draw avoids repeating table work for every inserted record.

### 32. How do I update one row?

Select it through the API, set its data, and draw the table. Use a stable row identifier when records may be reordered or filtered.

### 33. When should I use `ajax.reload()`?

Use it when the server is the source of truth and the table needs fresh records. Its paging-reset argument lets the application preserve the current page when appropriate.

### 34. When should I destroy and recreate a table?

Do so only when the table structure or an initialization-only option must change and no API can make that change. For row changes, use the existing instance.

## 09. Events and lifecycle

### 35. What is the difference between initialization and draw events?

Initialization events happen as the table is created and its initial data is loaded. Draw events happen after visible rows are rendered, including later paging, search, and ordering changes.

### 36. Why should row event handlers be delegated?

Rows can be replaced or created during later draws. A delegated handler on the DataTables API or a stable parent continues to receive events from those rows.

### 37. Why do duplicate event handlers appear?

A setup function may register the same handler each time a view mounts. Remove handlers during cleanup or ensure setup runs once for the table instance.

### 38. Where should I clean up a table in a single-page application?

Destroy the instance and remove application event handlers before the view's table element is removed. Reinitialize only after a new element is present.

## 10. Ajax and JSON

### 39. What is `ajax.dataSrc` for?

It tells DataTables where the row array is located in the response. Use it when the array is nested or stored under a property other than the default.

### 40. Why is a successful request still showing no rows?

A successful HTTP response can contain malformed JSON, an empty array, or a property name that does not match `dataSrc`. Inspect the response body in the browser's Network panel.

### 41. How should the page communicate loading?

Keep the loading message clear and visible, then show useful empty or error feedback when the request completes. Do not leave users with a blank table that looks broken.

### 42. How do I refresh Ajax data?

Call `ajax.reload()` on the existing instance. Choose whether to reset the page based on whether new data might invalidate the current page.

### 43. Should an API return every field in the database?

No. Return the fields needed for this table and only the records the current user may access. Smaller responses are easier to protect and faster to process.

## 11. Server-side processing

### 44. When should I enable server-side processing?

Use it when the full result set is too large to send to the browser or when filtering and ordering belong in a server query. It is unnecessary overhead for a small data set already loaded in the page.

### 45. What does the server receive for each draw?

It receives paging, ordering, and search information along with a draw counter. Treat the request as untrusted input and validate every field before building a query.

### 46. What counts must the response include?

The response reports the total number of records before filtering and the number remaining after filtering, as well as the current data rows and draw value. These counts let DataTables describe and page the result correctly.

### 47. Can I trust the requested column index?

No. Map requested indexes to a server-side allowlist of known database fields. Reject unsupported sort directions and never concatenate client-provided names into SQL.

### 48. How should filtering and authorization interact?

Apply authorization before returning records and before reporting counts. A filter must never allow a user to infer the existence of rows they cannot access.

## 12. Responsive tables and extensions

### 49. What does Responsive do?

It adjusts which columns are visible as the available width changes. It can expose hidden values through a details display, depending on its configuration.

### 50. Should I enable every extension?

No. Add extensions only when they solve a clear user need. Each extension adds assets, behavior, and interactions that need to be tested.

### 51. Are hidden responsive values automatically accessible?

Do not assume that every details display works equally well with every screen reader and input method. Test the actual layout and provide another way to access important values if needed.

### 52. When is horizontal scrolling a good choice?

Use it when preserving the full column structure matters more than reflow. Test navigation and screen-reader behavior in the chosen theme and avoid using it as the only small-screen strategy without user testing.

## 13. Buttons, export, and layout

### 53. How do I place Buttons controls?

Configure the Buttons extension in the DataTables `layout` option. Keep related actions together and give custom actions a clear name.

### 54. What is needed for Excel or PDF export?

Excel export uses JSZip, and PDF export uses PDFMake with the required fonts. Register the dependencies and modules that the selected buttons need.

### 55. Does a server-side table export every matching row?

Usually the browser only has the current page, so a client-side export cannot include records it never received. Use an authorized server export endpoint for a complete report.

### 56. Should a hidden column be included in an export?

Choose this deliberately. Export visible columns when that matches user expectations, or document why the report includes fields hidden from the table.

## 14. State saving and accessibility

### 57. What does `stateSave` remember?

It can restore paging, page length, ordering, search, and column visibility. The saved state can make a returning user's table feel familiar.

### 58. Where does DataTables store state by default?

The built-in behavior uses browser storage. By default, state is valid for two hours and uses `localStorage`; `stateDuration: -1` selects session storage and `0` means no expiry.

### 59. Can saved state contain private information?

Yes. Search terms can contain names or identifiers. Clear fields through `stateSaveParams` when they should not persist, and never rely on browser storage to enforce access.

### 60. Does `tabIndex` add keyboard access to every cell?

No. The option controls focus for DataTables' interactive controls, including search, ordering, and paging. Add cell navigation only when the application needs it and has tested the interaction.

### 61. What basic table markup helps screen-reader users?

Give the table a clear caption and use header cells with appropriate scope. Test the actual table and extensions with the assistive technology used by the audience.

## 15. Security, performance, and debugging

### 62. Can table content contain HTML?

It can, but untrusted strings should normally be rendered as text. Use `DataTable.render.text()` to escape plain text and avoid assembling markup from raw values.

### 63. Does deferred rendering avoid downloading every record?

No. It reduces how many DOM nodes are built up front for Ajax or JavaScript data. Server-side processing is needed when the browser should not receive the full data set.

### 64. Why does a table report that it cannot be reinitialized?

The constructor is being called for a table that already has an active instance. Check with `DataTable.isDataTable()` and update through the existing API.

### 65. What should I inspect when Ajax shows no rows?

Check the console, request URL, response status, response JSON, and `dataSrc`. Confirm that the response contains the field names expected by the configured columns.

### 66. How can I tell whether a renderer is the performance problem?

Measure before changing it. Compare with a small data set and temporarily remove custom renderers or extensions, then add each part back while observing draw time.

## 16. Integration and production

### 67. How do I add DataTables to a package-managed application?

Install the core styling package, import its constructor, and load its CSS. Add extension packages only for features the page actually uses.

### 68. Do core and extensions need matching styling packages?

Yes. Use compatible core and extension packages for the same styling framework. Mixed integrations can produce inconsistent or missing styles.

### 69. Who should own table rows in a single-page application?

Choose one owner. If DataTables manages the rows, send updates through its API rather than changing its generated body directly from another renderer.

### 70. What should happen when a table view is removed?

Clean up its instance and event handlers before its DOM is removed. This prevents stale listeners and duplicate initialization when the route is opened again.

### 71. What should be checked in a production build?

Confirm package versions, CSS assets, base paths, API routes, response shapes, authorization, loading and error states, keyboard behavior, and performance with realistic data.

## General design questions

### 72. Should all tables use the same configuration?

Share stable defaults such as a common page length or text rendering policy. Keep data mappings and user-specific behavior close to each table.

### 73. Should a table always use server-side processing?

No. Use client-side mode for modest data sets that the browser can handle comfortably. Choose server-side mode when the complete data set should stay on the server or client processing becomes too expensive.

### 74. How should I choose page length?

Consider the number of visible columns, screen size, typical task, and response time. Offer a small set of useful values instead of an unlimited list.

### 75. What is the safest order for building a table?

Start with valid semantic HTML and a small data set. Add data mapping, then one behavior at a time, and test search, order, paging, errors, and access rules as they are introduced.

## Official references

- [DataTables installation](https://datatables.net/manual/core/installation)
- [DataTables API](https://datatables.net/manual/core/api)
- [DataTables server-side processing](https://datatables.net/manual/server-side)
- [DataTables security](https://datatables.net/manual/core/security)
- [DataTables accessibility and keyboard settings](https://datatables.net/reference/option/tabIndex)
- [DataTables state saving](https://datatables.net/reference/option/stateSave)