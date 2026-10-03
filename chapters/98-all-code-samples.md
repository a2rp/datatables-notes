# 98. All code samples

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Integration and production patterns](./16-integration-and-production-patterns.md) | [Notes index](../README.md) | [Next: Complete questions and answers](./99-complete-q-and-a.md) |

This appendix gathers runnable examples from the core chapters. Each section links back to the chapter that explains the idea and its tradeoffs.

## 01. Getting started and setup

[Read the chapter](./01-getting-started-and-setup.md)

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

~~~html
<script defer src="/assets/dataTables.js"></script>
<script defer src="/assets/staff-table.js"></script>
~~~

~~~js
import DataTable from "datatables.net-dt";
import "datatables.net-dt/css/dataTables.dataTables.css";

const table = new DataTable("#staffTable", {
    pageLength: 10,
});
~~~

~~~sh
npm install datatables.net-dt
~~~

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

~~~js
const tableElement = document.querySelector("#staffTable");
const table = new DataTable(tableElement);

table.search("Engineer").draw();
~~~

## 02. HTML table structure and initialization

[Read the chapter](./02-html-structure-and-initialization.md)

~~~html
<table id="ordersTable">
    <caption>Recent customer orders</caption>
    <thead>
        <tr>
            <th scope="col">Order</th>
            <th scope="col">Customer</th>
            <th scope="col">Status</th>
            <th scope="col">Total</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>1048</td>
            <td>Ravi Shah</td>
            <td>Shipped</td>
            <td>125.50</td>
        </tr>
        <tr>
            <td>1049</td>
            <td>Meera Das</td>
            <td>Processing</td>
            <td>84.00</td>
        </tr>
    </tbody>
</table>
~~~

~~~js
const ordersTable = new DataTable("#ordersTable");
~~~

~~~js
const ordersTable = new DataTable("#ordersTable", {
    pageLength: 10,
    order: [[0, "desc"]],
});

ordersTable.search("Processing").draw();
~~~

~~~js
const ordersTable = new DataTable("#ordersTable", {
    columns: [
        { name: "order", orderable: true },
        { name: "customer", searchable: true },
        { name: "status", searchable: true },
        { name: "total", className: "numeric-cell" },
    ],
});
~~~

~~~js
const ordersTable = new DataTable("#ordersTable", {
    columnDefs: [
        {
            targets: [2],
            orderable: false,
        },
        {
            targets: [3],
            className: "numeric-cell",
        },
    ],
});
~~~

~~~html
<table id="salesTable">
    <caption>Sales by product</caption>
    <thead>
        <tr>
            <th scope="col">Product</th>
            <th scope="col">Units</th>
            <th scope="col">Revenue</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Notebook</td>
            <td>12</td>
            <td>96.00</td>
        </tr>
        <tr>
            <td>Pen set</td>
            <td>8</td>
            <td>40.00</td>
        </tr>
    </tbody>
    <tfoot>
        <tr>
            <th scope="row">Visible total</th>
            <td></td>
            <td></td>
        </tr>
    </tfoot>
</table>
~~~

~~~html
<table id="teamTable">
    <caption>Team member contact details</caption>
    <thead>
        <tr>
            <th rowspan="2" scope="col">Name</th>
            <th colspan="2" scope="colgroup">Contact</th>
        </tr>
        <tr>
            <th scope="col">Email</th>
            <th scope="col">Office</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Ada Lovelace</td>
            <td>ada@example.test</td>
            <td>London</td>
        </tr>
    </tbody>
</table>
~~~

~~~js
function createOrdersTable() {
    const element = document.querySelector("#ordersTable");

    if (!element) {
        return null;
    }

    return new DataTable(element, {
        pageLength: 10,
    });
}

const ordersTable = createOrdersTable();
~~~

## 03. Data sources: DOM, arrays, and objects

[Read the chapter](./03-data-sources.md)

~~~html
<table id="peopleTable">
    <thead>
        <tr>
            <th scope="col">Name</th>
            <th scope="col">Department</th>
            <th scope="col">Office</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Ada Lovelace</td>
            <td>Engineering</td>
            <td>London</td>
        </tr>
        <tr>
            <td>Grace Hopper</td>
            <td>Research</td>
            <td>New York</td>
        </tr>
    </tbody>
</table>
~~~

~~~js
const peopleTable = new DataTable("#peopleTable");
~~~

~~~html
<table id="inventoryTable">
    <thead>
        <tr>
            <th scope="col">SKU</th>
            <th scope="col">Product</th>
            <th scope="col">Quantity</th>
        </tr>
    </thead>
</table>
~~~

~~~js
const inventoryRows = [
    ["BK-104", "Notebook", 32],
    ["PN-220", "Pen set", 18],
    ["FD-510", "Folder", 41],
];

const inventoryTable = new DataTable("#inventoryTable", {
    data: inventoryRows,
});
~~~

~~~html
<table id="staffTable">
    <thead>
        <tr>
            <th scope="col">Name</th>
            <th scope="col">Department</th>
            <th scope="col">Office</th>
        </tr>
    </thead>
</table>
~~~

~~~js
const staff = [
    {
        name: "Ada Lovelace",
        department: "Engineering",
        office: "London",
    },
    {
        name: "Grace Hopper",
        department: "Research",
        office: "New York",
    },
];

const staffTable = new DataTable("#staffTable", {
    data: staff,
    columns: [
        { data: "name" },
        { data: "department" },
        { data: "office" },
    ],
});
~~~

~~~js
const staff = [
    {
        name: "Ada Lovelace",
        role: {
            title: "Engineer",
            team: "Platform",
        },
    },
];

const staffTable = new DataTable("#staffTable", {
    data: staff,
    columns: [
        { data: "name" },
        { data: "role.title" },
        { data: "role.team" },
    ],
});
~~~

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

~~~js
const staffTable = new DataTable("#staffTable", {
    data: [],
    columns: [
        { data: "name" },
        { data: "department" },
        { data: "office" },
    ],
});

function showStaff(records) {
    staffTable.clear();
    staffTable.rows.add(records);
    staffTable.draw();
}
~~~

~~~js
staffTable.ajax.reload(null, false);
~~~

## 04. Columns, rendering, and data types

[Read the chapter](./04-columns-rendering-and-data-types.md)

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

~~~js
{
    data: "price",
    render: DataTable.render.number(null, null, 2, "$"),
}
~~~

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

~~~js
{
    data: "manager",
    defaultContent: "Not assigned",
}
~~~

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

~~~js
{
    data: "customerName",
    render: DataTable.render.text(),
}
~~~

## 05. Searching and filtering

[Read the chapter](./05-searching-and-filtering.md)

~~~js
const staffTable = new DataTable("#staffTable");

staffTable.search("London").draw();
~~~

~~~js
staffTable
    .search("Engineer", {
        caseInsensitive: false,
        smart: true,
    })
    .draw();
~~~

~~~js
staffTable.column(1).search("Engineering").draw();
~~~

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

~~~js
staffTable
    .columns([1, 2])
    .search("London")
    .draw();
~~~

~~~js
staffTable
    .column(1)
    .search("Engineering")
    .column(2)
    .search("London")
    .draw();
~~~

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

~~~js
staffTable.search.fixed("active-records", (searchText, row) => {
    return row.active === true;
});

staffTable.draw();
~~~

~~~js
staffTable.search.fixed("active-records", null).draw();
~~~

~~~js
staffTable.search("");
staffTable.columns().search("");
staffTable.search.fixed("active-records", null);
staffTable.draw();
~~~

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

## 06. Ordering and custom sorting

[Read the chapter](./06-ordering-and-sorting.md)

~~~js
const ordersTable = new DataTable("#ordersTable", {
    order: [
        [2, "desc"],
        [0, "asc"],
    ],
});
~~~

~~~js
ordersTable.order([[1, "asc"]]).draw();
~~~

~~~js
const ordersTable = new DataTable("#ordersTable", {
    columns: [
        { data: "orderNumber" },
        { data: "customer" },
        { data: "status" },
        { data: "actions", orderable: false, searchable: false },
    ],
});
~~~

~~~js
{
    data: "customer",
    orderSequence: ["asc", "desc"],
}
~~~

~~~js
const peopleTable = new DataTable("#peopleTable", {
    order: [
        [1, "asc"],
        [0, "asc"],
    ],
});
~~~

~~~html
<table id="priorityTable">
    <thead>
        <tr>
            <th scope="col">Task</th>
            <th scope="col">Priority</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Update records</td>
            <td data-order="3">Medium</td>
        </tr>
        <tr>
            <td>Resolve outage</td>
            <td data-order="1">High</td>
        </tr>
        <tr>
            <td>Review copy</td>
            <td data-order="4">Low</td>
        </tr>
    </tbody>
</table>
~~~

~~~js
const priorityTable = new DataTable("#priorityTable", {
    order: [[1, "asc"]],
});
~~~

~~~js
const accountTable = new DataTable("#accountTable", {
    columns: [
        { data: "accountNumber" },
        { data: "owner" },
        { data: "balance" },
        {
            data: "accountNumber",
            orderable: false,
            searchable: false,
            render(data, type) {
                return type === "display" ? "Open account" : data;
            },
        },
    ],
});
~~~

~~~js
const amountFormatter = new Intl.NumberFormat("en-US", {
    style: "currency",
    currency: "USD",
});

const invoiceTable = new DataTable("#invoiceTable", {
    columns: [
        { data: "invoiceNumber" },
        { data: "customer" },
        {
            data: "amount",
            render(data, type) {
                return type === "display"
                    ? amountFormatter.format(data)
                    : data;
            },
        },
    ],
});
~~~

## 07. Paging and page length

[Read the chapter](./07-paging-and-page-length.md)

~~~js
const ordersTable = new DataTable("#ordersTable", {
    pageLength: 25,
    lengthMenu: [10, 25, 50, 100],
});
~~~

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

~~~js
ordersTable.page(2).draw("page");
~~~

~~~js
ordersTable.page("next").draw("page");
ordersTable.page("previous").draw("page");
~~~

~~~js
ordersTable
    .page.len(50)
    .order([[1, "asc"]])
    .draw(false);
~~~

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

~~~js
const pageStatus = document.querySelector("#pageStatus");

ordersTable.on("draw", () => {
    const { page, pages } = ordersTable.page.info();
    pageStatus.textContent = pages === 0
        ? "No pages"
        : "Page " + (page + 1) + " of " + pages;
});
~~~

~~~js
ordersTable.search("processing").draw();
~~~

## 08. The API and table updates

[Read the chapter](./08-api-and-table-updates.md)

~~~js
const staffTable = new DataTable("#staffTable", {
    pageLength: 10,
    rowId: "id",
});
~~~

~~~js
const firstRow = staffTable.row(0).data();
const allRows = staffTable.rows().data().toArray();

console.log(firstRow);
console.log(allRows);
~~~

~~~js
const selectedRecord = staffTable.row("#staff-104").data();
~~~

~~~js
const matchingRows = staffTable
    .rows({ search: "applied" })
    .data()
    .toArray();
~~~

~~~js
staffTable.row.add({
    id: "staff-108",
    name: "Dorothy Vaughan",
    department: "Computing",
    office: "Hampton",
}).draw(false);
~~~

~~~js
const newRecords = [
    {
        id: "staff-109",
        name: "Annie Easley",
        department: "Computing",
        office: "Cleveland",
    },
    {
        id: "staff-110",
        name: "Katherine Johnson",
        department: "Mathematics",
        office: "Hampton",
    },
];

staffTable.rows.add(newRecords).draw(false);
~~~

~~~js
const updatedRecord = {
    id: "staff-104",
    name: "Ada Lovelace",
    department: "Engineering",
    office: "London",
};

staffTable
    .row("#staff-104")
    .data(updatedRecord)
    .draw(false);
~~~

~~~js
staffTable
    .cell("#staff-104", 2)
    .data("Platform")
    .draw(false);
~~~

~~~js
staffTable
    .row("#staff-104")
    .remove()
    .draw(false);
~~~

~~~js
staffTable
    .rows((index, data) => data.expired === true)
    .remove()
    .draw();
~~~

~~~js
function replaceStaff(records) {
    staffTable.clear();
    staffTable.rows.add(records);
    staffTable.draw();
}
~~~

~~~js
staffTable.ajax.reload();
~~~

~~~js
staffTable.ajax.reload(null, false);
~~~

~~~js
staffTable.search("Hampton");
staffTable.order([[0, "asc"]]);
staffTable.page.len(25);
staffTable.draw();
~~~

~~~js
staffTable.destroy();

const rebuiltTable = new DataTable("#staffTable", {
    pageLength: 25,
    columns: [
        { data: "name" },
        { data: "department" },
        { data: "office" },
    ],
});
~~~

~~~js
const tableElement = document.querySelector("#staffTable");

if (!DataTable.isDataTable(tableElement)) {
    new DataTable(tableElement);
}
~~~

## 09. Events and lifecycle

[Read the chapter](./09-events-and-lifecycle.md)

~~~js
const staffTable = new DataTable("#staffTable", {
    ajax: "/api/staff",
    on: {
        xhr(event, settings, json) {
            console.log("Staff response received", json);
        },
        draw() {
            console.log("Staff rows redrawn");
        },
    },
});
~~~

~~~js
const staffTable = new DataTable("#staffTable");

function reportDraw() {
    const info = staffTable.page.info();
    console.log("Visible rows:", info.end - info.start);
}

staffTable.on("draw", reportDraw);
~~~

~~~js
staffTable.off("draw", reportDraw);
~~~

~~~html
<p id="pageSummary" aria-live="polite"></p>
~~~

~~~js
const pageSummary = document.querySelector("#pageSummary");

function updatePageSummary() {
    const { page, pages, recordsDisplay } = staffTable.page.info();

    pageSummary.textContent = recordsDisplay === 0
        ? "No matching records"
        : "Page " + (page + 1) + " of " + pages;
}

staffTable.on("draw", updatePageSummary);
updatePageSummary();
~~~

~~~html
<tbody id="staffRows">
    <tr id="staff-104">
        <td>Ada Lovelace</td>
        <td>Engineering</td>
        <td><button type="button" data-action="open">Open</button></td>
    </tr>
</tbody>
~~~

~~~js
const staffRows = document.querySelector("#staffRows");

staffRows.addEventListener("click", (event) => {
    if (!(event.target instanceof Element)) {
        return;
    }

    const button = event.target.closest("[data-action='open']");
    if (!button) {
        return;
    }

    const row = staffTable.row(button.closest("tr"));
    const record = row.data();

    console.log("Open staff record:", record.id);
});
~~~

~~~js
const auditTable = new DataTable("#auditTable", {
    ajax: "/api/audit",
    on: {
        preXhr() {
            document.querySelector("#loading").hidden = false;
        },
        xhr(event, settings, json) {
            document.querySelector("#loading").hidden = true;

            if (!json) {
                document.querySelector("#loadError").hidden = false;
            }
        },
    },
});
~~~

~~~js
staffTable.on("draw", () => {
    const visibleCount = staffTable.rows({ page: "current" }).count();
    document.querySelector("#visibleCount").textContent =
        String(visibleCount);
});
~~~

~~~js
function mountStaffTable(element) {
    const table = new DataTable(element, {
        pageLength: 10,
    });

    const onDraw = () => {
        console.log("Current page:", table.page.info().page + 1);
    };

    table.on("draw", onDraw);

    return function unmountStaffTable() {
        table.off("draw", onDraw);
        table.destroy();
    };
}
~~~

## 10. Ajax loading and JSON

[Read the chapter](./10-ajax-loading-and-json.md)

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

~~~js
refreshButton.addEventListener("click", () => {
    staffTable.ajax.reload(null, false);
});
~~~

~~~js
departmentFilter.addEventListener("change", () => {
    staffTable.ajax.reload();
});
~~~

~~~json
{
    "data": []
}
~~~

## 11. Server-side processing

[Read the chapter](./11-server-side-processing.md)

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

## 12. Responsive tables and extensions

[Read the chapter](./12-responsive-tables-and-extensions.md)

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

~~~js
const metricsTable = new DataTable("#metricsTable", {
    scrollX: true,
});
~~~

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

~~~js
import DataTable from "datatables.net-bs5";
import "datatables.net-responsive-bs5";
~~~

~~~js
modalElement.addEventListener("shown.bs.modal", () => {
    ordersTable.columns.adjust();
    ordersTable.responsive.recalc();
});
~~~

## 13. Buttons, export, and layout

[Read the chapter](./13-buttons-export-and-layout.md)

~~~sh
npm install datatables.net-buttons-dt
~~~

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

~~~js
import DataTable from "datatables.net-dt";
import "datatables.net-buttons-dt";
import "datatables.net-buttons/js/buttons.html5.mjs";
import JSZip from "jszip";

DataTable.Buttons.jszip(JSZip);
~~~

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

## 14. State saving and accessibility

[Read the chapter](./14-state-saving-and-accessibility.md)

~~~js
const ordersTable = new DataTable("#ordersTable", {
    stateSave: true,
});
~~~

~~~js
const ordersTable = new DataTable("#ordersTable", {
    stateSave: true,
    stateDuration: 60 * 60 * 24,
});
~~~

~~~js
const ordersTable = new DataTable("#ordersTable", {
    stateSave: true,
    stateDuration: -1,

    stateSaveParams(settings, data) {
        data.search.search = "";

        for (const column of data.columns) {
            column.search.search = "";
        }
    },
});
~~~

~~~html
<table id="ordersTable">
    <caption>Recent orders</caption>
    <thead>
        <tr>
            <th scope="col">Order number</th>
            <th scope="col">Customer</th>
            <th scope="col">Status</th>
            <th scope="col">Total</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <th scope="row">ORD-1042</th>
            <td>Riya Shah</td>
            <td>Processing</td>
            <td>Rs 2,450</td>
        </tr>
    </tbody>
</table>
~~~

~~~js
const ordersTable = new DataTable("#ordersTable", {
    tabIndex: 0,
    language: {
        search: "Search orders:",
        emptyTable: "No orders are available.",
        zeroRecords: "No orders match the current search.",
    },
});
~~~

~~~css
.dt-container :focus-visible {
    outline: 3px solid #ff6b35;
    outline-offset: 2px;
}
~~~

~~~html
<label for="ordersCustomerFilter">Filter by customer</label>
<input id="ordersCustomerFilter" type="search">
~~~

~~~js
const ordersTable = new DataTable("#ordersTable");

document
    .querySelector("#ordersCustomerFilter")
    .addEventListener("input", (event) => {
        ordersTable
            .column(1)
            .search(event.currentTarget.value)
            .draw();
    });
~~~

~~~html
<p id="ordersStatus" role="status" aria-live="polite"></p>
~~~

~~~js
const ordersTable = new DataTable("#ordersTable");
const ordersStatus = document.querySelector("#ordersStatus");

function updateOrdersStatus() {
    const page = ordersTable.page.info();
    ordersStatus.textContent =
        `${page.recordsDisplay} matching orders.`;
}

ordersTable.on("draw", updateOrdersStatus);
updateOrdersStatus();
~~~

## 15. Security, performance, and debugging

[Read the chapter](./15-security-performance-and-debugging.md)

~~~js
const customersTable = new DataTable("#customersTable", {
    data: customers,
    columns: [
        {
            data: "name",
            render: DataTable.render.text(),
        },
        {
            data: "email",
            render: DataTable.render.text(),
        },
    ],
});
~~~

~~~js
const productsTable = new DataTable("#productsTable", {
    ajax: "/api/products",
    deferRender: true,
    pageLength: 25,
});
~~~

~~~js
const productsTable = new DataTable("#productsTable", {
    ajax: "/api/products",
});

productsTable.on("click", "tbody button[data-action='open']", function (event) {
    const row = productsTable.row(event.target.closest("tr")).data();
    openProduct(row.id);
});
~~~

~~~js
productsTable
    .rows.add(newProducts)
    .draw(false);
~~~

~~~js
productsTable.ajax.reload(null, false);
~~~

~~~js
const tableElement = document.querySelector("#ordersTable");

if (!DataTable.isDataTable(tableElement)) {
    new DataTable(tableElement, {
        pageLength: 25,
    });
}
~~~

~~~js
const ordersTable = new DataTable("#ordersTable", {
    ajax: {
        url: "/api/orders",
        dataSrc: "results.orders",
    },
});
~~~

## 16. Integrating and running in production

[Read the chapter](./16-integration-and-production-patterns.md)

~~~sh
npm install datatables.net-dt
~~~

~~~js
import DataTable from "datatables.net-dt";
import "datatables.net-dt/css/dataTables.dataTables.css";
~~~

~~~sh
npm install datatables.net-responsive-dt
~~~

~~~js
import "datatables.net-responsive-dt";
~~~

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

