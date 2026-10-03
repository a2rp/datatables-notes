# 13. Buttons, export, and layout

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Responsive tables and extensions](./12-responsive-tables-and-extensions.md) | [Notes index](../README.md) | [Next: State saving and accessibility](./14-state-saving-and-accessibility.md) |

## Add Buttons only when they support a user task

The Buttons extension can copy table data, export files, open a print view, and control column visibility. It adds controls to the DataTables `layout` system.

The download builder can include the Buttons features and styling required by the project. For a package-managed project, install the default styling package:

~~~sh
npm install datatables.net-buttons-dt
~~~

The extension is modular. Load only the button definitions and dependencies that the table uses.

## Place buttons with layout

The `layout` option controls where table features appear. Place related actions together and choose button labels that make the output clear.

~~~js
const ordersTable = new DataTable("#ordersTable", {
    layout: {
        topStart: {
            buttons: [
                "copy",
                "csv",
                "excel",
                "print",
                "colvis",
            ],
        },
    },
});
~~~

The string names select built-in button types. Buttons can also be configured as objects when the action needs a custom label, file name, or export behavior.

## Configure export columns and file names

Choose whether the export includes all columns or only visible columns:

~~~js
const exportButtons = [
    {
        extend: "csvHtml5",
        text: "Download CSV",
        filename: "orders",
        exportOptions: {
            columns: ":visible",
        },
    },
    {
        extend: "excelHtml5",
        text: "Download Excel",
        filename: "orders",
        exportOptions: {
            columns: ":visible",
        },
    },
];

const ordersTable = new DataTable("#ordersTable", {
    layout: {
        topStart: {
            buttons: exportButtons,
        },
    },
});
~~~

On a client-side table, export options can use the current search and ordering state. Check whether the desired export should include all pages or only the current page, and whether hidden columns should be included.

Excel output requires JSZip. PDF output requires PDFMake and a suitable font file. The download builder can include these dependencies. Excel HTML5 export creates an XLSX file, but it does not preserve the table's visual styling.

## Load modular button files with ES modules

A project using a bundler can import only the required button definitions:

~~~js
import DataTable from "datatables.net-dt";
import "datatables.net-buttons-dt";
import "datatables.net-buttons/js/buttons.html5.mjs";
import JSZip from "jszip";

DataTable.Buttons.jszip(JSZip);
~~~

This registers the Buttons feature, HTML5 export definitions, and JSZip for Excel. Add the print or column visibility module when the table uses those button types. Use the current Buttons installation reference for package names and integration details.

## Control visible columns

The `colvis` button provides a menu for showing and hiding columns:

~~~js
const ordersTable = new DataTable("#ordersTable", {
    layout: {
        topStart: {
            buttons: [
                {
                    extend: "colvis",
                    text: "Choose columns",
                },
            ],
        },
    },
});
~~~

Keep essential identifying columns visible. If a user hides a field needed to identify or act on a record, provide another clear way to see it. After adding column visibility controls, verify the responsive details view still communicates hidden values correctly.

## Add a custom action

A custom button can call application code, such as opening a filtered report:

~~~js
const ordersTable = new DataTable("#ordersTable", {
    layout: {
        topStart: {
            buttons: [
                {
                    text: "Refresh orders",
                    action() {
                        ordersTable.ajax.reload(null, false);
                    },
                },
            ],
        },
    },
});
~~~

Keep custom actions specific and predictable. Give the button an accessible name and provide feedback if the action takes time or fails.

## Understand client-side export limits

For a client-side table, the browser has the records available for export. For a server-side table, the browser generally holds only the current page. A built-in export therefore cannot include every matching server record unless the application has loaded those records.

For a full report from a server-side table, create an authorized export endpoint that applies the same filters and permission rules. Keep the endpoint's sort and filter fields on a server-side allowlist. Do not trust a client export button to enforce access.

## Protect spreadsheet exports

CSV and spreadsheet files can interpret cell values that begin with formula characters as formulas. If untrusted data is included, define an export policy for values beginning with characters such as equals, plus, minus, or at-sign. Apply that policy in the export layer while preserving the original table data.

Check dates, currency, and long identifiers in the resulting file. Spreadsheet applications may auto-convert values, so a value that looks correct in the table can change when opened.

## Use print view intentionally

The print button opens a print-focused view of the table. Select columns and rows that make sense on paper, and check repeated headers, wide content, and page breaks. Do not rely on browser print alone for documents that require a fixed report format.

~~~js
const ordersTable = new DataTable("#ordersTable", {
    layout: {
        topStart: {
            buttons: [
                {
                    extend: "print",
                    text: "Print current results",
                    exportOptions: {
                        columns: ":visible",
                    },
                },
            ],
        },
    },
});
~~~

## Arrange table controls separately from table data

Use layout positions to keep search, page size, information, paging, and action buttons organized:

~~~js
const ordersTable = new DataTable("#ordersTable", {
    layout: {
        topStart: "pageLength",
        topEnd: {
            buttons: ["copy", "csv"],
        },
        bottomStart: "info",
        bottomEnd: "paging",
    },
});
~~~

On a narrow screen, check that controls wrap in a logical order. Provide enough spacing between controls and preserve clear keyboard focus.

## Buttons checklist

- Every button has a clear purpose and label.
- Export includes the columns and rows users expect.
- Excel and PDF dependencies are registered when those formats are used.
- Server-side exports use a protected server endpoint for the full result set.
- Untrusted values are handled safely for spreadsheet output.
- Print output is checked at paper widths and page breaks.
- Column visibility does not hide critical information.
- Layout controls remain usable at narrow widths and by keyboard.

## Further reading

- [Buttons installation](https://datatables.net/manual/extensions/buttons/installation)
- [Buttons usage and layout](https://datatables.net/manual/extensions/buttons/usage)
- [Built-in Buttons](https://datatables.net/manual/extensions/buttons/built-in)
- [DataTables layout option](https://datatables.net/reference/option/layout)
- [Buttons export options](https://datatables.net/extensions/buttons/examples/html5/columns.html)
