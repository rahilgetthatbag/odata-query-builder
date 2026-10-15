# OData UI5 Query Builder

A visual, single-file tool for building OData queries. Load a service's `$metadata`, pick fields, filters and expands from dropdowns, and get the finished URL plus ready-to-paste UI5 code. It handles both OData V2 and V4 and reads SAP Gateway annotations.

No build step, no dependencies, no network calls. Open `index.html` and it works.

<!-- Add a screenshot or GIF here: docs/screenshot.png -->

## Features

- **Metadata parsing:** paste `$metadata` XML, upload an `.xml` file, or use the built-in V2 and V4 samples. The OData version is detected automatically.
- **Query controls:** entity set, `$select`, `$filter` rows (field, operator, value, AND/OR), `$expand`, `$orderby`, `$top`, `$skip` and count.
- **Live output:** the result updates as you click. Copy it as:
  - a plain URL
  - a `curl` command
  - a `fetch` call
  - UI5 code (`oModel.read` for V2, `bindList` + `requestContexts` for V4)
- **Type-aware value inputs:** date pickers for dates, number fields for numerics, a dropdown for booleans, and correct quoting and escaping for strings.
- **SAP-aware:** reads `sap:label`, `sap:filterable` and `sap:sortable`. Labels are shown in the field lists, non-filterable fields are left out of filter rows, and non-sortable fields are left out of `$orderby`.

## Quick start

1. Download or clone the repo.
2. Open `index.html` in a browser.
3. Click **Sample V2 (SAP)** or **Sample V4**, or paste your own `$metadata`.
4. Build your query and copy the output from the tabs on the right.

To get `$metadata` from a service, open `<service root>/$metadata` in your browser and save the page as `.xml`. Saving the file avoids cross-origin issues.

## V2 and V4 differences handled

| Concern | OData V2 | OData V4 |
|---|---|---|
| Count | `$inlinecount=allpages` | `$count=true` |
| Text search | `substringof('x',Name)` | `contains(Name,'x')` |
| DateTime literal | `datetime'2026-10-14T00:00:00'` | `2026-10-14T00:00:00` |
| GUID literal | `guid'...'` | bare GUID |
| Decimal / Int64 | `20.5M` / `10L` | plain number |
| Navigation targets | resolved through `Association` elements | direct `Type` attribute |
| UI5 output | `oModel.read(path, { filters, sorters, urlParameters })` | `bindList(path, ctx, sorters, filters, params)` |

## Example output

For `Products`, with `$select=ProductID,ProductName`, `UnitPrice gt 20`, `$expand=Category`, ordered by `ProductName`, top 10, on a V2 service:

```
/Products?$select=ProductID,ProductName&$filter=UnitPrice gt 20M&$expand=Category&$orderby=ProductName&$top=10
```

```js
oModel.read("/Products", {
  filters: [
    new Filter("UnitPrice", FilterOperator.GT, 20)
  ],
  sorters: [new Sorter("ProductName", false)],
  urlParameters: { "$select": "ProductID,ProductName", "$expand": "Category", "$top": 10 }
});
```

## How it works

The query is held in one state object, and every control edits it. The generators are pure functions that take plain data and return text, so they are independent of the UI:

- `parseMeta(xml)`: DOMParser-based EDMX parser (one code path per OData version where they differ)
- `buildUrl(state, version, base, types)`
- `toUi5V2(state, types)` and `toUi5V4(state, types)`
- `toFetch(url)` and `toCurl(url)`
- `lit(type, value, version)`: OData literal formatting

## Limitations

- No live query execution yet. Browsers block cross-origin requests to services that do not send CORS headers, so execution needs a proxy or a service with CORS enabled.
- Filters are a flat list joined by one AND or OR. Nested groups are not supported.
- `$expand` supports first-level navigation properties only.
- Complex types and function imports are not parsed.
- Not yet tested against a wide range of real SAP services.

## Roadmap

- [ ] Node test suite for the generator functions
- [ ] Live execution through a small proxy, with a results table
- [ ] Saved queries (browser storage)
- [ ] Nested `$expand` and `$select` inside `$expand`
- [ ] Paging through results
- [ ] Optional Vite setup if the file outgrows a single page

## Test services

- OData V2: `https://services.odata.org/V2/Northwind/Northwind.svc`
- OData V4: `https://services.odata.org/v4/northwind/northwind.svc`
- OData V4 TripPin: `https://services.odata.org/V4/TripPinServiceRW`
- SAP APIs with downloadable EDMX: SAP Business Accelerator Hub (api.sap.com)

## Disclaimer

This is an independent project and is not affiliated with or endorsed by SAP SE. SAP, SAP Fiori and SAPUI5 are trademarks of SAP SE.

## License

MIT
