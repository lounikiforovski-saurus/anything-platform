# Proposals & Contracts - App Reference

> Pre-built app on `@ab/app-foundation` (MariaDB, multi-tenant)
> Port: 4004 | DB: `ab_proposals` | Path: `/apps/proposals/`
> Sprint 2 (Weeks 4-5)

---

## Purpose

Document builder for proposals, quotes, and contracts. Business owners create section-based documents from templates, send them to clients for review, and collect e-signatures. Version history tracks every edit. Documents lock after signing. PDF export via PDF connector.

**Multi-tenant**: One shared Proposals instance serves all users. Data isolated by `user_id` / `org_id` from JWT.

---

## Core Features

### For Business Owners (authenticated)
1. **Document editor** - Section-based WYSIWYG editor. Drag to reorder sections. Rich text content per section
2. **Template system** - Create reusable templates from existing proposals. Template library with categories
3. **Variable interpolation** - Merge fields like `{{client_name}}`, `{{date}}`, `{{total}}` auto-filled when sending
4. **Share & send** - Generate token-based share links. Email to recipients via Mail connector
5. **Recipient tracking** - See when document was viewed, by whom, how long. Multiple recipients per document
6. **E-signature** - Simple signature capture (typed name + checkbox consent + timestamp + IP). Legally binding with audit trail
7. **Version history** - Snapshot saved on every significant edit. Compare versions. Restore previous versions
8. **Lock after signing** - Once all required signatures are collected, document becomes read-only
9. **PDF export** - Generate professional PDF via PDF connector at any time
10. **Status management** - Draft → Sent → Viewed → Accepted / Declined / Expired

### For Recipients (public, no auth required)
1. **Document view** - Clean, professional read-only document view via token link
2. **Accept & sign** - Review document, type name, check consent box, submit signature
3. **Decline** - Decline with optional reason
4. **PDF download** - Download signed document as PDF

### For Employees (authenticated, org-scoped)
1. **Own proposals** - Create and manage proposals assigned to them
2. **Org proposals** - Owner/admin sees all proposals across the organisation
3. **Shared templates** - Template library shared within the org

---

## Database Schema (MariaDB - `ab_proposals`)

```sql
-- Template categories
CREATE TABLE template_categories (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  user_id BIGINT UNSIGNED NOT NULL,
  org_id BIGINT UNSIGNED NULL,
  name VARCHAR(255) NOT NULL,             -- "Web Development", "Consulting", etc.
  sort_order INT DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_user (user_id)
);

-- Reusable templates
CREATE TABLE templates (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  user_id BIGINT UNSIGNED NOT NULL,
  org_id BIGINT UNSIGNED NULL,
  category_id BIGINT UNSIGNED NULL,
  name VARCHAR(255) NOT NULL,             -- "Standard Web Project Proposal"
  description TEXT,
  sections_json LONGTEXT NOT NULL,        -- JSON array of section templates
  variables_json LONGTEXT NULL,           -- JSON array of merge field definitions
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (category_id) REFERENCES template_categories(id) ON DELETE SET NULL,
  INDEX idx_user (user_id),
  INDEX idx_category (category_id)
);

-- Proposals / Contracts
CREATE TABLE proposals (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  user_id BIGINT UNSIGNED NOT NULL,       -- business owner / employee
  org_id BIGINT UNSIGNED NULL,
  template_id BIGINT UNSIGNED NULL,       -- source template (if created from one)
  title VARCHAR(500) NOT NULL,
  client_name VARCHAR(255),
  client_email VARCHAR(255),
  client_company VARCHAR(255),
  status ENUM('draft', 'sent', 'viewed', 'accepted', 'declined', 'expired', 'cancelled') DEFAULT 'draft',
  document_type ENUM('proposal', 'contract', 'quote') DEFAULT 'proposal',
  total_value DECIMAL(12,2) NULL,         -- optional deal value (for reporting)
  currency VARCHAR(3) DEFAULT 'GBP',
  valid_until DATE NULL,                  -- expiration date for the proposal
  require_signature BOOLEAN DEFAULT TRUE, -- whether e-signature is needed
  is_locked BOOLEAN DEFAULT FALSE,        -- locked after all signatures collected
  variables_json LONGTEXT NULL,           -- resolved merge field values for this document
  sent_at TIMESTAMP NULL,
  viewed_at TIMESTAMP NULL,
  accepted_at TIMESTAMP NULL,
  declined_at TIMESTAMP NULL,
  declined_reason TEXT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (template_id) REFERENCES templates(id) ON DELETE SET NULL,
  INDEX idx_user_status (user_id, status),
  INDEX idx_org_status (org_id, status),
  INDEX idx_client_email (client_email),
  INDEX idx_valid_until (valid_until)
);

-- Document sections (ordered content blocks)
CREATE TABLE sections (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  proposal_id BIGINT UNSIGNED NOT NULL,
  title VARCHAR(255),                     -- section heading (optional)
  content LONGTEXT NOT NULL,              -- rich text / HTML content
  section_type ENUM('text', 'pricing', 'terms', 'signature', 'divider', 'image') DEFAULT 'text',
  sort_order INT DEFAULT 0,
  is_visible BOOLEAN DEFAULT TRUE,        -- can hide sections without deleting
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (proposal_id) REFERENCES proposals(id) ON DELETE CASCADE,
  INDEX idx_proposal_order (proposal_id, sort_order)
);

-- Pricing table (for pricing-type sections)
CREATE TABLE pricing_items (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  section_id BIGINT UNSIGNED NOT NULL,
  description VARCHAR(500) NOT NULL,
  quantity DECIMAL(10,2) DEFAULT 1.00,
  unit_price DECIMAL(12,2) NOT NULL,
  total DECIMAL(12,2) NOT NULL,           -- quantity * unit_price
  is_optional BOOLEAN DEFAULT FALSE,      -- client can accept/decline optional items
  sort_order INT DEFAULT 0,
  FOREIGN KEY (section_id) REFERENCES sections(id) ON DELETE CASCADE,
  INDEX idx_section (section_id)
);

-- Share links / Recipients
CREATE TABLE recipients (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  proposal_id BIGINT UNSIGNED NOT NULL,
  name VARCHAR(255) NOT NULL,
  email VARCHAR(255) NOT NULL,
  role ENUM('signer', 'viewer', 'approver') DEFAULT 'signer',
  view_token VARCHAR(64) NOT NULL UNIQUE, -- unique per recipient
  viewed_at TIMESTAMP NULL,
  view_count INT DEFAULT 0,
  total_view_seconds INT DEFAULT 0,       -- approximate engagement tracking
  signed_at TIMESTAMP NULL,
  signature_name VARCHAR(255) NULL,       -- typed name for e-signature
  signature_ip VARCHAR(45) NULL,          -- IP address at time of signing
  signature_user_agent TEXT NULL,         -- browser user agent
  declined_at TIMESTAMP NULL,
  declined_reason TEXT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (proposal_id) REFERENCES proposals(id) ON DELETE CASCADE,
  INDEX idx_proposal (proposal_id),
  INDEX idx_token (view_token)
);

-- Version history (snapshots)
CREATE TABLE versions (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  proposal_id BIGINT UNSIGNED NOT NULL,
  version_number INT NOT NULL,
  snapshot_json LONGTEXT NOT NULL,        -- full document state (proposal + sections + pricing)
  change_summary VARCHAR(500),            -- "Updated pricing section", "Added terms"
  created_by BIGINT UNSIGNED NOT NULL,    -- user who made the change
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (proposal_id) REFERENCES proposals(id) ON DELETE CASCADE,
  INDEX idx_proposal_version (proposal_id, version_number)
);

-- Activity log (audit trail)
CREATE TABLE activity_log (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  proposal_id BIGINT UNSIGNED NOT NULL,
  actor_type ENUM('owner', 'recipient', 'system') NOT NULL,
  actor_name VARCHAR(255),                -- user display name or recipient name
  action ENUM('created', 'edited', 'sent', 'viewed', 'signed', 'declined', 'expired', 'cancelled', 'restored', 'pdf_downloaded') NOT NULL,
  details TEXT NULL,                      -- additional context
  ip_address VARCHAR(45) NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (proposal_id) REFERENCES proposals(id) ON DELETE CASCADE,
  INDEX idx_proposal (proposal_id)
);

-- Proposal settings (per business)
CREATE TABLE proposal_settings (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  user_id BIGINT UNSIGNED NOT NULL UNIQUE,
  org_id BIGINT UNSIGNED NULL,
  business_name VARCHAR(255),
  business_email VARCHAR(255),
  business_address TEXT,
  logo_url VARCHAR(1024),
  default_valid_days INT DEFAULT 30,       -- default expiration period
  default_footer TEXT,                     -- default footer text for proposals
  signature_disclaimer TEXT DEFAULT 'By typing your name above and clicking Accept, you agree to the terms outlined in this document.',
  branding_color VARCHAR(7) DEFAULT '#3B82F6', -- accent colour for documents
  timezone VARCHAR(50) DEFAULT 'UTC',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

---

## API Routes

### Authenticated (JWT required)

```
# Templates
GET    /api/templates                    → List templates (query: category_id)
POST   /api/templates                    → Create template
GET    /api/templates/:id                → Get template with sections
PUT    /api/templates/:id                → Update template
DELETE /api/templates/:id                → Deactivate template

# Template Categories
GET    /api/template-categories          → List categories
POST   /api/template-categories          → Create category
PUT    /api/template-categories/:id      → Update category
DELETE /api/template-categories/:id      → Delete category

# Proposals
GET    /api/proposals                    → List proposals (query: status, type, client_email, date_from, date_to)
POST   /api/proposals                    → Create proposal (optionally from template_id)
GET    /api/proposals/:id                → Get proposal with sections, recipients, activity
PUT    /api/proposals/:id                → Update proposal metadata (only if not locked)
DELETE /api/proposals/:id                → Cancel proposal

# Sections
GET    /api/proposals/:id/sections       → Get all sections (ordered)
POST   /api/proposals/:id/sections       → Add section
PUT    /api/proposals/:id/sections/:sid  → Update section content
DELETE /api/proposals/:id/sections/:sid  → Remove section
PUT    /api/proposals/:id/sections/reorder → Reorder sections

# Pricing Items (within pricing sections)
POST   /api/sections/:sid/pricing        → Add pricing item
PUT    /api/sections/:sid/pricing/:pid   → Update pricing item
DELETE /api/sections/:sid/pricing/:pid   → Remove pricing item

# Recipients
POST   /api/proposals/:id/recipients     → Add recipient
PUT    /api/proposals/:id/recipients/:rid → Update recipient
DELETE /api/proposals/:id/recipients/:rid → Remove recipient

# Actions
POST   /api/proposals/:id/send           → Send to all recipients (emails via Mail connector)
POST   /api/proposals/:id/resend/:rid    → Resend to specific recipient
POST   /api/proposals/:id/expire         → Manually expire proposal
POST   /api/proposals/:id/save-as-template → Save current proposal as a template

# Versions
GET    /api/proposals/:id/versions       → List versions
GET    /api/proposals/:id/versions/:vid  → Get version snapshot
POST   /api/proposals/:id/versions/:vid/restore → Restore to version (creates new version)

# PDF
GET    /api/proposals/:id/pdf            → Generate and download PDF (via PDF connector)

# Activity
GET    /api/proposals/:id/activity       → Get activity log

# Settings
GET    /api/settings                     → Get proposal settings
PUT    /api/settings                     → Update proposal settings

# Reports
GET    /api/reports/pipeline             → Pipeline report (proposals by status, total values)
GET    /api/reports/conversion           → Conversion rate (sent vs accepted, avg time to sign)
```

### Public (no auth, rate-limited)

```
# Recipient View
GET    /api/public/view/:token           → Get document for recipient (tracks view)
POST   /api/public/sign/:token           → Submit signature (name, consent checkbox)
POST   /api/public/decline/:token        → Decline document (optional reason)
GET    /api/public/pdf/:token            → Download PDF of document
```

---

## Status State Machine

```
                  ┌──────────┐
                  │  DRAFT   │ (editable)
                  └────┬─────┘
                       │ send
                  ┌────v─────┐
                  │   SENT   │
                  └────┬─────┘
                       │ recipient opens link
                  ┌────v─────┐
                  │  VIEWED  │
                  └────┬─────┘
                       │
              ┌────────┼────────┐
              │        │        │
         ┌────v───┐    │   ┌────v─────┐
         │ACCEPTED│    │   │ DECLINED │
         └────────┘    │   └──────────┘
                       │
                  ┌────v─────┐
                  │ EXPIRED  │ (auto, past valid_until)
                  └──────────┘

  Any status except ACCEPTED → CANCELLED (manual)
```

**Transitions**:
- `draft → sent`: On send action. Emails all recipients with unique view tokens
- `sent → viewed`: When any recipient opens the document link
- `viewed → accepted`: When all required signers have signed
- `viewed → declined`: When any signer declines
- `sent/viewed → expired`: Automatic when `valid_until` date passes
- `any → cancelled`: Manual cancellation by owner

---

## Version History Logic

A snapshot is saved when significant changes happen:

```javascript
async function createVersionSnapshot(proposalId, userId, changeSummary) {
  const proposal = await db('proposals').where({ id: proposalId }).first();
  const sections = await db('sections').where({ proposal_id: proposalId }).orderBy('sort_order');

  // Include pricing items for pricing sections
  const sectionsWithPricing = await Promise.all(sections.map(async (s) => {
    if (s.section_type === 'pricing') {
      s.pricing_items = await db('pricing_items').where({ section_id: s.id }).orderBy('sort_order');
    }
    return s;
  }));

  const lastVersion = await db('versions')
    .where({ proposal_id: proposalId })
    .orderBy('version_number', 'desc')
    .first();

  const versionNumber = (lastVersion?.version_number || 0) + 1;

  await db('versions').insert({
    proposal_id: proposalId,
    version_number: versionNumber,
    snapshot_json: JSON.stringify({
      proposal: { title: proposal.title, client_name: proposal.client_name, total_value: proposal.total_value },
      sections: sectionsWithPricing,
    }),
    change_summary: changeSummary,
    created_by: userId,
  });

  return versionNumber;
}
```

**When snapshots are created**:
- After adding/removing/reordering sections
- After editing section content (debounced - not on every keystroke)
- After changing pricing items
- Before sending to recipients (captures the sent version)
- Before restoring a previous version

---

## E-Signature Flow

```
1. Recipient clicks link in email → lands on public view page
2. Reads full document (view time tracked)
3. Scrolls to signature section at bottom
4. Types their full name in signature field
5. Checks consent checkbox ("I agree to the terms...")
6. Clicks "Accept & Sign" button
7. System captures:
   - Typed name
   - Timestamp (UTC)
   - IP address
   - User agent string
   - Consent checkbox state
8. Recipient record updated with signature data
9. Activity log entry created
10. Owner notified via email
11. If all required signers have signed → proposal status → accepted, document locked
```

```javascript
async function processSignature(token, signatureData) {
  const recipient = await db('recipients').where({ view_token: token }).first();
  const proposal = await db('proposals').where({ id: recipient.proposal_id }).first();

  if (proposal.is_locked) throw new Error('Document is already signed');
  if (proposal.status === 'expired') throw new Error('Document has expired');
  if (recipient.signed_at) throw new Error('Already signed');

  // Record signature
  await db('recipients').where({ id: recipient.id }).update({
    signed_at: new Date(),
    signature_name: signatureData.name,
    signature_ip: signatureData.ip,
    signature_user_agent: signatureData.userAgent,
  });

  // Log activity
  await db('activity_log').insert({
    proposal_id: proposal.id,
    actor_type: 'recipient',
    actor_name: recipient.name,
    action: 'signed',
    details: `Signed as "${signatureData.name}"`,
    ip_address: signatureData.ip,
  });

  // Check if all required signers have signed
  const pendingSigners = await db('recipients')
    .where({ proposal_id: proposal.id, role: 'signer' })
    .whereNull('signed_at')
    .count('id as count')
    .first();

  if (pendingSigners.count === 0) {
    // All signed → lock document
    await db('proposals').where({ id: proposal.id }).update({
      status: 'accepted',
      accepted_at: new Date(),
      is_locked: true,
    });

    // Notify owner
    await mail.send({
      to: ownerEmail,
      template: 'proposal-accepted',
      data: {
        proposalTitle: proposal.title,
        clientName: proposal.client_name,
        signedBy: recipient.name,
        acceptedAt: new Date(),
      },
    });
  } else {
    // Notify owner of individual signature
    await mail.send({
      to: ownerEmail,
      template: 'proposal-signed',
      data: {
        proposalTitle: proposal.title,
        signedBy: recipient.name,
        pendingCount: pendingSigners.count,
      },
    });
  }
}
```

---

## Connector Usage

### Mail Connector (`@ab/connectors/mail`)

```javascript
// Send proposal to recipient
await mail.send({
  to: recipient.email,
  template: 'proposal-review',
  data: {
    recipientName: recipient.name,
    proposalTitle: proposal.title,
    senderName: settings.business_name,
    documentType: proposal.document_type, // "proposal", "contract", "quote"
    validUntil: proposal.valid_until,
    viewUrl: `${BASE_URL}/api/public/view/${recipient.view_token}`,
  },
});

// Signature notification to owner
await mail.send({
  to: settings.business_email,
  template: 'proposal-signed',
  data: {
    proposalTitle: proposal.title,
    signedBy: recipient.name,
    signedAt: recipient.signed_at,
    pendingCount: remainingSigners,
  },
});

// All signatures collected → fully accepted
await mail.send({
  to: settings.business_email,
  template: 'proposal-accepted',
  data: {
    proposalTitle: proposal.title,
    clientName: proposal.client_name,
    totalValue: proposal.total_value,
    acceptedAt: proposal.accepted_at,
    pdfUrl: `${BASE_URL}/api/proposals/${proposal.id}/pdf`,
  },
});

// Declined notification
await mail.send({
  to: settings.business_email,
  template: 'proposal-declined',
  data: {
    proposalTitle: proposal.title,
    declinedBy: recipient.name,
    reason: recipient.declined_reason,
  },
});
```

### PDF Connector (`@ab/connectors/pdf`)

```javascript
const pdfBuffer = await pdf.generate({
  template: 'proposal',
  data: {
    businessName: settings.business_name,
    businessAddress: settings.business_address,
    logoUrl: settings.logo_url,
    brandingColor: settings.branding_color,
    title: proposal.title,
    documentType: proposal.document_type,
    clientName: proposal.client_name,
    clientCompany: proposal.client_company,
    createdDate: proposal.created_at,
    validUntil: proposal.valid_until,
    sections: sections.map(s => ({
      title: s.title,
      content: s.content,
      type: s.section_type,
      pricingItems: s.pricing_items || [],
    })),
    totalValue: proposal.total_value,
    currency: proposal.currency,
    signatures: recipients
      .filter(r => r.signed_at)
      .map(r => ({
        name: r.signature_name,
        signedAt: r.signed_at,
      })),
    isLocked: proposal.is_locked,
    footer: settings.default_footer,
  },
});
```

---

## Variable Interpolation

Templates and proposals support merge fields:

```javascript
const BUILT_IN_VARIABLES = [
  { key: 'client_name', label: 'Client Name' },
  { key: 'client_email', label: 'Client Email' },
  { key: 'client_company', label: 'Client Company' },
  { key: 'business_name', label: 'Your Business Name' },
  { key: 'date', label: 'Current Date' },
  { key: 'valid_until', label: 'Valid Until Date' },
  { key: 'total_value', label: 'Total Value' },
  { key: 'proposal_title', label: 'Document Title' },
];

function resolveVariables(content, variables) {
  return content.replace(/\{\{(\w+)\}\}/g, (match, key) => {
    return variables[key] || match; // Keep unresolved variables as-is
  });
}

// Applied when rendering for recipients
function renderSectionForRecipient(section, proposal, settings) {
  const variables = {
    client_name: proposal.client_name,
    client_email: proposal.client_email,
    client_company: proposal.client_company,
    business_name: settings.business_name,
    date: formatDate(new Date()),
    valid_until: formatDate(proposal.valid_until),
    total_value: formatCurrency(proposal.total_value, proposal.currency),
    proposal_title: proposal.title,
    ...JSON.parse(proposal.variables_json || '{}'), // custom variables
  };

  return {
    ...section,
    content: resolveVariables(section.content, variables),
  };
}
```

---

## Expiration Job

A daily cron marks expired proposals:

```javascript
async function markExpiredProposals() {
  const expired = await db('proposals')
    .whereIn('status', ['sent', 'viewed'])
    .whereNotNull('valid_until')
    .where('valid_until', '<', today())
    .select();

  for (const proposal of expired) {
    await db('proposals')
      .where({ id: proposal.id })
      .update({ status: 'expired' });

    await db('activity_log').insert({
      proposal_id: proposal.id,
      actor_type: 'system',
      action: 'expired',
      details: `Expired on ${proposal.valid_until}`,
    });
  }
}
```

---

## Engagement Tracking

When a recipient views the document, the system tracks:

```javascript
async function trackView(token, ip) {
  const recipient = await db('recipients').where({ view_token: token }).first();

  // Update view count
  await db('recipients').where({ id: recipient.id }).update({
    viewed_at: recipient.viewed_at || new Date(), // first view only
    view_count: recipient.view_count + 1,
  });

  // Update proposal status
  const proposal = await db('proposals').where({ id: recipient.proposal_id }).first();
  if (proposal.status === 'sent') {
    await db('proposals')
      .where({ id: proposal.id })
      .update({ status: 'viewed', viewed_at: new Date() });
  }

  // Log activity
  await db('activity_log').insert({
    proposal_id: recipient.proposal_id,
    actor_type: 'recipient',
    actor_name: recipient.name,
    action: 'viewed',
    ip_address: ip,
  });
}
```

---

## UI Pages

### Admin (authenticated)

| Page | Route | Description |
|------|-------|-------------|
| Dashboard | `/` | Pipeline overview (proposals by status, total values). Recent activity. Quick create |
| Proposals | `/proposals` | List with filters (status, type, client, date range). Status badges. Bulk actions |
| Proposal Editor | `/proposals/new`, `/proposals/:id/edit` | Section-based editor. Drag to reorder. Rich text per section. Pricing table editor. Variable insertion. Live preview toggle |
| Proposal View | `/proposals/:id` | Read-only view with recipient status, engagement stats, signature status, activity timeline, version history |
| Templates | `/templates` | Template library with categories. Create from scratch or save existing proposal |
| Template Editor | `/templates/:id/edit` | Same editor as proposals, but for template content |
| Reports | `/reports` | Pipeline report (value by status), conversion rate (sent→accepted), avg time to sign |
| Settings | `/settings` | Business details, logo, default footer, signature disclaimer, branding colour, default validity |

### Public (no auth)

| Page | Route | Description |
|------|-------|-------------|
| Document View | `/view/:token` | Professional document display. Sections rendered. Pricing table. Signature area at bottom (if signer role) |
| Sign | `/view/:token` (bottom section) | Type name, check consent, submit. Confirmation state after signing |
| Decline | `/view/:token` (action) | Decline button with optional reason textarea |

### UI Components

- **SectionEditor** - Rich text editor per section (bold, italic, lists, links, headings). Drag handle for reorder
- **PricingTable** - Line item editor within pricing sections. Optional items with checkbox. Auto-calculated totals
- **VariableInserter** - Dropdown to insert `{{variable}}` placeholders into content
- **RecipientManager** - Add/remove recipients. Set role (signer/viewer/approver). Show view/sign status per recipient
- **SignatureCapture** - Text input for typed name, consent checkbox, legal disclaimer, submit button
- **VersionTimeline** - Visual timeline of version snapshots. Click to preview. Restore button
- **ActivityFeed** - Chronological list of all events (created, edited, viewed, signed, etc.) with timestamps
- **StatusBadge** - Colour-coded status pill (draft=grey, sent=blue, viewed=amber, accepted=green, declined=red, expired=slate)
- **DocumentPreview** - Side-by-side preview of how the document appears to recipients, with resolved variables

---

## Key Business Rules

1. **Lock after signing**: Once all required signers have signed, `is_locked = true`. No further edits allowed. New version cannot be created. Owner must cancel and create new proposal to make changes
2. **One decline = declined**: If any signer declines, the entire proposal status becomes `declined`. Owner can cancel and re-send or discuss with client
3. **Unique tokens per recipient**: Each recipient gets their own view token. Enables per-recipient tracking and prevents link sharing issues
4. **Version snapshots are immutable**: Once created, snapshots cannot be modified. Restore creates a new version, not an overwrite
5. **Expiration is automatic**: Daily cron checks `valid_until` and transitions to `expired`. Expired proposals cannot be signed. Owner can cancel and re-issue with new validity
6. **Signature audit trail**: Every signature captures name, timestamp, IP, user agent. This data is included in the PDF and stored permanently. Cannot be edited or deleted
7. **Multi-tenant isolation**: All queries filtered by `user_id`. Org members share templates but see only their own proposals (admin sees all)
8. **Optional items in pricing**: Pricing sections can mark items as optional. The public view shows checkboxes for optional items. Total recalculates based on selected items
9. **Template inheritance**: Creating a proposal from a template copies all sections. The proposal is independent after creation (editing the template doesn't affect existing proposals)
10. **View tracking is per-recipient**: Each recipient's engagement is tracked independently. Owner can see who has viewed and for how long
