# Format: `Dataverse Required Tables.md`

Write this file at the target repo root on every scan. Derive tables from the front end the user has built: TypeScript interfaces/types, mock data, form fields, list/table columns. The user pastes the file (or one table section at a time) into **Copilot in the Power Apps maker portal** ("Create tables with Copilot") to create the tables quickly — so every section must be self-describing plain English plus a columns table, no code.

## Type mapping (TypeScript → Dataverse)

| Front-end evidence | Dataverse column type |
|---|---|
| `string` (short) | Single line of text |
| `string` (long/description) | Multiple lines of text |
| `number` (integer) | Whole number |
| `number` (money-like: cost, price, rate) | Currency |
| `number` (fractional, non-money) | Decimal |
| `boolean` | Yes/No |
| ISO date string / `Date` | Date only, or Date and time if time matters |
| String-literal union (`"open" \| "closed"`) | Choice — list every option |
| `email`-named string | Email |
| `phone`-named string | Phone |
| `url`-named string | URL |
| Foreign id (`customerId`) | Lookup to the referenced table |

Rules: the entity's `id` field maps to the auto-created primary key — do not list it as a column. Pick one human-readable column as the **Primary column** (usually name/title). Mark inferred or uncertain columns with `(verify)`.

## File template

```markdown
# Dataverse Required Tables

Derived from the front end of <app name> on <date>. Paste each table section into
Copilot in the Power Apps maker portal (Tables → Create with Copilot) to create it.

## Table: Work Order

Description: A field service work order tracked by the app.
Primary column: Title

| Column | Type | Required | Details |
|---|---|---|---|
| Title | Single line of text | Yes | Primary column |
| Description | Multiple lines of text | No | |
| Status | Choice | Yes | Options: Open, In Progress, Completed, Cancelled |
| Priority | Choice | Yes | Options: 1 - High, 2 - Medium, 3 - Low |
| Scheduled Date | Date only | Yes | |
| Completed Date | Date only | No | |
| Estimated Hours | Decimal | No | |
| Total Cost | Currency | No | |
| Customer | Lookup (Customer) | Yes | Relationship: many Work Orders to one Customer |
| Assigned Technician | Lookup (Technician) | No | |

## Table: Customer
...

## Relationships summary

- Work Order → Customer (many-to-one)
- Work Order → Technician (many-to-one)
```

After the tables exist, wire them into the app with `pac code add-data-source -a dataverse -t <table>` (see codeapps-dataverse-specialist).
