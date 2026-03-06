# Appointment Scheduler - App Reference

> Pre-built app on `@ab/app-foundation` (MariaDB, multi-tenant)
> Port: 4002 | DB: `ab_scheduler` | Path: `/apps/scheduler/`
> Sprint 1 (Weeks 2-3)

---

## Purpose

Appointment booking system for service businesses (salons, consultants, clinics, etc.). Business owners configure their availability, clients book through a public booking page. Confirmations and reminders sent via Mail connector.

**Multi-tenant**: One shared Scheduler instance serves all users. Data isolated by `user_id` / `org_id` from JWT.

---

## Core Features

### For Business Owners (authenticated)
1. **Availability management** - Set working hours, recurring patterns, buffer time between appointments, blocked dates
2. **Appointment calendar** - Day/week/month views, drag to reschedule, colour-coded by status
3. **Booking management** - View, confirm, cancel, reschedule appointments
4. **Client directory** - List of clients with booking history, contact info, notes
5. **Public booking page config** - Customise the public booking link (services offered, durations, intro text)
6. **Reminders** - Automated email reminders to clients (configurable timing: 24h, 1h before)

### For Clients (public, no auth required)
1. **Public booking page** - Select service, pick date, see available slots, fill contact form, confirm
2. **Booking confirmation** - Email with appointment details and calendar invite (.ics)
3. **Cancellation link** - One-click cancel from confirmation email (token-based)

### For Employees (authenticated, org-scoped)
1. **Own calendar** - See own appointments only
2. **Availability per employee** - Each employee sets their own hours
3. **Org calendar** - Owner/admin sees all employees' appointments

---

## Database Schema (MariaDB - `ab_scheduler`)

```sql
-- Services offered by the business
CREATE TABLE services (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  user_id BIGINT UNSIGNED NOT NULL,       -- business owner (from JWT)
  org_id BIGINT UNSIGNED NULL,            -- org scoping
  name VARCHAR(255) NOT NULL,             -- "Haircut", "Consultation", etc.
  description TEXT,
  duration_minutes INT NOT NULL DEFAULT 60,
  price DECIMAL(10,2) NULL,               -- optional price display
  color VARCHAR(7) DEFAULT '#3B82F6',     -- calendar colour
  is_active BOOLEAN DEFAULT TRUE,
  sort_order INT DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  INDEX idx_user (user_id),
  INDEX idx_org (org_id)
);

-- Recurring availability patterns
CREATE TABLE availability_rules (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  user_id BIGINT UNSIGNED NOT NULL,       -- the person whose availability this is
  org_id BIGINT UNSIGNED NULL,
  day_of_week TINYINT NOT NULL,           -- 0=Sunday, 6=Saturday
  start_time TIME NOT NULL,               -- e.g. '09:00'
  end_time TIME NOT NULL,                 -- e.g. '17:00'
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_user_day (user_id, day_of_week)
);

-- Specific date overrides (holidays, special hours, blocked days)
CREATE TABLE availability_overrides (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  user_id BIGINT UNSIGNED NOT NULL,
  date DATE NOT NULL,
  start_time TIME NULL,                   -- NULL = entire day blocked
  end_time TIME NULL,
  is_blocked BOOLEAN DEFAULT FALSE,       -- TRUE = day off
  reason VARCHAR(255),                    -- "Bank Holiday", "Personal"
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE KEY uk_user_date (user_id, date)
);

-- Buffer time and booking settings
CREATE TABLE booking_settings (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  user_id BIGINT UNSIGNED NOT NULL UNIQUE,
  org_id BIGINT UNSIGNED NULL,
  buffer_minutes INT DEFAULT 15,          -- gap between appointments
  min_notice_hours INT DEFAULT 2,         -- minimum advance booking time
  max_advance_days INT DEFAULT 60,        -- how far ahead clients can book
  cancellation_hours INT DEFAULT 24,      -- cancellation deadline
  reminder_hours JSON DEFAULT '[24]',     -- when to send reminders [24, 1]
  public_page_intro TEXT,                 -- intro text on public page
  public_page_slug VARCHAR(255),          -- custom slug for public booking URL
  timezone VARCHAR(50) DEFAULT 'UTC',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Appointments
CREATE TABLE appointments (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  user_id BIGINT UNSIGNED NOT NULL,       -- business owner / employee
  org_id BIGINT UNSIGNED NULL,
  service_id BIGINT UNSIGNED NOT NULL,
  client_id BIGINT UNSIGNED NULL,         -- FK to clients (NULL for walk-ins)
  date DATE NOT NULL,
  start_time TIME NOT NULL,
  end_time TIME NOT NULL,
  status ENUM('scheduled', 'confirmed', 'completed', 'cancelled', 'no_show') DEFAULT 'scheduled',
  notes TEXT,
  cancellation_token VARCHAR(64) NULL UNIQUE,  -- for client self-cancel
  cancelled_at TIMESTAMP NULL,
  cancelled_by ENUM('owner', 'client') NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (service_id) REFERENCES services(id),
  FOREIGN KEY (client_id) REFERENCES clients(id) ON DELETE SET NULL,
  INDEX idx_user_date (user_id, date),
  INDEX idx_org_date (org_id, date),
  INDEX idx_status (status)
);

-- Client directory
CREATE TABLE clients (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  user_id BIGINT UNSIGNED NOT NULL,       -- which business this client belongs to
  org_id BIGINT UNSIGNED NULL,
  name VARCHAR(255) NOT NULL,
  email VARCHAR(255),
  phone VARCHAR(50),
  notes TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  INDEX idx_user (user_id),
  INDEX idx_email (user_id, email)
);

-- Reminder log (track what was sent)
CREATE TABLE reminders (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  appointment_id BIGINT UNSIGNED NOT NULL,
  type ENUM('confirmation', 'reminder_24h', 'reminder_1h', 'cancellation') NOT NULL,
  sent_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  email_to VARCHAR(255),
  FOREIGN KEY (appointment_id) REFERENCES appointments(id) ON DELETE CASCADE,
  INDEX idx_appointment (appointment_id)
);
```

---

## API Routes

### Authenticated (JWT required)

```
# Services
GET    /api/services                    → List services for current user/org
POST   /api/services                    → Create service
PUT    /api/services/:id                → Update service
DELETE /api/services/:id                → Delete service (soft: set is_active=false)

# Availability
GET    /api/availability                → Get rules + overrides for current user
PUT    /api/availability/rules          → Batch update weekly rules
POST   /api/availability/overrides      → Add date override (block day, special hours)
DELETE /api/availability/overrides/:id  → Remove override

# Booking Settings
GET    /api/settings                    → Get booking settings
PUT    /api/settings                    → Update booking settings

# Appointments
GET    /api/appointments                → List appointments (query: date_from, date_to, status)
GET    /api/appointments/:id            → Get appointment detail
POST   /api/appointments                → Create appointment (owner-created)
PUT    /api/appointments/:id            → Update appointment (reschedule, change status)
DELETE /api/appointments/:id            → Cancel appointment

# Clients
GET    /api/clients                     → List clients (search by name/email)
POST   /api/clients                     → Create client
PUT    /api/clients/:id                 → Update client
GET    /api/clients/:id/history         → Client's booking history

# Calendar (aggregated view)
GET    /api/calendar                    → Appointments + availability for date range
GET    /api/calendar/org                → Org-wide calendar (owner/admin only)
```

### Public (no auth, rate-limited)

```
# Public Booking
GET    /api/public/:slug                → Get business info, services, intro text
GET    /api/public/:slug/slots          → Available slots for date + service
POST   /api/public/:slug/book           → Create booking (client name, email, service, slot)
GET    /api/public/cancel/:token        → Cancel appointment via token
```

---

## Slot Calculation Logic

Available slots are computed, not stored:

```
1. Load availability rules for the requested day_of_week
2. Apply overrides for the specific date (blocked days, special hours)
3. Load existing appointments for the date
4. Load buffer_minutes from booking settings
5. For each service duration:
   a. Generate all possible start times within available hours (e.g. every 15 min)
   b. Remove slots that overlap with existing appointments (including buffer)
   c. Remove slots that violate min_notice_hours
   d. Remove slots past max_advance_days
6. Return available slots with start_time and end_time
```

---

## Connector Usage

### Mail Connector (`@ab/connectors/mail`)

```javascript
// Booking confirmation
await mail.send({
  to: client.email,
  template: 'booking-confirmation',
  data: {
    clientName: client.name,
    serviceName: service.name,
    date: appointment.date,
    startTime: appointment.start_time,
    endTime: appointment.end_time,
    businessName: settings.business_name,
    cancellationUrl: `${BASE_URL}/api/public/cancel/${appointment.cancellation_token}`,
  },
});

// Reminder (24h or 1h before)
await mail.send({
  to: client.email,
  template: 'appointment-reminder',
  data: { clientName, serviceName, date, startTime, hoursUntil },
});

// Cancellation confirmation
await mail.send({
  to: client.email,
  template: 'booking-cancelled',
  data: { clientName, serviceName, date, startTime },
});
```

### Reminder Scheduler

A cron-like job (Fastify plugin or setInterval) runs every 15 minutes:

```javascript
async function processReminders() {
  const settings = await db('booking_settings').select();

  for (const setting of settings) {
    const reminderHours = JSON.parse(setting.reminder_hours); // [24, 1]

    for (const hours of reminderHours) {
      const targetTime = new Date(Date.now() + hours * 60 * 60 * 1000);
      const appointments = await db('appointments')
        .where({ user_id: setting.user_id, status: 'scheduled' })
        .where('date', targetTime.toISOString().split('T')[0])
        // ... time window check
        .whereNotExists(/* already sent this reminder type */);

      for (const appt of appointments) {
        await mail.send({ ... });
        await db('reminders').insert({ appointment_id: appt.id, type: `reminder_${hours}h` });
      }
    }
  }
}
```

---

## UI Pages

### Admin (authenticated)

| Page | Route | Description |
|------|-------|-------------|
| Calendar | `/` | Default view. Day/week/month toggle. Shows appointments colour-coded by service. Click to view/edit |
| Appointments | `/appointments` | List view with filters (date range, status, service). Bulk actions |
| Availability | `/availability` | Weekly grid (Mon-Sun, drag to set hours). Override list for special dates |
| Services | `/services` | CRUD list of services (name, duration, price, colour) |
| Clients | `/clients` | Searchable client directory. Click for booking history |
| Settings | `/settings` | Booking rules (buffer, notice, advance days, reminders). Public page config |

### Public Booking (no auth)

| Page | Route | Description |
|------|-------|-------------|
| Booking | `/book/:slug` | Select service → pick date → see available slots → fill form → confirm |
| Confirmation | `/book/:slug/confirmed` | "You're booked!" with details and .ics download |
| Cancellation | `/cancel/:token` | "Are you sure?" → confirm → "Cancelled" |

### UI Components

- **CalendarView** - Week/month grid, appointment blocks, drag-to-reschedule
- **SlotPicker** - Date picker + available time slots grid (public booking)
- **ServiceCard** - Service name, duration, price, colour indicator
- **AppointmentDetail** - Modal/drawer with full appointment info, status actions
- **AvailabilityGrid** - Weekly hours editor (drag to set time ranges per day)

---

## Key Business Rules

1. **Conflict detection**: No overlapping appointments for the same user (including buffer time)
2. **Cancellation deadline**: Clients can only cancel before `cancellation_hours` threshold
3. **Minimum notice**: Clients can't book with less than `min_notice_hours` lead time
4. **Timezone handling**: All times stored in UTC, displayed in business owner's configured timezone
5. **Public page rate limiting**: Max 10 booking requests per IP per hour
6. **Auto-status**: Appointments auto-transition to `completed` after end_time passes (daily cron)
7. **Cancellation tokens**: Randomly generated, single-use, expire after appointment time
