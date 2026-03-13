# Data-Dashboard---Codex
For the data visualization project using codex
# Codex Master Prompt — Catoctin Mtn Growers Buyer Dashboard

You are building a **local-first, production-shaped prototype** for **Catoctin Mtn Growers**.

Your job is to create a stable, secure, maintainable customer-facing dashboard application that will later be shared with real grocery-chain buyers. This is not a throwaway demo. Build it as a strong prototype with production-ready architecture decisions.

## Primary Objective

Build a web application that allows:

- **Internal Catoctin Mtn Growers users** to import customer sales files and shipment files, manage mappings, monitor alerts, and administer users
- **Customer buyers** to securely log in and view only their own company’s dashboard data
- Buyers to monitor plant item performance across stores using sales and shipment data
- Buyers to export PDF and CSV/Excel reports
- Internal users to receive alerts when sell-through exceeds thresholds

The app must be:
- secure
- auditable
- stable
- easy to extend
- cleanly organized
- suitable for local development first, with a future path to cloud deployment

---

## Required Tech Stack

Use this stack unless there is a compelling technical reason not to:

- **Next.js**
- **TypeScript**
- **React**
- **Supabase**
  - Postgres
  - Auth
  - Storage
- Tailwind CSS for UI styling
- A robust charting library suitable for dashboards
- A reliable PDF export approach
- Structured test coverage

Target architecture:
- local-first development
- deployment-ready later for **Vercel + Supabase**

Do not overengineer infrastructure in phase 1.

---

## Users and Roles

Implement these roles:

### 1. Super Admin
Initial super admin:
- **Tyler Van Wingerden**

Capabilities:
- create users
- disable users
- reset passwords or trigger reset flow
- manage item mappings
- manage store mappings
- approve imports
- rollback imports
- view import logs
- configure alert recipients

### 2. Internal User
Capabilities:
- access all customer data
- upload/import files
- view dashboards for all customers
- manage mappings if permission is granted
- view alerts
- view logs

### 3. Customer Buyer
Capabilities:
- log in with email/password
- see only their own customer’s data
- access dashboard views
- export PDF snapshots
- export CSV/Excel files

### Security Rule
**Customer users must only be able to see their own company’s data.**
This must be enforced at the database and server level, not just in the UI.

Use a multi-tenant design with strong authorization.

---

## Customers in Prototype Scope

Seed or support these customers:

- Safeway
- Aldi
- Giant of Landover
- Food Lion
- Sam's Club

Each customer may have a different sales file format.

---

## Business Context

Catoctin Mtn Growers supplies plant items to grocery chains and big box stores.

The dashboard combines:

### 1. Customer Sales Data
- arrives as **Excel attachments**
- each customer has its own file layout
- fields typically include:
  - sales date
  - customer item number
  - item description
  - store number
  - units sold
  - dollars sold

### 2. CMG Shipment Data from SBI
Exported from SBI as a flat file with fields such as:
- ship date
- customer name (bill to)
- CMG item number
- plant description
- shipped quantity
- shipped cost dollars
- store number

### Mapping Requirements
Because identifiers do not align across systems, the app needs:

#### Item Mapping Table
Maps:
- customer item number -> CMG item number

Rules:
- one CMG item can map to multiple customer item numbers
- mapping is manual
- mapping does not change over time

#### Store Mapping Table
Maps:
- customer store number -> CMG store number -> canonical store

Also includes:
- region

Store mapping is required because customer sales store numbers and shipment store numbers will not always match.

---

## Development Phasing

## Phase 1 — Build This Now
Build these features first:

- authentication
- role-based access
- customer data isolation
- branded dashboard
- manual upload of customer sales files
- manual upload of shipment files
- manual item mapping management
- manual store mapping management
- import validation
- import approval workflow
- import logs
- rollback support
- dashboard metrics and charts
- PDF export
- CSV/Excel export
- internal alerting
- tests
- clear documentation

## Phase 2 — Do Not Build Yet
Leave structured hooks for:
- automatic email attachment ingestion for customer sales files
- automatic Google Drive ingestion for shipment files

Do not implement automation yet unless explicitly asked later.

---

## Core Functional Requirements

## Dashboard Requirements

Customer buyers should land on a **homepage dashboard** with high-level insights first.

The dashboard must help answer these questions:

1. What is my current year-over-year sales trend?
2. What is my current sell-through?
3. Which items are my top performers?
4. Which stores are my top performers?
5. What is the value of current inventory?

### Dashboard hierarchy
Use this reporting hierarchy:
- Category
- Item Size
- Item Description

### Dashboard grain
Must support analysis by:
- item
- store
- day

### Filters
Support:
- date range
- region
- store
- category
- item size
- item description / item

Internal users can also filter by customer.

### Default date range
Use:
- **current season to date**

If practical, make season configurable in admin settings.

---

## Required Metrics

Implement these metrics:

1. Units Sold
2. Sales Dollars
3. Sell-Through
4. Weeks of Supply
5. Net Margin
6. Year-over-Year Comp
7. Top Performing Items
8. Top Performing Stores
9. Current Inventory Value

### Metric Definitions

#### Sell-Through
Use:
- cumulative units sold / cumulative units shipped

Support calculation at:
- item
- store
- customer
- region

#### Inventory
Customers do not send inventory.
Estimate inventory using:

- cumulative shipments - cumulative sales

Assume shrink, damages, returns, and transfers are not material enough to matter for phase 1.

#### Inventory Value
Use:
- **estimated retail value on hand**

Derive it using estimated units on hand multiplied by an average selling price basis from customer sales data.

Suggested implementation:
- avg_sales_price_per_unit = sales_dollars / units_sold over an appropriate historical scope
- estimated_retail_value_on_hand = estimated_units_on_hand * avg_sales_price_per_unit

Document the chosen formula clearly in code and README.

Prevent obviously broken display values. If inventory becomes negative, clamp to zero or clearly flag the condition.

#### Weeks of Supply
Use estimated inventory divided by an average recent sales rate.
Make the formula explicit and easy to adjust later.

#### Net Margin
For version 1 define:
- customer total sales dollars minus total shipped cost dollars

#### YOY Comparison
Use:
- **same week last year**
- align by weekday so comparisons reflect similar shopping days, e.g. Friday vs Friday

Do not use simple same calendar date comparisons if weekday alignment is available.

---

## Alerts

Implement internal alerts for:
- store/item combinations where sell-through exceeds 90%

### Alert formula
- cumulative units sold / cumulative units shipped to that store for that item
- measured over the full season

### Alert delivery
- show alert inside the app
- send email to internal recipients

Admins must be able to:
- manage recipient emails
- enable/disable alerts
- avoid duplicate repeated alert spam for the same condition

---

## Import System Requirements

Build imports as a reliable pipeline, not one-off scripts.

## Customer Sales Imports
Support:
- manual upload in phase 1
- customer-specific parsing logic because each customer layout differs

Required fields to parse:
- sales date
- customer item number
- item description
- store number
- units sold
- dollars sold

## Shipment Imports
Support:
- manual upload in phase 1

Required fields:
- ship date
- customer name / bill to
- CMG item number
- plant description
- shipped quantity
- shipped cost dollars
- store number

## Validation Rules
If critical validation fails, reject the **entire file**.

Examples of critical failures:
- missing required columns
- unknown customer
- unknown item mapping
- unknown store mapping
- invalid dates
- non-numeric quantities or dollar values
- empty required fields

Do not partially import bad files.

## Approval Workflow
Implement a configurable import mode:

### Mode A: Review Required
- file is uploaded
- validated
- must be explicitly approved before commit

### Mode B: Auto Import
- valid file can import automatically

Make this admin-configurable because the business wants to start cautiously and later relax review requirements if the process proves stable.

## Auditability
Always preserve:
- original uploaded file
- import status
- row counts
- errors
- timestamps
- approving user
- rollback actions

## Rollback
Allow admins to:
- rollback a recent import cleanly
- overwrite bad data with corrected data
- preserve audit history

Structure writes around import batches so rollback is safe and traceable.

---

## Data Model Requirements

Create a normalized schema with migrations.

At minimum include these tables or their equivalent:

- customers
- users
- regions
- stores
- cmg_items
- customer_items
- item_mappings
- store_mappings
- customer_sales_import_files
- customer_sales_import_errors
- shipment_import_files
- shipment_import_errors
- customer_sales_daily
- shipments_daily
- inventory_daily
- alerts
- alert_recipients
- audit_log

The schema must support:
- tenant isolation
- import traceability
- manual mappings
- historical daily facts
- derived inventory calculations
- alerting
- admin actions

Use strong typing and constraints wherever practical.

---

## Admin Area Requirements

Build an internal admin area that includes:

### User Management
- create user
- disable user
- assign role
- assign customer to customer users
- reset password flow or trigger

### Import Management
- upload file
- validate file
- approve/reject file
- show errors
- rollback import
- overwrite import
- show import history and status

### Mapping Management
- item mappings
- store mappings
- easy editing
- filtering for unmapped exceptions if useful

### Alert Settings
- recipient email management
- alert enable/disable

### Branding
Brand the application for:
- **Catoctin Mtn Growers**

The app should support:
- logo
- brand colors
- clean customer-facing identity

Hardcoding brand config in an easy-to-update place is acceptable in phase 1.

---

## Export and Sharing Requirements

For phase 1 support:

- PDF export/download
- CSV or Excel export/download

Do **not** build external public share links yet.
Do **not** build in-app emailing for customer buyers yet.

Customer buyers will download a PDF and send it themselves outside the app.

Also support a store-specific underperformance report export.

---

## UX and Design Guidance

Design should be:
- professional
- clean
- fast to understand
- stable
- desktop-first
- responsive enough for mobile viewing

Preferred landing experience:
- headline KPIs
- trend chart
- sell-through summary
- top items
- top stores
- inventory value
- simple filtering

Avoid flashy unnecessary UI.
Favor clarity and trust.

---

## Reliability and Engineering Standards

This project will eventually be shared with real customers.
Favor correctness and maintainability over clever shortcuts.

### Engineering principles
- typed schemas
- clear folder structure
- defensive parsing
- explicit validation
- transaction-safe writes
- no silent failures
- observable statuses
- import idempotency where practical
- strong error handling
- clean audit trails
- documented formulas
- maintainable code over hacks

### Failure handling
If a file fails:
- reject the whole file
- do not partially import
- notify the internal team
- log all relevant errors

---

## Suggested Project Structure

Use a clean structure similar to:

- `src/app/`
- `src/components/`
- `src/lib/auth/`
- `src/lib/db/`
- `src/lib/imports/`
- `src/lib/imports/customers/`
- `src/lib/imports/shipments/`
- `src/lib/metrics/`
- `src/lib/alerts/`
- `src/lib/pdf/`
- `supabase/`
- `scripts/`
- `tests/`

Customer-specific sales parsers should be isolated, for example:
- `src/lib/imports/customers/safeway.ts`
- `src/lib/imports/customers/aldi.ts`
- etc.

---

## Required Seed Data

Create realistic local seed data for at least:
- 2 customers
- several stores
- several mapped items
- shipment history
- sales history
- one alert condition
- one super admin
- one internal user
- one customer user per sample customer

Include these sample users in seeds:
- Tyler Van Wingerden as super admin
- one internal sales user
- sample buyer users

Use realistic fake data suitable for UI evaluation.

---

## Testing Requirements

Include meaningful tests.

Cover at least:
- auth and authorization
- tenant/customer data isolation
- item mapping logic
- store mapping logic
- customer sales import validation
- shipment import validation
- rollback behavior
- metric calculations
- alert logic

Prefer:
- unit tests for formulas and parsers
- integration tests for import flows
- authorization tests for data isolation

---

## Documentation Requirements

Produce a strong README that explains:
- what the app does
- tech stack
- local setup
- environment variables
- database and auth setup
- import workflow
- role model
- metric definitions
- export behavior
- known limitations
- phase 2 roadmap

Document any assumptions clearly.

---

## Acceptance Criteria

The phase 1 build is acceptable only when all of the following are true:

1. Super admin can log in locally.
2. Internal users can upload customer sales Excel files.
3. Internal users can upload shipment files.
4. The system validates files before import.
5. The system rejects the whole file if critical validation fails.
6. The system stores original uploaded files and logs import results.
7. The system supports manual item mappings.
8. The system supports manual store mappings.
9. Customer buyers can log in and see only their own customer data.
10. The dashboard displays:
   - units sold
   - sales dollars
   - sell-through
   - weeks of supply
   - net margin
   - YOY comp
   - top items
   - top stores
   - estimated retail inventory value
11. Dashboard supports filters by region, store, category, item size, and item.
12. Customer users land on an overview dashboard.
13. PDF export works.
14. CSV/Excel export works.
15. Internal users can see alerts for sell-through > 90%.
16. Internal users receive email notifications for alert events and failed imports.
17. Admins can view import logs and rollback recent imports.
18. Branding reflects Catoctin Mtn Growers.
19. The codebase is organized and documented enough for continued iteration toward production.

---

## Non-Goals for Phase 1

Do not spend time on:
- automatic email ingestion
- automatic Google Drive ingestion
- AI-assisted mapping
- public share links
- multiple permission levels inside one customer
- complex forecasting
- customer self-service administration
- advanced infrastructure

Leave room for these later, but do not build them now.

---

## Implementation Order

Follow this build sequence:

1. Initialize Next.js + TypeScript project
2. Set up Supabase auth, schema, migrations, and RLS
3. Build role model and secure tenant isolation
4. Seed demo data
5. Build admin authentication and basic layout
6. Build item/store mapping management
7. Build import pipeline and validation framework
8. Build sales and shipment import screens
9. Build dashboard metrics layer
10. Build customer-facing overview dashboard
11. Build filters and drill-down views
12. Build exports
13. Build alerts
14. Add tests
15. Finalize README and setup instructions

---

## Final Instruction

Build a **local-first, production-shaped prototype** for Catoctin Mtn Growers using Next.js, TypeScript, Supabase, and a clean modular architecture.

Optimize for:
- correctness
- security
- auditability
- maintainability
- reliable imports
- clean customer experience

Do not optimize for quick hacks.
Do not skip tenant isolation.
Do not rely on frontend-only security.
Do not partially import invalid files.

When making decisions, choose the simplest implementation that preserves correctness and future extensibility.
