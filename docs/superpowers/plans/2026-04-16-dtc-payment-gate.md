# DTC Payment Gate — Figma Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create all new Figma design nodes for the DTC payment gate feature — Issue Invoice stage card, Pending Payment status pill, Approve Job override modal, and Job Approved email template.

**Architecture:** New nodes only — no modifications to existing Figma components. All new work lives in a new section on the Purchase Orders page. Each surface is built using existing component instances, bound to design tokens, and verified against Figma Design Rules from CLAUDE.md.

**Tech Stack:** Figma plugin API via `figma_execute`, `figma_search_components`, `figma_get_variables`, `figma_take_screenshot`

**Spec:** `docs/superpowers/specs/2026-04-16-dtc-payment-gate-design.md`

---

## Reference nodes (read-only — do NOT modify)

| What | Node ID | Purpose |
|---|---|---|
| PO record side peek (stage cards) | `24873:478860` | Visual grammar for Source / Review / Markup cards |
| PO table | `24873:477216` | Status pill pattern |
| Markup screen (with Approve Job) | `17928:20235` | Top navbar + breadcrumb + action bar |
| Reject PO modal | `24883:630548` | Modal pattern |
| Report Issue modal | `24883:630549` | Modal pattern |
| Job Completed email | `7661:18640` | Email template pattern |
| Action Button component set | Search via `figma_search_components` | Button instances |
| Review PO section | `24883:589749` | Parent section on Purchase Orders page |
| Purchase Orders page | `13773:100280` | Target page |

---

## Chunk 1: Pre-build setup & Section creation

### Task 0: Pre-build — Resolve tokens, fonts, styles, and components

Before creating any nodes, gather all required design system references.

- [ ] **Step 0.1: Fetch variables**

```
figma_get_variables — get Semantic + Fixed collection variable IDs.
Build lookup map for: fills (surface, border, text colors), spacing, radius.
Store as a JS object for reuse across all tasks.
```

- [ ] **Step 0.2: Check available fonts**

```js
// via figma_execute
const textNodes = figma.currentPage.findAll(n => n.type === 'TEXT').slice(0, 50);
const fonts = [...new Set(textNodes.map(n => JSON.stringify(n.fontName)))];
return fonts;
```

Expected: Inter Regular, Inter Medium (confirmed in CLAUDE.md). Flag any others found.

- [ ] **Step 0.3: Check available text styles**

```js
// via figma_execute
const styles = await figma.getLocalTextStylesAsync();
return styles.map(s => ({ id: s.id, name: s.name, fontSize: s.fontSize }));
```

Map style names to IDs for use in all text nodes.

- [ ] **Step 0.4: Search for reusable components**

Search for these components via `figma_search_components`:
- `Action Button` — for modal actions, Send reminder
- `Status pill` / `Badge` / `Tag` — for substate pills
- `Input` / `Select` / `Dropdown` — for override modal reason dropdown
- `Modal` — for override modal shell
- `Textarea` / `Text Input` — for notes field
- `Icon` — for card header icons

Record component IDs and variant properties for instantiation.

- [ ] **Step 0.5: Screenshot existing stage cards for reference**

```
figma_take_screenshot — nodeId: 24873:478860
```

Study: card padding, title font/style, pill position, body layout, border treatment, spacing between cards.

### Task 1: Create section on Purchase Orders page

- [ ] **Step 1.1: Screenshot Purchase Orders page to find clear space**

```
figma_take_screenshot — nodeId: 13773:100280
```

Identify coordinates below/beside existing sections. Per memory: sections run horizontally, use margins (100/50/100/50), 50px header-to-content gap.

- [ ] **Step 1.2: Create "DTC Payment Gate" section**

```js
// via figma_execute on Purchase Orders page
const page = await figma.getNodeByIdAsync('13773:100280');
// Position to the right of existing sections or below
const section = figma.createSection();
section.name = 'DTC Payment Gate';
// Position based on screenshot findings
section.x = [calculated from step 1.1];
section.y = [calculated from step 1.1];
section.resizeWithoutConstraints(5000, 5700);
page.appendChild(section);
return { id: section.id };
```

- [ ] **Step 1.3: Verify section placement**

```
figma_take_screenshot — verify no overlap with existing sections
```

---

## Chunk 2: Issue Invoice stage card (7 substates)

### Task 2: Build Issue Invoice stage card — "Not issued" substate

Reference the existing stage card (Source/Review/Markup) visual grammar from node `24873:478860`.

- [ ] **Step 2.1: Inspect existing stage card structure**

```js
// via figma_execute — deep-read the Source card structure
const peek = await figma.getNodeByIdAsync('24873:478860');
// Walk first card child to understand: frame structure, padding, auto-layout, 
// pill component, title text style, body layout, border/stroke, icon usage
```

Document: exact padding, spacing, layout mode, text styles used, pill variant, icon component.

- [ ] **Step 2.2: Create "Issue Invoice — Not Issued" card frame**

Build inside the DTC Payment Gate section. Match:
- Same width as existing stage cards
- Auto-layout VERTICAL
- Same padding, spacing, border treatment
- Header row: icon + "Issue Invoice" title + grey "Awaiting invoice" pill
- Body: guidance text — "Create the invoice in your accounting system. It will sync here once issued."
- All fills/strokes bound to variables, all text using text styles

- [ ] **Step 2.3: Screenshot and verify**

```
figma_take_screenshot — verify card matches existing card grammar
```

### Task 3: Build remaining 6 substate variants

Each variant reuses the card frame from Task 2, varying the pill and body content.

- [ ] **Step 3.1: "Issued" variant**

- Pill: Blue "Issued"
- Body: Invoice number (linked text), value, due date
- Actions: "View invoice" link, "Send reminder" button (Action Button instance, Secondary)

- [ ] **Step 3.2: "Sent" variant**

- Pill: Green "Sent"
- Body: same as Issued
- Actions: same as Issued

- [ ] **Step 3.3: "Viewed" variant**

- Pill: Green "Viewed"
- Body: same as Issued
- Actions: same as Issued

- [ ] **Step 3.4: "Overdue" variant**

- Pill: Amber "Overdue"
- Body: same as Issued + amber warning treatment on card border/background
- Actions: same as Issued

- [ ] **Step 3.5: "Payment failed" variant**

- Pill: Red "Payment failed"
- Body: same as Issued + red error treatment on card border/background
- Actions: same as Issued

- [ ] **Step 3.6: "Paid" variant**

- Pill: Green "Paid"
- Body: "Paid on [date] · $[amount]", invoice number linked
- Actions: "View invoice" link only (no Send reminder)

- [ ] **Step 3.7: "Post-override" variant (e.g. Sent — after manual approve)**

- Same as Sent variant BUT: "Send reminder" button is **hidden** (job already approved via override)
- Shows the card is read-only post-override
- Can be shown with a subtle "Job approved" indicator or greyed treatment

- [ ] **Step 3.8: Screenshot all variants and verify**

```
figma_take_screenshot — verify all 8 cards side by side
```

Check: consistent sizing, aligned pills, correct colors, all tokens bound. Verify post-override variant clearly omits Send reminder.

---

## Chunk 3: PO table status pill & Override modal

### Task 4: "Pending Payment" status pill

- [ ] **Step 4.1: Inspect existing status pills on PO table**

```js
// via figma_execute — find status pill instances on PO table node 24873:477216
// Document: component used, variant properties, text style, colors
```

- [ ] **Step 4.2: Create "Pending Payment" pill instance**

Instantiate the same component, set text to "Pending Payment". Place in the DTC section.

- [ ] **Step 4.3: Screenshot and verify matches existing pill grammar**

### Task 5: Override modal — "Approve job before payment?"

Reference existing modals: Reject PO (`24883:630548`), Report Issue (`24883:630549`).

- [ ] **Step 5.1: Inspect existing modal structure**

```js
// via figma_execute — deep-read Reject PO modal
const modal = await figma.getNodeByIdAsync('24883:630548');
// Document: width, padding, title style, body style, action bar layout,
// button variants used, overall frame structure
```

- [ ] **Step 5.2: Create override modal frame**

Build inside DTC section. Structure:
- Modal shell (same dimensions/style as existing modals)
- Title: "Approve job before payment?"
- Body text: "This DTC job hasn't been paid yet. Approving now will let work commence before payment is received."
- Dropdown (Select component instance): "Reason for early approval" with placeholder
- Dropdown options listed beside modal for reference:
  - Customer paid offline (cash / bank transfer pending reconciliation)
  - Trusted account / repeat customer
  - Manager exception approved
  - Urgent / time-sensitive install
  - Other
- Text input / textarea: "Notes (optional)"
- Action bar: `Cancel` (Secondary) · `Approve anyway` (Primary)
- All tokens bound, all text styled

- [ ] **Step 5.3: Screenshot and verify**

Compare side-by-side with Reject PO modal for consistency.

---

## Chunk 4: Markup screen variants & Email template

### Task 6: Markup screen — Approve Job button states (DTC)

Reference: existing Markup screen `17928:20235`.

- [ ] **Step 6.1: Inspect Markup screen top navbar / action bar**

```js
// via figma_execute — read the action bar frame from the Markup screen
// Find Approve Job button, document its position, variant, parent frame
```

- [ ] **Step 6.2: Create DTC Markup screen variant — unpaid**

Duplicate the Markup screen layout (as a new frame, not modifying existing). Changes:
- **Breadcrumb: `Source > Review > Issue Invoice > Markup`** — this is a key deliverable (Spec Section 6). The "Issue Invoice" step must be clearly visible in the breadcrumb nav.
- Approve Job button: **Secondary** style (instead of Primary)
- Helper text below or beside button: "Job will auto-approve when payment is received"
- Place in DTC section

- [ ] **Step 6.3: Create DTC Markup screen variant — paid**

Same as Step 6.2 but:
- Approve Job button: **Primary** style (standard)
- No helper text

- [ ] **Step 6.4: Screenshot both variants and verify**

Check: breadcrumb reads correctly, button style difference is clear, helper text is legible.

### Task 7: Job Approved email template

Reference: Job Completed email `7661:18640`.

- [ ] **Step 7.1: Inspect Job Completed email structure**

```js
// via figma_execute — deep-read Job Completed email node
// Document: frame width, header bg color, title style, body padding,
// text styles, footer structure, CTA button
```

- [ ] **Step 7.2: Create "Job Approved" email**

Build inside DTC section. Structure (matching Job Completed pattern):
- Header: Capture logo + green gradient background
- Title: "Job Approved"
- Body: 
  > Hey {{First-name}},
  >
  > PO **{{PO-number}}** for **{{Customer Name}}** has been automatically approved following payment of invoice **{{Invoice-number}}**.
- CTA button: "View in the app" (Primary button style from email)
- Footer: standard Capture Sales footer (support email, phone, copyright, unsubscribe)

- [ ] **Step 7.3: Screenshot and verify**

Compare with Job Completed email for visual consistency.

---

## Chunk 5: Assembly & PO record view

### Task 8: PO record — DTC view with Issue Invoice card

Create a full PO record side peek showing the DTC flow with the new Issue Invoice card.

- [ ] **Step 8.1: Inspect existing PO record side peek**

```
figma_take_screenshot — nodeId: 24873:478860
```

Document card order, spacing between cards, overall frame structure.

- [ ] **Step 8.2: Create DTC PO record view**

New frame in DTC section showing:
1. Source card — "Received" (instance or replica of existing)
2. Review card — "Confirmed" (instance or replica of existing)
3. **Issue Invoice card** — show "Sent" substate as the default/representative state
4. Markup card — "Pending" state (not yet complete)

Title: "PO-750772-459-1" (example), status badge "Pending Payment"

- [ ] **Step 8.3: Screenshot and verify**

Check: card stacking, consistent spacing, Issue Invoice card fits visually between Review and Markup.

---

## Chunk 6: Post-build verification

### Task 9: Full audit

Per CLAUDE.md Figma Design Rules — run ALL verification scans.

- [ ] **Step 9.1: Tokens audit**

```js
// via figma_execute — scan all nodes in the DTC Payment Gate section
// Flag any hardcoded fills, strokes, spacing, radius (not bound to variables)
```

- [ ] **Step 9.2: Font audit**

```js
// Scan all text nodes for fontName — flag anything not Inter Regular or Inter Medium
```

- [ ] **Step 9.3: Text style audit**

```js
// Verify all text nodes are linked to a Figma text style, not raw properties
```

- [ ] **Step 9.4: Component audit**

```js
// Flag raw frames that should be component instances (buttons, pills, inputs, etc.)
```

- [ ] **Step 9.5: Fix any flagged issues**

Address each issue found in steps 9.1–9.4.

- [ ] **Step 9.6: Final screenshot of entire DTC section**

```
figma_take_screenshot — full section
```

Confirm all deliverables are present and visually consistent.

---

## Deliverables checklist

| # | Deliverable | Task |
|---|---|---|
| 1 | Issue Invoice card — Not issued | Task 2 |
| 2 | Issue Invoice card — Issued | Task 3 |
| 3 | Issue Invoice card — Sent | Task 3 |
| 4 | Issue Invoice card — Viewed | Task 3 |
| 5 | Issue Invoice card — Overdue | Task 3 |
| 6 | Issue Invoice card — Payment failed | Task 3 |
| 7 | Issue Invoice card — Paid | Task 3 |
| 8 | Pending Payment status pill | Task 4 |
| 9 | Override modal | Task 5 |
| 10 | DTC Markup screen — unpaid (secondary button + helper) | Task 6 |
| 11 | DTC Markup screen — paid (primary button) | Task 6 |
| 12 | Job Approved email template | Task 7 |
| 13 | DTC PO record view (assembled) | Task 8 |
| 14 | DTC breadcrumb (`Source > Review > Issue Invoice > Markup`) | Task 6 |
| 15 | Post-override card variant (Send reminder hidden) | Task 3 |

**Explicitly excluded from Figma deliverables:**
- **History tab** — override reason and auto-approval log entries are data/backend concerns. The existing History tab UI already supports free-form log entries; no new Figma design needed.

**Tools/MCPs used:** figma_execute, figma_take_screenshot, figma_search_components, figma_get_variables, figma_get_component_image
**Skills used:** executing-plans, verification-before-completion
