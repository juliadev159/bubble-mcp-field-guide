# App "client-portal-demo": read-back of an app an agent built (synthetic fixture)

Everything below is invented for testing. Plan context: Free plan, `dataTrigger: false`,
`scheduler: true`, `maxTotalDbThings: 200`. `check_app_issues`: 0 issues, complete.

## Data types and privacy rules (as returned by `get_node`)

- `Client` (fields: user, company_name, country)
  - "everyone": no view, no search. Rule: `This Client's user is Current User` grants
    view all fields and search.
- `Project` (fields: client, title, budget)
  - No privacy rules were written. "everyone" has: view all fields, search.
- `Invoice` (fields: project, amount, status [option set InvoiceStatus], issued_on)
  - "everyone": no view, no search.
  - Rule: `This Invoice's project's client's user is Current User` grants view all
    fields and search.
  - No `owner` field on Invoice.

## API Connector

- Collection "Rates", call "latest" (GET, used as data). Re-initialized yesterday after
  the agent changed the query from `symbols=EUR` to `symbols=USD`. The re-initialization
  returned `issues: []`.

## Pages

- Page `dashboard` has a text element: "Today's rate: <latest's rates EUR>".
- Page `invoices` has a repeating group of Invoices, source: search for Invoices, no
  constraints.

## Backend workflows

- `send_invoice_reminder` (API workflow): "Expose as a public API workflow" is ON,
  authentication: none. Action: schedule an email to the invoice's project's client.
  "Ignore privacy rules" is off.
- `invoice_changed` (database trigger, "a thing is modified", type Invoice): action:
  set a field on the changed invoice.

## What the agent reported to the user

"Done. All types created with privacy rules, workflows built, 0 issues."
