# 12. Responsive tables and extensions

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Server-side processing](./11-server-side-processing.md) | [Notes index](../README.md) | [Next: Buttons, export, and layout](./13-buttons-export-and-layout.md) |

## Design for the available width

A table is naturally wider than a paragraph because each row presents related values side by side. Start by deciding which fields people need to compare at the same time. Then choose how the table should behave when its container becomes narrow.

Common choices include:

- Reflow content outside the table when each record reads better as a card.
- Keep only the most useful columns visible and reveal other values on demand.
- Allow horizontal scrolling when users must compare many fields across rows.
- Offer a focused detail view for a record when the table is too dense.

Do not remove information without giving users a way to reach it when it matters.

## Enable the Responsive extension

Responsive can hide lower-priority columns when the available width becomes too small, then show their values in a details row. It is an extension, so load its JavaScript and styling package along with DataTables.

For a package-managed project using the default styling:

~~~sh
npm install datatables.net-responsive-dt
~~~

~~~js
import DataTable from "datatables.net-dt";
import "datatables.net-responsive-dt";

const ordersTable = new DataTable("#ordersTable", {
    responsive: true,
});
~~~

For CDN or local files, use the download builder to select DataTables, Responsive, and the styling integration together. This helps keep compatible versions and required assets aligned.

## Set column visibility priority

Responsive assigns a priority to each column. A lower number means the column should remain visible longer. Give the key identifier and status higher priorities than optional details:

~~~js
const ordersTable = new DataTable("#ordersTable", {
    responsive: true,
    columnDefs: [
        { responsivePriority: 1, targets: 0 },
        { responsivePriority: 2, targets: 1 },
        { responsivePriority: 100, targets: 4 },
    ],
});
~~~

In this example, the first two columns are more important to keep visible, while column 4 is more likely to be hidden. Confirm indexes against the real header order and test at the narrowest supported width.

Priority does not make a column's information disappear from the data set. With the standard details display, users can open the row details to see hidden values.

## Choose a details control

The default Responsive behavior can place an expand control in the first column. A dedicated control column can keep that marker separate from the record identifier:

~~~js
const ordersTable = new DataTable("#ordersTable", {
    responsive: {
        details: {
            type: "column",
        },
    },
    columnDefs: [
        {
            className: "dtr-control",
            orderable: false,
            searchable: false,
            targets: 0,
        },
    ],
});
~~~

If the control uses the first column, make sure its heading still describes the record identifier and the expand control has an understandable accessible name. Test it with a keyboard and a screen reader. The control should not be the only way to access a critical action.

## Use horizontal scrolling for comparison

Some data is useful only when several columns can be compared side by side. Horizontal scrolling can be clearer than hiding fields and opening each row.

~~~js
const metricsTable = new DataTable("#metricsTable", {
    scrollX: true,
});
~~~

Make the scroll region apparent and test keyboard access. Use a concise table caption and keep essential identifying columns visible when the user scrolls. Horizontal scrolling is a presentation choice, not a replacement for a small-screen layout check.

## Set explicit breakpoint classes when needed

Automatic hiding works for many tables. A class-based breakpoint is useful when a field should appear only at certain widths.

~~~js
const peopleTable = new DataTable("#peopleTable", {
    responsive: true,
    columns: [
        { data: "name", className: "all" },
        { data: "department", className: "desktop tablet" },
        { data: "office", className: "desktop" },
        { data: "email", className: "none" },
    ],
});
~~~

The `all` class keeps a column visible. The `none` class hides it from the main row while allowing its value in the Responsive details view. Do not use a hidden detail value as the only way to expose a primary action or a required field.

Breakpoint names depend on the extension's configured breakpoints. Check the installed version and test actual container widths rather than assuming a device model.

## Select extensions for a user need

Extensions add capabilities beyond the core table. Add them when a user task needs them, then keep their controls discoverable and their output accessible.

- **Responsive:** fit columns into narrow containers and expose hidden values.
- **Buttons:** provide export, copy, print, and column visibility controls.
- **Select:** let users select rows or cells for a follow-up action.
- **FixedHeader:** keep a header visible while scrolling a long table.
- **RowGroup:** organize rows under a shared value.
- **SearchBuilder or ColumnControl:** provide richer filter controls.

Each extension needs its own package or download-builder selection. Check whether a feature is part of the free DataTables set or DataTables Plus before relying on it. Extensions and styling integrations also have their own compatible releases.

## Style integrations

DataTables provides styling integrations for frameworks such as Bootstrap. Use the package that matches both the DataTables extension and the framework version in the project.

~~~js
import DataTable from "datatables.net-bs5";
import "datatables.net-responsive-bs5";
~~~

The package names in this example use Bootstrap 5 styling. Follow the selected integration's install instructions and do not load two competing DataTables theme stylesheets.

## Recalculate after a hidden container becomes visible

A table initialized inside a hidden tab or dialog may measure its columns before it has a visible width. When the container opens, ask DataTables to recalculate widths:

~~~js
modalElement.addEventListener("shown.bs.modal", () => {
    ordersTable.columns.adjust();
    ordersTable.responsive.recalc();
});
~~~

The modal event name above is from Bootstrap. For another tab or dialog library, use its event that fires after the container is visible. Only call Responsive methods if the extension is installed.

## Test responsive behavior with real content

Check the table with long names, missing values, localized labels, narrow and wide containers, and zoom. Inspect which columns remain visible and whether hidden values appear in the details area.

Avoid using CSS alone to hide DataTables columns. DataTables and its extensions need to know which columns are present so search, ordering, accessibility, and details behavior remain consistent.

## Responsive and extension checklist

- Important identifiers and statuses remain easy to scan.
- Hidden column values are available in a clear details view when needed.
- Dense comparison tasks have an intentional horizontal-scroll or detail design.
- Expand controls are keyboard accessible and clearly labeled.
- Extension CSS and JavaScript come from compatible builds.
- Only extensions that support a real user task are included.
- Licensing is checked for every feature used.
- Tables inside hidden panels recalculate their widths after opening.

## Further reading

- [Responsive extension manual](https://datatables.net/manual/extensions/responsive)
- [Responsive installation](https://datatables.net/manual/extensions/responsive/installation)
- [Responsive column priority](https://datatables.net/extensions/responsive/priority)
- [Responsive details views](https://datatables.net/manual/extensions/responsive/details-views)
- [DataTables styling frameworks](https://datatables.net/manual/styling/frameworks)
- [DataTables download builder](https://datatables.net/download/)
