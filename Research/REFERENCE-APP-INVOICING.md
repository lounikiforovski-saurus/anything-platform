# Invoicing & Late Payment Chasing - App Reference

> Pre-built app on `@ab/app-foundation` (MariaDB, multi-tenant)
> Port: 4003 | DB: `ab_invoicing` | Path: `/apps/invoicing/`
> Sprint 2 (Weeks 4-5)

---

## Purpose

Invoice management system for freelancers and small businesses. Create professional invoices, send to clients, track payments, and automatically chase late payments with escalating email reminders. PDF generation via PDF connector.

**Multi-tenant**: One shared Invoicing instance serves all users. Data isolated by `user_id` / `org_id` from JWT.

---

## Core Features

### For Business Owners (authenticated)
1. **Invoice management** - Create, edit, duplicate, send invoices. Auto-numbering with configurable prefix
2. **Line items** - Add/remove/reorder line items with description, quantity, unit price, tax rate. Auto-calculated totals
3. **Status tracking** - Draft → Sent → Viewed → Paid / Overdue. Visual status badges and filters
4. **Payment recording** - Record full or partial payments. Payment method, date, reference number, notes
5. **Client directory** - Manage clients with billing info, payment terms, contact details, invoice history
6. **Late payment chasing** - Automated escalating reminders (configurable schedule: 1 day, 7 days, 14 days, 30 days overdue)
7. **Aging report** - Overview of outstanding invoices grouped by age (current, 1-30, 31-60, 61-90, 90+ days)
8. **PDF download** - Generate professional PDF invoices via PDF connector
9. **Tax summary** - Tax collected per period for reporting

### For Clients (public, no auth required)
1. **Invoice view** - View invoice via token link (clean, print-friendly layout)
2. **Payment confirmation** - See payment status and history on the invoice page
3. **PDF download** - Download invoice as PDF from the public view

### For Employees (authenticated, org-scoped)
1. **Own invoices** - Create and manage invoices assigned to them
2. **Org invoices** - Owner/admin sees all invoices across the organisation
3. **Client sharing** - Shared client directory within the org

---

## Database Schema (MariaDB - `ab_invoicing`)

```sql
-- Client directory
CREATE TABLE clients (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  user_id BIGINT UNSIGNED NOT NULL,       -- business owner (from JWT)
  org_id BIGINT UNSIGNED NULL,            -- org scoping
  name VARCHAR(255) NOT NULL,             -- company or individual name
  email VARCHAR(255),
  phone VARCHAR(50),
  address_line1 VARCHAR(255),
  address_line2 VARCHAR(255),
  city VARCHAR(100),
  postcode VARCHAR(20),
  country VARCHAR(100) DEFAULT 'UK',
  tax_id VARCHAR(100),                    -- VAT number, EIN, etc.
  payment_terms_days INT DEFAULT 30,      -- default payment terms for this client
  notes TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  INDEX idx_user (user_id),
  INDEX idx_org (org_id),
  INDEX idx_email (user_id, email)
);

-- Invoice settings (per business)
CREATE TABLE invoice_settings (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  user_id BIGINT UNSIGNED NOT NULL UNIQUE,
  org_id BIGINT UNSIGNED NULL,
  business_name VARCHAR(255),
  business_email VARCHAR(255),
  business_phone VARCHAR(50),
  business_address TEXT,                  -- full address block
  tax_id VARCHAR(100),                    -- business VAT/tax number
  invoice_prefix VARCHAR(20) DEFAULT 'INV-',
  next_invoice_number INT DEFAULT 1,      -- auto-increment counter
  default_payment_terms_days INT DEFAULT 30,
  default_tax_rate DECIMAL(5,2) DEFAULT 0.00, -- e.g. 20.00 for 20% VAT
  default_currency VARCHAR(3) DEFAULT 'GBP',
  bank_details TEXT,                      -- bank name, sort code, account number
  payment_instructions TEXT,              -- "Please pay by bank transfer to..."
  footer_note TEXT,                       -- appears at bottom of every invoice
  logo_url VARCHAR(1024),                 -- business logo (via Storage connector)
  chase_schedule JSON DEFAULT '[1, 7, 14, 30]', -- days after due date to send reminders
  chase_enabled BOOLEAN DEFAULT TRUE,
  timezone VARCHAR(50) DEFAULT 'UTC',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Invoices
CREATE TABLE invoices (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  user_id BIGINT UNSIGNED NOT NULL,       -- business owner / employee
  org_id BIGINT UNSIGNED NULL,
  client_id BIGINT UNSIGNED NOT NULL,
  invoice_number VARCHAR(50) NOT NULL,    -- e.g. "INV-0042"
  status ENUM('draft', 'sent', 'viewed', 'paid', 'overdue', 'cancelled', 'write_off') DEFAULT 'draft',
  issue_date DATE NOT NULL,
  due_date DATE NOT NULL,
  currency VARCHAR(3) DEFAULT 'GBP',
  subtotal DECIMAL(12,2) DEFAULT 0.00,    -- sum of line items (excl. tax)
  tax_total DECIMAL(12,2) DEFAULT 0.00,   -- sum of tax amounts
  total DECIMAL(12,2) DEFAULT 0.00,       -- subtotal + tax_total
  amount_paid DECIMAL(12,2) DEFAULT 0.00, -- sum of payments recorded
  amount_due DECIMAL(12,2) DEFAULT 0.00,  -- total - amount_paid
  notes TEXT,                             -- internal notes (not shown to client)
  client_note TEXT,                       -- note shown on invoice to client
  view_token VARCHAR(64) NULL UNIQUE,     -- for client to view without auth
  sent_at TIMESTAMP NULL,
  viewed_at TIMESTAMP NULL,
  paid_at TIMESTAMP NULL,
  cancelled_at TIMESTAMP NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (client_id) REFERENCES clients(id),
  UNIQUE KEY uk_user_number (user_id, invoice_number),
  INDEX idx_user_status (user_id, status),
  INDEX idx_org_status (org_id, status),
  INDEX idx_due_date (due_date),
  INDEX idx_client (client_id)
);

-- Invoice line items
CREATE TABLE line_items (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  invoice_id BIGINT UNSIGNED NOT NULL,
  description VARCHAR(500) NOT NULL,
  quantity DECIMAL(10,2) DEFAULT 1.00,
  unit_price DECIMAL(12,2) NOT NULL,
  tax_rate DECIMAL(5,2) DEFAULT 0.00,     -- percentage (e.g. 20.00)
  tax_amount DECIMAL(12,2) DEFAULT 0.00,  -- calculated: quantity * unit_price * tax_rate / 100
  line_total DECIMAL(12,2) DEFAULT 0.00,  -- calculated: quantity * unit_price + tax_amount
  sort_order INT DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (invoice_id) REFERENCES invoices(id) ON DELETE CASCADE,
  INDEX idx_invoice (invoice_id)
);

-- Payments recorded against invoices
CREATE TABLE payments (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  invoice_id BIGINT UNSIGNED NOT NULL,
  amount DECIMAL(12,2) NOT NULL,
  payment_date DATE NOT NULL,
  payment_method ENUM('bank_transfer', 'cash', 'cheque', 'card', 'other') DEFAULT 'bank_transfer',
  reference VARCHAR(255),                 -- payment reference / transaction ID
  notes TEXT,
  recorded_by BIGINT UNSIGNED NOT NULL,   -- user who recorded the payment
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (invoice_id) REFERENCES invoices(id) ON DELETE CASCADE,
  INDEX idx_invoice (invoice_id)
);

-- Chase reminder log (track what was sent)
CREATE TABLE chase_reminders (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  invoice_id BIGINT UNSIGNED NOT NULL,
  days_overdue INT NOT NULL,              -- how many days overdue when sent
  sent_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  email_to VARCHAR(255),
  escalation_level INT DEFAULT 1,         -- 1=gentle, 2=firm, 3=urgent, 4=final
  FOREIGN KEY (invoice_id) REFERENCES invoices(id) ON DELETE CASCADE,
  INDEX idx_invoice (invoice_id)
);

-- Recurring invoice templates (for repeat billing)
CREATE TABLE recurring_invoices (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  user_id BIGINT UNSIGNED NOT NULL,
  org_id BIGINT UNSIGNED NULL,
  client_id BIGINT UNSIGNED NOT NULL,
  frequency ENUM('weekly', 'monthly', 'quarterly', 'yearly') DEFAULT 'monthly',
  next_issue_date DATE NOT NULL,
  is_active BOOLEAN DEFAULT TRUE,
  line_items_json LONGTEXT NOT NULL,      -- JSON array of line item templates
  client_note TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (client_id) REFERENCES clients(id),
  INDEX idx_user_active (user_id, is_active),
  INDEX idx_next_date (next_issue_date)
);
```

---

## API Routes

### Authenticated (JWT required)

```
# Clients
GET    /api/clients                      → List clients (search by name/email)
POST   /api/clients                      → Create client
GET    /api/clients/:id                  → Get client detail
PUT    /api/clients/:id                  → Update client
DELETE /api/clients/:id                  → Soft delete client
GET    /api/clients/:id/invoices         → Client's invoice history

# Invoices
GET    /api/invoices                     → List invoices (query: status, client_id, date_from, date_to)
POST   /api/invoices                     → Create invoice (auto-number assigned)
GET    /api/invoices/:id                 → Get invoice with line items + payments
PUT    /api/invoices/:id                 → Update invoice (only if draft)
DELETE /api/invoices/:id                 → Cancel invoice (sets status=cancelled)
POST   /api/invoices/:id/duplicate       → Duplicate invoice as new draft
POST   /api/invoices/:id/send            → Send invoice to client (sets status=sent, emails via Mail connector)
POST   /api/invoices/:id/remind          → Manually trigger chase reminder

# Line Items
POST   /api/invoices/:id/items           → Add line item (recalculates totals)
PUT    /api/invoices/:id/items/:itemId   → Update line item (recalculates totals)
DELETE /api/invoices/:id/items/:itemId   → Remove line item (recalculates totals)
PUT    /api/invoices/:id/items/reorder   → Reorder line items

# Payments
POST   /api/invoices/:id/payments        → Record payment (updates amount_paid, amount_due, status)
DELETE /api/invoices/:id/payments/:payId  → Remove payment record

# PDF
GET    /api/invoices/:id/pdf             → Generate and download PDF (via PDF connector)

# Settings
GET    /api/settings                     → Get invoice settings
PUT    /api/settings                     → Update invoice settings

# Reports
GET    /api/reports/aging                → Aging report (outstanding by age bracket)
GET    /api/reports/revenue              → Revenue summary (query: period, date_from, date_to)
GET    /api/reports/tax                  → Tax collected summary

# Recurring
GET    /api/recurring                    → List recurring invoice templates
POST   /api/recurring                    → Create recurring template
PUT    /api/recurring/:id                → Update recurring template
DELETE /api/recurring/:id                → Deactivate recurring template
```

### Public (no auth, rate-limited)

```
# Public Invoice View
GET    /api/public/invoice/:token        → Get invoice data for public display
GET    /api/public/invoice/:token/pdf    → Download PDF of invoice
```

---

## Status State Machine

```
                  ┌──────────┐
                  │  DRAFT   │ (editable)
                  └────┬─────┘
                       │ send
                  ┌────v─────┐
            ┌─────│   SENT   │
            │     └────┬─────┘
            │          │ client opens link
            │     ┌────v─────┐
            │     │  VIEWED  │
            │     └────┬─────┘
            │          │
            │    ┌─────┴──────┐
            │    │            │
            │    │ (past      │ record
            │    │  due date) │ payment
            │    │            │
       ┌────v────v┐     ┌────v─────┐
       │ OVERDUE  │────>│   PAID   │
       └────┬─────┘     └──────────┘
            │ payment
            └──────────> PAID

  Any status except PAID → CANCELLED (manual)
  OVERDUE → WRITE_OFF (manual, after exhausting chase)
```

**Transitions**:
- `draft → sent`: On send action. Generates view token, emails client
- `sent → viewed`: When client opens the public view link (tracked via token)
- `sent/viewed → overdue`: Automatic when `due_date` passes without full payment
- `sent/viewed/overdue → paid`: When `amount_paid >= total`
- `any → cancelled`: Manual cancellation by owner
- `overdue → write_off`: Manual, when debt is deemed uncollectable

---

## Auto-numbering Logic

```javascript
async function getNextInvoiceNumber(userId) {
  const settings = await db('invoice_settings').where({ user_id: userId }).first();
  const prefix = settings?.invoice_prefix || 'INV-';
  const nextNum = settings?.next_invoice_number || 1;

  // Pad to 4 digits minimum
  const invoiceNumber = `${prefix}${String(nextNum).padStart(4, '0')}`;

  // Increment counter
  await db('invoice_settings')
    .where({ user_id: userId })
    .update({ next_invoice_number: nextNum + 1 });

  return invoiceNumber;
}
```

---

## Total Recalculation

Triggered whenever line items change:

```javascript
async function recalculateTotals(invoiceId) {
  const items = await db('line_items').where({ invoice_id: invoiceId });

  let subtotal = 0;
  let taxTotal = 0;

  for (const item of items) {
    const lineSubtotal = item.quantity * item.unit_price;
    const lineTax = lineSubtotal * (item.tax_rate / 100);

    await db('line_items').where({ id: item.id }).update({
      tax_amount: lineTax,
      line_total: lineSubtotal + lineTax,
    });

    subtotal += lineSubtotal;
    taxTotal += lineTax;
  }

  const payments = await db('payments').where({ invoice_id: invoiceId });
  const amountPaid = payments.reduce((sum, p) => sum + parseFloat(p.amount), 0);
  const total = subtotal + taxTotal;

  await db('invoices').where({ id: invoiceId }).update({
    subtotal,
    tax_total: taxTotal,
    total,
    amount_paid: amountPaid,
    amount_due: total - amountPaid,
  });
}
```

---

## Connector Usage

### Mail Connector (`@ab/connectors/mail`)

```javascript
// Invoice sent to client
await mail.send({
  to: client.email,
  template: 'invoice-sent',
  data: {
    clientName: client.name,
    invoiceNumber: invoice.invoice_number,
    total: invoice.total,
    currency: invoice.currency,
    dueDate: invoice.due_date,
    businessName: settings.business_name,
    viewUrl: `${BASE_URL}/api/public/invoice/${invoice.view_token}`,
  },
});

// Chase reminder (escalating tone based on level)
await mail.send({
  to: client.email,
  template: 'payment-reminder',
  data: {
    clientName: client.name,
    invoiceNumber: invoice.invoice_number,
    total: invoice.total,
    amountDue: invoice.amount_due,
    currency: invoice.currency,
    dueDate: invoice.due_date,
    daysOverdue: daysOverdue,
    escalationLevel: level, // 1=gentle, 2=firm, 3=urgent, 4=final
    businessName: settings.business_name,
    viewUrl: `${BASE_URL}/api/public/invoice/${invoice.view_token}`,
  },
});

// Payment received confirmation
await mail.send({
  to: client.email,
  template: 'payment-received',
  data: {
    clientName: client.name,
    invoiceNumber: invoice.invoice_number,
    amountPaid: payment.amount,
    remainingBalance: invoice.amount_due,
    currency: invoice.currency,
    businessName: settings.business_name,
  },
});
```

### PDF Connector (`@ab/connectors/pdf`)

```javascript
const pdfBuffer = await pdf.generate({
  template: 'invoice',
  data: {
    businessName: settings.business_name,
    businessAddress: settings.business_address,
    businessEmail: settings.business_email,
    businessPhone: settings.business_phone,
    taxId: settings.tax_id,
    logoUrl: settings.logo_url,
    invoiceNumber: invoice.invoice_number,
    issueDate: invoice.issue_date,
    dueDate: invoice.due_date,
    currency: invoice.currency,
    clientName: client.name,
    clientAddress: formatAddress(client),
    clientTaxId: client.tax_id,
    lineItems: items.map(i => ({
      description: i.description,
      quantity: i.quantity,
      unitPrice: i.unit_price,
      taxRate: i.tax_rate,
      lineTotal: i.line_total,
    })),
    subtotal: invoice.subtotal,
    taxTotal: invoice.tax_total,
    total: invoice.total,
    amountPaid: invoice.amount_paid,
    amountDue: invoice.amount_due,
    bankDetails: settings.bank_details,
    paymentInstructions: settings.payment_instructions,
    footerNote: settings.footer_note,
  },
});
```

### Chase Reminder Scheduler

A cron-like job runs every hour:

```javascript
async function processChaseReminders() {
  const allSettings = await db('invoice_settings')
    .where({ chase_enabled: true })
    .select();

  for (const settings of allSettings) {
    const schedule = JSON.parse(settings.chase_schedule); // [1, 7, 14, 30]

    const overdueInvoices = await db('invoices')
      .where({ user_id: settings.user_id })
      .whereIn('status', ['sent', 'viewed', 'overdue'])
      .where('due_date', '<', today())
      .where('amount_due', '>', 0)
      .select();

    for (const invoice of overdueInvoices) {
      const daysOverdue = daysBetween(invoice.due_date, today());

      // Find which schedule step we're at
      for (let i = 0; i < schedule.length; i++) {
        if (daysOverdue >= schedule[i]) {
          const alreadySent = await db('chase_reminders')
            .where({ invoice_id: invoice.id, days_overdue: schedule[i] })
            .first();

          if (!alreadySent) {
            await mail.send({
              to: client.email,
              template: 'payment-reminder',
              data: { ..., escalationLevel: i + 1 },
            });

            await db('chase_reminders').insert({
              invoice_id: invoice.id,
              days_overdue: schedule[i],
              email_to: client.email,
              escalation_level: i + 1,
            });

            // Update status to overdue if not already
            if (invoice.status !== 'overdue') {
              await db('invoices')
                .where({ id: invoice.id })
                .update({ status: 'overdue' });
            }
          }
        }
      }
    }
  }
}
```

### Escalation Tone Guide

| Level | Days Overdue | Subject | Tone |
|-------|-------------|---------|------|
| 1 | 1 day | "Friendly reminder: Invoice {number} is due" | Gentle, assumes oversight |
| 2 | 7 days | "Payment reminder: Invoice {number} is 7 days overdue" | Firm but polite |
| 3 | 14 days | "Urgent: Invoice {number} is 14 days overdue" | Direct, requests immediate action |
| 4 | 30 days | "Final notice: Invoice {number} is 30 days overdue" | Formal, mentions next steps |

---

## Overdue Detection Job

A daily cron marks invoices overdue:

```javascript
async function markOverdueInvoices() {
  await db('invoices')
    .whereIn('status', ['sent', 'viewed'])
    .where('due_date', '<', today())
    .where('amount_due', '>', 0)
    .update({ status: 'overdue' });
}
```

---

## Recurring Invoice Job

A daily cron generates invoices from recurring templates:

```javascript
async function processRecurringInvoices() {
  const due = await db('recurring_invoices')
    .where({ is_active: true })
    .where('next_issue_date', '<=', today())
    .select();

  for (const template of due) {
    // Create invoice from template
    const invoiceNumber = await getNextInvoiceNumber(template.user_id);
    const lineItems = JSON.parse(template.line_items_json);

    const invoiceId = await createInvoice({
      user_id: template.user_id,
      org_id: template.org_id,
      client_id: template.client_id,
      invoice_number: invoiceNumber,
      issue_date: today(),
      due_date: addDays(today(), client.payment_terms_days),
      line_items: lineItems,
      client_note: template.client_note,
    });

    // Calculate next issue date
    const nextDate = calculateNextDate(template.next_issue_date, template.frequency);
    await db('recurring_invoices')
      .where({ id: template.id })
      .update({ next_issue_date: nextDate });
  }
}
```

---

## UI Pages

### Admin (authenticated)

| Page | Route | Description |
|------|-------|-------------|
| Dashboard | `/` | Summary cards (total outstanding, overdue count, paid this month, revenue chart). Quick actions |
| Invoices | `/invoices` | List with filters (status, client, date range). Status badges. Bulk actions (send, remind) |
| Invoice Editor | `/invoices/new`, `/invoices/:id/edit` | Line item editor, auto-calc totals, tax handling, client picker, date pickers. Preview pane |
| Invoice View | `/invoices/:id` | Read-only view with status timeline, payment history, chase history, action buttons |
| Clients | `/clients` | Searchable client directory. Click for detail + invoice history |
| Client Detail | `/clients/:id` | Contact info, billing details, all invoices, payment summary |
| Reports | `/reports` | Aging report, revenue summary, tax summary. Date range selectors, export |
| Recurring | `/recurring` | Manage recurring invoice templates. Enable/disable, edit frequency |
| Settings | `/settings` | Business details, logo upload, invoice numbering, default tax rate, bank details, chase schedule config |

### Public (no auth)

| Page | Route | Description |
|------|-------|-------------|
| Invoice View | `/view/:token` | Clean, professional invoice display. Payment status shown. PDF download button. Print-friendly CSS |

### UI Components

- **InvoiceEditor** - Line item table with add/remove/reorder, auto-calculated totals, tax per line
- **InvoicePreview** - Live preview of invoice as it will appear to client (matching PDF layout)
- **StatusBadge** - Colour-coded status pill (draft=grey, sent=blue, viewed=amber, paid=green, overdue=red, cancelled=slate)
- **AgingReport** - Table with expandable rows grouped by age bracket, totals per bracket
- **PaymentRecorder** - Modal to record payment (amount, date, method, reference)
- **ChaseTimeline** - Visual timeline showing sent reminders with dates and escalation levels
- **ClientPicker** - Autocomplete dropdown for selecting existing client or creating new

---

## Key Business Rules

1. **Immutable after send**: Invoices can only be edited while in `draft` status. Once sent, create a credit note or cancel and re-issue
2. **Auto-numbering**: Sequential within each business. Prefix + zero-padded number. Never reuses numbers (even after cancellation)
3. **Partial payments**: Multiple payments can be recorded against one invoice. Status becomes `paid` when `amount_paid >= total`
4. **Chase automation**: Only runs when `chase_enabled = true`. Respects the schedule array. Never sends the same escalation level twice for the same invoice
5. **Overdue detection**: Daily job transitions `sent`/`viewed` invoices past due date to `overdue` status
6. **Currency per invoice**: Each invoice has its own currency (defaults from settings). Line items inherit from invoice
7. **Tax handling**: Tax rate is per line item (allows mixed VAT rates). Invoice shows subtotal, tax breakdown, and total
8. **View tracking**: When client opens the public view link, `viewed_at` is set and status transitions from `sent` to `viewed`
9. **Multi-tenant isolation**: All queries filtered by `user_id`. Org members share client directory but see only their own invoices (admin sees all)
10. **Cancellation**: Sets status to `cancelled`, records `cancelled_at`. Does not delete data. Cancelled invoices still visible in reports
