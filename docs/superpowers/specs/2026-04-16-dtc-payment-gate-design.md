# DTC Payment Gate — Design Spec

**Date**: 2026-04-16
**Status**: Approved
**Figma file**: [Ops & Installer App (UI/UX)](https://www.figma.com/design/jiyL7WRxpAsyYlyaE6tIXf/Ops---Installer-App--UI-UX-)

---

## Design references

### Figma nodes (read-only — do not modify)

| Reference | Node ID | URL | Purpose |
|---|---|---|---|
| PO record side peek (stage cards) | `24873:478860` | [View](https://www.figma.com/design/jiyL7WRxpAsyYlyaE6tIXf/Ops---Installer-App--UI-UX-?node-id=24873-478860) | Visual grammar for Source / Review / Markup cards — match card width, padding, spacing, border, pill placement |
| PO table | `24873:477216` | [View](https://www.figma.com/design/jiyL7WRxpAsyYlyaE6tIXf/Ops---Installer-App--UI-UX-?node-id=24873-477216) | Status pill pattern — match "Pending Review" / "Pending Markup" pill styling |
| Markup screen | `17928:20235` | [View](https://www.figma.com/design/jiyL7WRxpAsyYlyaE6tIXf/Ops---Installer-App--UI-UX-?node-id=17928-20235) | Top navbar breadcrumb + Approve Job button placement |
| Reject PO modal | `24883:630548` | [View](https://www.figma.com/design/jiyL7WRxpAsyYlyaE6tIXf/Ops---Installer-App--UI-UX-?node-id=24883-630548) | Modal shell pattern — match dimensions, title style, action bar layout |
| Report Issue modal | `24883:630549` | [View](https://www.figma.com/design/jiyL7WRxpAsyYlyaE6tIXf/Ops---Installer-App--UI-UX-?node-id=24883-630549) | Modal with form fields pattern — match dropdown + text input layout |
| Job Completed email | `7661:18640` | [View](https://www.figma.com/design/jiyL7WRxpAsyYlyaE6tIXf/Ops---Installer-App--UI-UX-?node-id=7661-18640) | Email template — match header, body, CTA, footer treatment |
| Review Form component | `24883:616159` | [View](https://www.figma.com/design/jiyL7WRxpAsyYlyaE6tIXf/Ops---Installer-App--UI-UX-?node-id=24883-616159) | Form/Action panel context |
| Invoices table | `20301:78372` | [View](https://www.figma.com/design/jiyL7WRxpAsyYlyaE6tIXf/Ops---Installer-App--UI-UX-?node-id=20301-78372) | Existing Send reminder action pattern |

### Components to use (search via `figma_search_components` at build time)

- **Action Button** — for modal actions (Cancel, Approve anyway), Send reminder, View invoice CTA
- **Status pill / Badge / Tag** — for substate pills on the Issue Invoice card and "Pending Payment" in PO table
- **Select / Dropdown** — for override modal reason field
- **Text Input / Textarea** — for override modal notes field
- **Modal** — shell component if one exists, otherwise build from Reject PO / Report Issue reference
- **Radio** — for process indicator (existing component `Radio` in component set)
- **Top Navbar** — existing component set, variant `Type=Purchase Orders`

### Design tokens

All nodes must bind to Figma variables — never use raw hex, px spacing, or raw font properties. Key token areas:

- **Colors**: surface fills, text fills, border strokes — resolve from Semantic + Fixed variable collections via `figma_get_variables`
- **Status pill colors**: grey (neutral), blue (info), green (success), amber (warning), red (error) — match existing pill component variants
- **Spacing**: item spacing and padding — bind via `setBoundVariable`
- **Radius**: corner radius — bind via `setBoundVariable`
- **Typography**: all text must use Figma text styles named `Text-{weight}/{size}` (e.g. `Text-semibold/2xl`, `Text-normal/xs`). Approved fonts: Inter Regular, Inter Medium.

### UI aesthetic notes

- Stage cards use a clean, bordered card with subtle background fill, left-aligned icon + title, right-aligned status pill
- Status pills use rounded-full shape with coloured background + matching text
- Modals use consistent width (~393px based on existing), with clear title → body → action bar vertical flow
- Email templates follow Capture Sales branding: dark green header with white logo, light body, green CTA button
- Overall aesthetic: clean, structured, operational — not decorative. Prioritise clarity and scannability.

---

## Context

Lovelight has introduced "Direct to Customer" (DTC) jobs that capture billing details during PO confirmation. These jobs require customers to pay their invoices before work commences. We need to gate job approval on invoice payment — while still allowing ops to override when necessary.

DTC is a value of the existing **Division** field (not a separate job type).

## Current workflow

Source → Review → Markup → *Approve Job*

- **Source**: PO email arrives, status = "Received"
- **Review**: Ops confirms PO details (the Review Form / Action Panel), status = "Confirmed"
- **Markup**: Ops uploads marked-up floorplan, selects accepted quote. "Approve Job" button (top-right of Markup screen) completes this stage.
- Stages render as cards on the PO record's "Review & Markup" tab
- Breadcrumb on Markup screen: `Source > Review > Markup`

## DTC workflow (new)

Source → Review → **Issue Invoice** → Markup → *Approve Job (gated)*

Changes apply **only** to POs where Division = Direct to Customer. Non-DTC flow is unchanged.

---

## 1. Issue Invoice stage card

A new stage card on the PO record, positioned between **Review** and **Markup**. Follows the same visual grammar as the existing Source / Review / Markup cards.

### Card structure

- **Header**: Icon + "Issue Invoice" title + status pill (right-aligned)
- **Body**: varies by substate (see below)

### Substates

Invoices are created in an external accounting system (e.g. Xero/MYOB) and synced into the platform. The card is read-only — no invoice creation happens in-app.

| Substate | Pill style | Pill label | Trigger |
|---|---|---|---|
| Not issued | Grey | Awaiting invoice | No invoice synced for this PO |
| Issued | Blue | Issued | Invoice exists, not yet sent |
| Sent | Green | Sent | Invoice sent to customer |
| Viewed | Green | Viewed | Customer opened invoice |
| Paid | Green | Paid | Payment received |
| Overdue | Amber | Overdue | Past due date, unpaid |
| Failed | Red | Payment failed | Payment attempt failed |

### Card body by substate

**Not issued**
- Guidance text: "Create the invoice in your accounting system. It will sync here once issued."
- No action button

**Issued / Sent / Viewed / Overdue / Failed**
- Invoice number (linked), value, due date
- "View invoice" link → navigates to `/Invoices` filtered to this invoice
- "Send reminder" action button (mirrors existing Invoices table action). Hidden once the job has been manually approved via override — reminders are no longer relevant.
- Overdue: amber warning treatment
- Failed: red error treatment

**Paid**
- Success state: "Paid on [date] · $[amount]"
- Invoice number linked to `/Invoices`

### Relationship to Markup

The Markup stage is **not blocked** by Issue Invoice — ops can upload floorplans and do markup work in parallel. The gate is only on the **Approve Job** action.

---

## 2. PO table status

### New status pill

- **Pending Payment** — new status value, same pill grammar as "Pending Review" / "Pending Markup"
- Appears in the Status column of the PO table
- PO enters this status after Review is confirmed (for DTC only), replacing "Confirmed". The PO stays at "Pending Payment" until either payment is received (auto-approve) or ops manually overrides.

### Filtering

- Rolls up under the existing **Pending** tab (no new top-level tab)
- DTC POs are identifiable via the existing **Division** filter group (Division = Direct to Customer). No new column or badge needed.

---

## 3. Approve Job gate (DTC only)

The **Approve Job** button on the Markup screen (top-right action bar) gets modified behaviour for DTC POs. Non-DTC behaviour is unchanged.

### Button presentation

| Invoice substate | Button style | Helper text |
|---|---|---|
| Not issued / Issued / Sent / Viewed / Overdue / Failed | **Secondary** (de-emphasised) | "Job will auto-approve when payment is received" |
| Paid | **Primary** (standard) | *(none)* |

The button is never disabled — ops can always click it.

### Override flow (invoice != Paid)

Clicking Approve Job when the invoice is not Paid opens a confirmation modal:

**Modal: "Approve job before payment?"**

- Body: "This DTC job hasn't been paid yet. Approving now will let work commence before payment is received."
- **Reason for early approval** *(required dropdown)*:
  - Customer paid offline (cash / bank transfer pending reconciliation)
  - Trusted account / repeat customer
  - Manager exception approved
  - Urgent / time-sensitive install
  - Other
- **Notes** *(optional free text; required if "Other" selected)*
- Actions: `Cancel` · `Approve anyway` (primary)

The selected reason and notes are stored against the PO/Job record and visible in the **History** tab for audit.

On manual override, the PO status transitions immediately from "Pending Payment" → approved. The Issue Invoice card remains visible (read-only) showing the last known invoice substate, but the Send reminder action is hidden since the job is no longer gated.

### Standard flow (invoice = Paid)

Clicking Approve Job follows the existing approval flow (same as non-DTC today).

---

## 4. Auto-approval on payment

When the invoice substate flips to **Paid** while the PO is still at "Pending Payment" status:

**Preconditions for auto-approval:**
- Invoice substate = Paid
- PO status = Pending Payment (not already manually approved)
- Markup is complete (floorplan uploaded and accepted quote selected). If markup is incomplete, the PO remains at Pending Payment and auto-approval fires once markup is also done.

**When preconditions are met:**
1. Job is automatically approved (same effect as clicking Approve Job)
2. PO status transitions from "Pending Payment" → approved
3. History entry: "Job auto-approved on payment of invoice [invoice-number]"
4. Email notification fires to **Job Owner** (see Section 5)

If the PO has already been manually approved (override), the payment event is logged but no action is taken.

**Multiple invoices:** If an invoice is voided and reissued in the external system, the card displays the most recent active invoice. Auto-approval logic only fires on the active invoice's Paid event.

---

## 5. Email notification — Job auto-approved

### Template

Based on the existing **"Job Completed"** email template (simple status-change notification with positive green header treatment).

### Details

- **Trigger**: Auto-approval fires (invoice = Paid → job approved)
- **Recipient**: Job Owner (ops user assigned to the PO)
- **Subject**: "Job Approved — [PO Number]"
- **Header**: "Job Approved"
- **Body**:
  > Hey {{First-name}},
  >
  > PO **{{PO-number}}** for **{{Customer Name}}** has been automatically approved following payment of invoice **{{Invoice-number}}**.
- **CTA button**: "View in the app" → deep-link to PO/Job
- **Footer**: Standard Capture Sales footer

---

## 6. Breadcrumb update

DTC POs on the Markup screen show an updated breadcrumb:

`Source > Review > Issue Invoice > Markup`

Non-DTC remains: `Source > Review > Markup`

---

## Surfaces affected (summary)

| Surface | Change | Scope |
|---|---|---|
| PO record — Review & Markup tab | New "Issue Invoice" stage card | DTC only |
| PO table — Status column | New "Pending Payment" pill | DTC only |
| PO table — Division filter | Already exists; no change needed | — |
| Markup screen — Approve Job button | Secondary style + helper text when unpaid | DTC only |
| Markup screen — Breadcrumb | Add "Issue Invoice" step | DTC only |
| Override modal | New modal component | DTC only |
| History tab | Log override reason + auto-approval events | DTC only |
| Email templates | New "Job Approved" email (based on Job Completed) | DTC only |
| Invoices section | No changes (existing Send reminder action suffices) | — |

---

## Out of scope

- Invoice creation in-app (invoices are created externally and synced)
- Changes to the Invoices section UI
- Changes to non-DTC PO workflow
- Payment processing or reconciliation UI
- Customer-facing email notifications (auto-approval email is internal, to Job Owner only)
