# 01. Getting started and setup

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| Start of notes | [Notes index](../README.md) | [Next: HTML table structure and initialization](./02-html-structure-and-initialization.md) |

## What DataTables adds

DataTables enhances an HTML table with controls for searching, ordering, paging, and displaying records. The table remains ordinary HTML, so its structure and data should make sense before JavaScript runs.

The library manages table interactions in the browser. It does not replace server-side authorization, input validation, or database access rules. Those responsibilities belong to the application.

As of 4 October 2026, the DataTables CDN lists version 3.1.3 as the current stable release. These notes use the DataTables 3 JavaScript constructor. Check the current download builder when starting a project because releases and extension versions change.

## Choose an installation method

The core files can be loaded from the DataTables CDN, installed with a package manager, or hosted locally. Use the official download builder to select the core, styling integration, and extensions that the project needs. Keep the JavaScript and CSS versions aligned.

A CDN is convenient for a small page or quick experiment. NPM fits an application that already uses a bundler. Local files are useful when the application must serve its own assets.

## Load the CDN files

This complete page uses a semantic table with static rows. The stylesheet and script use the current version verified above. The script appears after the table, so the table element exists when initialization runs.

~~~html
<!doctype html>
<html lang="en">
    <head>
        <meta charset="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
        <title>Staff directory</title>
        <link
            rel="stylesheet"
            href="https://cdn.datatables.net/3.1.3/css/dataTables.dataTables.css"
        />
    </head>
    <body>
        <main>
            <h1>Staff directory</h1>

            <table id="staffTable">
                <caption>Current team members</caption>
                <thead>
                    <tr>
                        <th scope="col">Name</th>
                        <th scope="col">Role</th>
                        <th scope="col">Office</th>
                    </tr>
                </thead>
                <tbody>
                    <tr>
                        <td>Ada Lovelace</td>
                        <td>Engineer</td>
                        <td>London</td>
                    </tr>
                    <tr>
                        <td>Grace Hopper</td>
                        <td>Analyst</td>
                        <td>New York</td>
                    </tr>
                    <tr>
                        <td>Katherine Johnson</td>
                        <td>Mathematician</td>
                        <td>Hampton</td>
                    </tr>
                    <tr>
                        <td>Margaret Hamilton</td>
                        <td>Software engineer</td>
                        <td>Cambridge</td>
                    </tr>
                    <tr>
                        <td>Mary Jackson</td>
                        <td>Engineer</td>
                        <td>Hampton</td>
                    </tr>
                </tbody>
            </table>
        </main>

        <script src="https://cdn.datatables.net/3.1.3/js/dataTables.js"></script>
        <script>
            const staffTable = new DataTable("#staffTable", {
                pageLength: 5,
                order: [[0, "asc"]],
            });
        </script>
    </body>
</html>
~~~

The constructor finds the table and returns an API instance. The instance is stored in `staffTable` so application code can later search, change pages, or update rows.

## Initialize after the table exists

When a regular script is placed after the table in the document, the browser has already parsed the table before it runs. If the script is loaded from the document head, use `defer` or wait for the document to be ready.

~~~html
<script defer src="/assets/dataTables.js"></script>
<script defer src="/assets/staff-table.js"></script>
~~~

The external files must be loaded in the intended order. The DataTables library has to be available before code that calls `new DataTable()`. A missing or blocked script causes a reference error in the browser console.

For a page that loads a module, module scripts are deferred by default:

~~~js
import DataTable from "datatables.net-dt";
import "datatables.net-dt/css/dataTables.dataTables.css";

const table = new DataTable("#staffTable", {
    pageLength: 10,
});
~~~

Install the default styling package with:

~~~sh
npm install datatables.net-dt
~~~

The package manager installs the version selected by the project and its lockfile. Review the current DataTables installation instructions for the package names required by a styling framework or extension.

## DataTables 3 does not require jQuery

DataTables 3 can run without jQuery. Native JavaScript is enough for a new page, and the constructor shown here is the direct API.

Existing applications can continue using jQuery when it is already part of their design. Avoid adding jQuery only to initialize a new DataTables 3 table. Projects on older DataTables releases may have different dependencies and APIs, so identify the installed version before copying an example.

## Configure a small table

The second constructor argument is an options object. Add an option only when it improves the table for its users.

~~~js
const ordersTable = new DataTable("#ordersTable", {
    pageLength: 10,
    lengthMenu: [10, 25, 50],
    order: [[2, "desc"]],
    searching: true,
    ordering: true,
    paging: true,
});
~~~

- `pageLength` chooses how many records are shown on the first page.
- `lengthMenu` offers page-size choices.
- `order` identifies the column index and direction used for initial ordering.
- `searching`, `ordering`, and `paging` turn the corresponding features on or off.

The defaults already provide common controls. Keep configuration concise so the behavior is easy to understand.

## What initialization does

When a table is initialized, DataTables reads its header and available rows, prepares the configured features, and draws the first view. It also adds interface controls around the table. Keep the original table markup valid so the generated interface can work reliably.

Initialize a table once. Re-running the constructor on the same element can produce a warning because the table already has an instance. Keep the returned API instance when later code needs to control the table.

~~~js
const tableElement = document.querySelector("#staffTable");
const table = new DataTable(tableElement);

table.search("Engineer").draw();
~~~

If a table's columns or fundamental configuration must change, plan a deliberate destroy and rebuild. For routine data refreshes, use the API rather than replacing the entire table element.

## Keep the table useful before enhancement

Use a caption to describe the table. Put column headings in a `thead` row with `th` cells and use `scope="col"` for ordinary column headers. Keep body rows consistent with the header columns. DataTables does not support `colspan` or `rowspan` in body rows.

A table should still communicate its content when JavaScript is delayed or unavailable. For remote data, provide a clear loading and error state in the application around the table.

## Common setup problems

- **The table is unchanged:** check that the script loaded successfully and the selector matches the table ID.
- **The console reports DataTable is not defined:** load the library before the initialization code and check the network request.
- **The table initializes twice:** centralize initialization and keep the returned API instance.
- **A column count warning appears:** make the number of body cells match the table headers and column configuration.
- **Styles look broken:** confirm the DataTables CSS file and JavaScript are compatible and both loaded.
- **Old examples behave differently:** check which major version the project uses. Do not assume legacy jQuery examples describe DataTables 3.

## Setup checklist

Before adding table features, confirm:

- The table has a unique selector and valid header and body markup.
- The chosen DataTables CSS and JavaScript files load successfully.
- The library version is known and the files are from compatible releases.
- Initialization happens after the table exists and runs only once.
- The starting table is understandable without relying on generated controls.
- New code uses the constructor and API for the installed major version.

## Further reading

- [DataTables installation](https://datatables.net/manual/core/installation)
- [DataTables usage and options](https://datatables.net/manual/core/usage)
- [DataTables CDN and current release](https://cdn.datatables.net/)
- [New in DataTables 3](https://datatables.net/download/upgrade/core/3.0/new)
- [Upgrading to DataTables 3](https://datatables.net/download/upgrade/core/3.0/upgrade)
