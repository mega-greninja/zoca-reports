# Booking Platform API Reference

Consolidated API reference across all 8 salon-booking platforms used by Zoca's AI Frontdesk. Each platform's access model, auth, endpoint list by function, and key request examples.

---

## Quick tier map

| Platform | Access model | Auth | Webhooks |
|---|---|---|---|
| **Square** | Public | OAuth2 Bearer token | Yes — rich event set |
| **Mindbody** | Public (v6) | API-Key + SiteId + user token | Yes — 30+ events |
| **Acuity** | Public | HTTP Basic (userID:apiKey) | Yes — 5 events |
| **Phorest** | Partner API (request access) | HTTP Basic (`global/<email>`:pw) | **No — must poll** |
| **Vagaro** | Enterprise partner-gated ($10/mo + per-call) | OAuth2 client-credentials | Yes — rich event set |
| **GlossGenius** | **No public API** | — | — |
| **Booksy** | **No public API** | — | — |
| **Fresha** | **No public API** (GraphQL internal only) | — | — |

---

# 1. Square

**Base:** `https://connect.squareup.com` · all endpoints `v2`
**Auth:** `Authorization: Bearer {ACCESS_TOKEN}` + `Square-Version: 2024-XX-XX` + `Content-Type: application/json`
**OAuth:** `GET https://connect.squareup.com/oauth2/authorize` → `POST /oauth2/token`
**Scopes:** `APPOINTMENTS_READ/WRITE APPOINTMENTS_ALL_READ APPOINTMENTS_BUSINESS_SETTINGS_READ CUSTOMERS_READ/WRITE ITEMS_READ PAYMENTS_READ/WRITE PAYMENTS_WRITE_IN_PERSON EMPLOYEES_READ GIFTCARDS_READ/WRITE LOYALTY_READ/WRITE SUBSCRIPTIONS_READ/WRITE INVOICES_READ/WRITE ORDERS_READ/WRITE`

### Services (Catalog API)
- `GET /v2/catalog/list?types=ITEM` — list items
- `GET /v2/catalog/object/{id}` — get service/variation
- `GET /v2/catalog/list?types=CATEGORY` — list categories
- `POST /v2/catalog/search-catalog-items` — filter by category/name
- `POST /v2/catalog/search` — generic catalog search
- `POST /v2/catalog/batch-retrieve`

### Staff
- `POST /v2/team-members/search`
- `GET /v2/team-members/{id}`
- `GET /v2/bookings/team-member-booking-profiles`
- `GET /v2/bookings/team-member-booking-profiles/{id}`
- `POST /v2/bookings/team-member-booking-profiles/bulk-retrieve`
- `GET /v2/bookings/location-booking-profiles`

### Locations
- `GET /v2/locations` — includes `business_hours.periods`
- `GET /v2/locations/{id}`

### Availability
- `POST /v2/bookings/availability/search`

### Bookings
- `POST /v2/bookings` — create
- `GET /v2/bookings/{id}` — retrieve
- `PUT /v2/bookings/{id}` — update
- `POST /v2/bookings/{id}/cancel` — cancel
- `GET /v2/bookings` — list
- `POST /v2/bookings/bulk-retrieve`

### Customers
- `POST /v2/customers` · `POST /v2/customers/search` · `GET /v2/customers/{id}` · `PUT /v2/customers/{id}` · `DELETE /v2/customers/{id}` · `GET /v2/customers`

### Cards on File
- `POST /v2/cards` — save (needs Web SDK nonce)
- `GET /v2/cards?customer_id=` — list for customer
- `GET /v2/cards/{id}` · `POST /v2/cards/{id}/disable`

### Payments / Deposits
- `POST /v2/payments` — charge (use `autocomplete:false` for auth-only / delayed capture)
- `POST /v2/payments/{id}/complete` · `/cancel`
- `POST /v2/orders` — create order, link via `order_id` for deposits
- `POST /v2/refunds` — refund

### Gift Cards
- `POST /v2/gift-cards` · `POST /v2/gift-cards/from-gan` (lookup) · `GET /v2/gift-cards/{id}`
- `POST /v2/gift-cards/activities` — type `REDEEM` for redemption

### Loyalty
- `GET /v2/loyalty/programs/main`
- `POST /v2/loyalty/accounts` · `POST /v2/loyalty/accounts/search`
- `POST /v2/loyalty/accounts/{id}/accumulate` · `/adjust`
- `POST /v2/loyalty/rewards` · `POST /v2/loyalty/rewards/{id}/redeem`

### Subscriptions / Memberships
- `POST /v2/subscriptions` · `/search` · `GET/PUT /v2/subscriptions/{id}` · `/cancel` `/pause` `/resume` `/swap-plan`

### Inventory
- `POST /v2/inventory/counts/batch-retrieve` · `/changes/batch-create`

### Invoices
- `POST /v2/invoices` · `/search` · `GET/PUT /v2/invoices/{id}` · `/publish` · `/cancel`

### Webhooks
- `POST /v2/webhooks/subscriptions` · `GET` · `PUT` · `DELETE` · `/test` · `/event-types`
- **Key events:** `booking.created/updated`, `customer.created/updated/deleted`, `payment.created/updated`, `refund.created/updated`, `invoice.*`, `catalog.version.updated`, `order.*`, `team_member.*`, `gift_card.*`, `loyalty.*`, `subscription.*`
- Signature header: `x-square-hmacsha256-signature`

### Example: Create Booking
```json
POST /v2/bookings
{
  "idempotency_key": "a1b2c3",
  "booking": {
    "start_at": "2026-04-22T15:00:00Z",
    "location_id": "L123",
    "customer_id": "CUST_XYZ",
    "appointment_segments": [{
      "duration_minutes": 60,
      "service_variation_id": "VAR_ABC",
      "service_variation_version": 1712000000000,
      "team_member_id": "TM_1"
    }]
  }
}
```

### Not in API
- Rooms/resources (not exposed)
- Reviews / feedback
- Packages as first-class (model via catalog items + multi-segment bookings)

---

# 2. Mindbody (Public API v6)

**Base:** `https://api.mindbodyonline.com/public/v6`
**Auth headers on every call:** `Api-Key: <dev key>` + `SiteId: <numeric>` + `Authorization: <user token>`
**Token issue:** `POST /usertoken/issue` (body: `{Username, Password}`) → Staff or Client credentials
**Activate first:** `POST /site/activatesite`

### Session Types / Programs / Services
- `GET /site/sessiontypes`
- `GET /site/programs`

### Staff
- `GET /staff/staff`
- `GET /staff/staffpermissions`
- `GET /appointment/staffappointments`

### Sites / Locations / Resources
- `GET /site/sites`
- `GET /site/locations`
- `GET /site/resources` — rooms/equipment

### Availability
- `GET /appointment/bookableitems`
- `GET /appointment/scheduleitems`
- `GET /class/classes`
- `GET /class/classschedules`
- `GET /client/activeclientmemberships`

### Appointments
- `POST /appointment/addappointment`
- `POST /appointment/updateappointment`
- `GET /appointment/appointmentoptions` — deposit/cancel rules

### Classes
- `POST /class/addclienttoclass` · `/removeclientfromclass` · `/substituteclassteacher`
- `GET /class/waitlistentries`
- `POST /class/addclienttoclasswaitlist` · `/removefromwaitlist`

### Clients
- `POST /client/addclient` · `/updateclient`
- `GET /client/clients` · `/clientcontracts` · `/clientservices` · `/clientvisits` · `/clientpurchases` · `/clientaccountbalances` · `/clientformulanotes`
- `POST /client/sendpasswordresetemail`

### Cards on File
- `POST /client/addorupdateclientcreditcard`

### Sales / Cart / Checkout
- `POST /sale/checkoutshoppingcart` — atomic cart checkout
- `GET /sale/sales` · `/contracts` · `/giftcards` · `/packages` · `/products` · `/services` · `/acceptedcardtypes`
- `POST /sale/purchasegiftcard` · `/purchasecontract`
- `POST /sale/returnsale` — refund

### Enrollments (multi-session courses)
- `GET /enrollment/enrollments`
- `POST /enrollment/addclienttoenrollment` · `/removeclientfromenrollment`

### Rewards
- `GET /client/clientrewards`

### Webhooks (separate API)
**Base:** `https://mb-api.mindbodyonline.com/push/api/v1`
- `POST /subscriptions` · `GET` · `GET /{id}` · `PATCH /{id}` · `DELETE /{id}`
- `GET /messages` — delivered messages
- **Key events:** `appointmentBooking.created/updated/cancelled`, `appointmentAddOn.created/deleted`, `classSchedule.*`, `class.updated`, `classRosterBooking.created/cancelled`, `classWaitlistRequest.*`, `client.created/updated/deactivated`, `clientMembershipAssignment.created/cancelled`, `clientSale.created`, `staff.*`, `location.*`, `site.*`

### Example: AddAppointment
```json
POST /public/v6/appointment/addappointment
{
  "ClientId": "100012345",
  "LocationId": 1,
  "SessionTypeId": 25,
  "StaffId": 100000123,
  "StartDateTime": "2026-04-22T14:00:00",
  "Duration": 60,
  "SendEmail": true,
  "ApplyPayment": false
}
```

---

# 3. Acuity Scheduling

**Base:** `https://acuityscheduling.com/api/v1`
**Auth:** HTTP Basic `Authorization: Basic base64(userID:apiKey)` (from Acuity → Integrations → API)
**OAuth2** also available for third-party apps.

### Appointment Types (Services)
- `GET /appointment-types`
- `GET /appointment-types/{id}/form-fields`

### Calendars (Staff)
- `GET /calendars`

### Availability
- `GET /availability/dates?month=&appointmentTypeID=&calendarID=&timezone=`
- `GET /availability/times?date=&appointmentTypeID=&calendarID=&timezone=`
- `GET /availability/classes?month=&appointmentTypeID=`
- `GET /availability/check-times`

### Appointments
- `GET /appointments` — filters: minDate, maxDate, calendarID, appointmentTypeID, canceled
- `GET /appointments/{id}`
- `POST /appointments` (supports `?admin=true` to bypass availability)
- `PUT /appointments/{id}/reschedule`
- `PUT /appointments/{id}/cancel`
- `PUT /appointments/{id}` — update forms/labels/notes
- `GET /appointments/{id}/payments` · `POST /appointments/{id}/payments`

### Clients
- `GET /clients?search=`
- `POST /clients` — standalone, not tied to appointment

### Forms
- `GET /forms`

### Gift Certificates / Coupons
- `GET /certificates` · `POST /certificates` · `DELETE /certificates/{id}`
- `GET /certificates/check?certificate=&appointmentTypeID=`

### Products / Orders
- `GET /products`
- `GET /orders` · `GET /orders/{id}`

### Webhooks
- `GET /webhooks` · `POST /webhooks` · `DELETE /webhooks/{id}`
- **Events:** `appointment.scheduled`, `appointment.rescheduled`, `appointment.canceled`, `appointment.changed`, `order.completed`

### Example: Create Appointment
```json
POST /api/v1/appointments
{
  "datetime": "2026-05-15T14:00:00-0700",
  "appointmentTypeID": 123456,
  "calendarID": 78910,
  "firstName": "Jane",
  "lastName": "Doe",
  "email": "jane@example.com",
  "phone": "+15551234567",
  "fields": [{"id": 1111, "value": "First visit"}]
}
```

---

# 4. Phorest (Partner API)

**Base:** `https://api-gateway-eu1.phorest.com/third-party-api-server/api` (US region has its own host)
**Auth:** HTTP Basic `Authorization: Basic base64(global/<email>:password)`
**Webhooks:** **Not supported — poll instead.**

### Services
- `GET /business/{bId}/branch/{brId}/service`
- `GET /business/{bId}/branch/{brId}/servicecategory`
- `GET /business/{bId}/branch/{brId}/service/specialoffer`
- `GET /business/{bId}/branch/{brId}/servicepackage`

### Staff
- `GET /business/{bId}/branch/{brId}/staff/{staffId}`
- `GET /business/{bId}/branch/{brId}/staff/{staffId}/break/{breakId}`

### Branches
- `GET /business/{bId}/branch`

### Availability
- `POST /business/{bId}/branch/{brId}/appointments/availability`

### Appointments
- `GET /business/{bId}/branch/{brId}/appointment`
- `POST /business/{bId}/branch/{brId}/appointment/checkin`
- `POST /business/{bId}/branch/{brId}/appointment/cancel`

### Bookings (primary write flow)
- `POST /business/{bId}/branch/{brId}/booking` — create (often `RESERVED` status first)
- `POST /business/{bId}/branch/{brId}/booking/{bookingId}/activate` — activate (accepts `depositAmount`)
- `POST /business/{bId}/branch/{brId}/booking/{bookingId}/cancel`
- `POST /business/{bId}/branch/{brId}/booking/{bookingId}/note`

### Clients
- `GET /business/{bId}/client` · `GET /business/{bId}/client/{clientId}`
- `POST /business/{bId}/client` · `PUT /business/{bId}/client/{clientId}`
- `POST /business/{bId}/client/{clientId}/loyaltypoints`

### Purchases / Sales
- `POST /business/{bId}/branch/{brId}/purchase`
- `GET /business/{bId}/branch/{brId}/supplementalpaymenttype`

### Vouchers / Gift Cards
- `GET /business/{bId}/voucher` · `POST /business/{bId}/voucher`
- Redemption flows through `/purchase`

### Courses
- `GET /business/{bId}/clientcourse`
- `PUT /business/{bId}/clientcourse/{id}`

### Reviews
- `GET /business/{bId}/branch/{brId}/reviews`

### Examples
```json
// Availability
POST /appointments/availability
{
  "startTime":"2026-04-21T09:00:00Z",
  "endTime":"2026-04-21T18:00:00Z",
  "clientId":"clientAbc",
  "clientServiceSelections":[{"clientId":"clientAbc","serviceIds":["svc-123"]}],
  "isOnlineAvailability":true
}

// Create booking (reserved)
POST /booking
{
  "bookingStatus":"RESERVED",
  "clientId":"clientAbc",
  "clientAppointmentSchedules":[{
    "clientId":"clientAbc",
    "clientAppointments":[{
      "serviceId":"svc-123","staffId":"staff-9",
      "startTime":"2026-04-21T14:00:00Z"
    }]
  }]
}

// Activate with deposit
POST /booking/{id}/activate
{"depositAmount": 25.00}
```

---

# 5. Vagaro (Enterprise Partner API V2)

**Access:** Paid merchants only, credentials provisioned via Vagaro Enterprise Sales. **$10/mo base** + 5,000 calls included + **$0.002/call** over.
**Enable:** Merchant UI → Settings → Developers → APIs & Webhooks
**Docs:** https://docs.vagaro.com/

**Base:** `https://api.vagaro.com/{region}/api/v2`
**Auth:** OAuth2 client-credentials
- `POST /merchants/generate-access-token` — body: `{clientId, clientSecretKey, scope}` → returns token
- Subsequent calls pass `accessToken:` **as a header** (not Bearer)
- Rate-limit: 429

### Documented endpoints (most are **POST** with JSON body)
- `POST /merchants/generate-access-token`
- `POST /merchants/employees/assign` · `/unassign`
- `POST /locations` — retrieve business locations
- `POST /employees` — retrieve employee by `businessId` + `serviceProviderId`
- `POST /customers` — retrieve customer
- `POST /services` — retrieve services
- `POST /appointments` — retrieve appointments

### Not publicly documented (gaps)
- Availability / slots search
- Appointment create / update / cancel
- Transactions read
- Memberships / gift cards / products reads
Partner-specific access required for write flows (book on customer's behalf).

### Webhooks (full coverage)
- `appointment.created/updated/deleted`
- `customer.created/updated`
- `employee.created/updated/deleted`
- `business_location.created/updated/deleted`
- `transaction.created`
- `formResponse.created`

Receiver: HTTPS, accept JSON POST, return 2xx within 20s; up to 5 retries over 15 min.

**Appointment payload fields:** `appointmentId, startTime/endTime, bookingStatus, serviceId/Title, customerId, serviceProviderId, businessId/Alias, amount, eventType (Appointment|Class), onlineVsInhouse, appointmentTypeCode/Name (new|returning), bookingSource (Google|Yelp|Facebook|…), createdDate/modifiedDate, formResponseIds[]`

---

# 6. GlossGenius

**Status: NO public API.**

- `glossgenius.com/developers`, `/api`, `/partners` → 404 / no docs
- GlossGenius GitHub has no SDK
- Confirmed via Goodcall/Smith.ai/industry sources

### Private API (reverse-engineered, ToS risk)
- Host: `api.glossgenius.com` — gateway is live
- Consumer booking site: `glossgenius.com/<slug>/booking` (Next.js, unauthenticated public booking flow)
- Inferred endpoint shape (confirm via Playwright network capture):
  - `GET /v1/public/businesses/{slug}`
  - `GET /v1/public/businesses/{slug}/services`
  - `GET /v1/public/businesses/{slug}/staff`
  - `GET /v1/public/availability?business_id=&service_id=&staff_id=&date=`
  - `POST /v1/public/bookings` (requires Stripe PaymentMethod token for deposit-required businesses)
- Auth: JWT bearer for merchant app; public booking flow is anonymous per slug

### Sanctioned backdoor
- **Google Calendar sync** — if a Zoca client grants GCal OAuth, booked GG appointments appear as events. Only official read path.

### Recommended Zoca path
1. Continue directing bookings through public `glossgenius.com/<slug>/booking` URLs (already captured in `~/glossgenius_booking_links.csv`).
2. For internal bot flows, capture the booking network trace in Playwright, then script against the private endpoints treating them as an undocumented contract.

---

# 7. Booksy

**Status: NO public API. No partner API program.**

- `developer.booksy.com` / `booksy.com/api` / `/partners` — no docs
- Booksy has a Partner/Affiliate referral program (not API)

### Private API (reverse-engineered)
- Hosts: `https://us.booksy.com/api/us/2/customer_api/...` (consumer) · `https://biz.booksy.com/api/...` (merchant)
- Version prefix: `/2/` · `customer_api` vs `business_api`

### Observed paths (subject to change)
- `GET /businesses?query=&location=`
- `GET /businesses/{id}`
- `GET /businesses/{id}/services`
- `GET /businesses/{id}/staffers`
- `GET /businesses/{id}/appointments/time_slots?service_variant_id=&staffer_id=&date=`
- `GET /businesses/{id}/reviews`
- `POST /me/login`
- `POST /me/appointments`

### Auth pattern (private)
- Headers: `x-api-key` (static app key from mobile client), `x-language`, `x-fingerprint` (device ID), mobile-app `User-Agent`
- User session: Bearer JWT after `POST /me/login`
- Anonymous browse works with just the `x-api-key`
- Cloudflare + bot detection — aggressive scraping → 403/429

### Third-party integrations
- Reserve with Google — official, but closed pipe (no endpoints exposed to devs)
- Zapier — no official app as of 2026
- QuickBooks — CSV export only

**Only path for programmatic access: reverse-engineered `us.booksy.com/api/us/2/customer_api/...` with scraped `x-api-key`.**

---

# 8. Fresha

**Status: NO public or partner API.** Asked for 6+ years publicly, never shipped.

- `developer.fresha.com` — doesn't resolve
- `partners.fresha.com` — affiliate login only, not dev portal
- GitHub `fresha` org: internal OpenAPI tooling only, no SDKs or service schemas

### Private API (reverse-engineered)
- **Primary: GraphQL** — `POST https://www.fresha.com/graphql`
  - Introspection disabled, but error messages leak field names
  - Observed root Query fields include: `location`, `locations`, `liteLocation`, `geolocation`, `searchHistory`
  - Subscriptions gateway: `https://api-gateway-ws.fresha.com/graphql`
- **Secondary JSON:API microservices** (`Accept: application/vnd.api+json`):
  - `https://api.fresha.com` — root gateway
  - `https://analytics-api.fresha.com`
  - `https://deals-api.fresha.com`
  - `https://gift-cards-api.fresha.com`
  - `https://payments.fresha.com`
  - `https://refresh.fresha.com` — token refresh
  - `https://b2c-web-ff.fresha.com/proxy` — feature flags

### Auth (private)
- Bearer JWT, refreshed via `refresh.fresha.com`
- Anonymous sessions minted on first page load (cookies + JWT)
- Same JWT scheme for authenticated consumer + business sessions

### Third-party data access
- Reserve with Google — Google-side only, no endpoints exposed
- Zapier — no Fresha app
- Only practical extraction: **scraping public marketplace pages** (e.g. Apify has a Fresha lead extractor) or reverse-engineering GraphQL (ToS risk)

---

# Zoca implementation pattern

For the AI Frontdesk, use this per-tier routing:

- **🟢 Open (Square, Mindbody, Acuity)** → full API flow: search availability → create booking → native card-on-file / deposit → webhooks drive Stage-2 status updates
- **🟡 Partner (Phorest, Vagaro)** → Phorest: poll (no webhooks); Vagaro: need partner credentials, webhooks available
- **🔴 No API (GlossGenius, Booksy, Fresha)** → browser automation (Playwright) against private endpoints, our Stripe for all money, no webhooks (poll or scrape)

---

*Generated 2026-04-21. Verify endpoints against platform reference pages before production use — partner APIs change.*
