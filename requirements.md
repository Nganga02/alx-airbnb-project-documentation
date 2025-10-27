# alx-airbnb-project-documentation requirements


## Feature 1 — User Authentication & Authorization

### Functional Requirements
- User registration (email + password) with optional role (`guest` as default; `host` upopn switch).
- Email verification flow (token-based).
- Login returns JWT access token and refresh token.
- Password reset via email (token-based).
- Get current user profile; update profile.
- Role-based authorization (RBAC): `guest`, `host`, `admin`.

### REST API Endpoints (REST-only)

#### 1.1 Register
- **POST** `/api/v1/auth/register`
- **Description:** Create a new user account; sends verification email (async).

**Request**
```json
{
  "first_name": "Alice",
  "last_name": "Mwangi",
  "email": "alice@example.com",
  "password": "Str0ngPass!",
  "phone_number": "+254712345678",
  "role": "guest"            // optional: "guest" | "host"
}
```

Successful Response (201 Created)

```json
{
  "user_id": "c2f5e3b4-...-abcd",
  "message": "User created. Verification email sent."
}
```
#### 1.2 Verify Email

- GET /api/v1/auth/verify?token=<verification_token>

- Description: Verifies user email.

Successful Response (200 OK)

```json
{
  "message": "Email verified. You may now log in."
}
```

#### 1.3 Login

- POST /api/v1/auth/login

- Description: Authenticate and return tokens.

Request
```json
{
  "email": "alice@example.com",
  "password": "Str0ngPass!"
}
```

Successful Response (200 OK)

```json
{
  "access_token": "<jwt_access_token>",
  "expires_in": 3600,
  "refresh_token": "<opaque_refresh_token>",
  "token_type": "Bearer",
  "user": {
    "user_id": "c2f5e3b4-...-abcd",
    "first_name": "Alice",
    "last_name": "Mwangi",
    "email": "alice@example.com",
    "role": "guest"
  }
}
```
#### 1.4 Refresh Token

- POST /api/v1/auth/refresh

Request
```json
{ "refresh_token": "<opaque_refresh_token>" }
```
Response
```json
{ "access_token": "<new_jwt>", "expires_in": 3600 }
```

#### 1.5 Logout (Revoke Refresh Token)

POST /api/v1/auth/logout

- Header: Authorization: Bearer <access_token>

- Body:
```json
{ "refresh_token": "<opaque_refresh_token>" }
```
    Response: 204 No Content

#### 1.6 Get Profile

    GET /api/v1/users/me

    Header: Authorization: Bearer <access_token>

    Response (200)

```json
{
  "user_id": "...",
  "first_name": "...",
  "last_name": "...",
  "email": "...",
  "phone_number": "+2547...",
  "role": "guest",
  "created_at": "2025-10-01T12:00:00Z"
}
```

#### 1.7 Update Profile

    PUT /api/v1/users/me

    Header: Authorization: Bearer <access_token>

    Request (partial fields allowed)

```json
{ "first_name": "Alice", "phone_number": "+254711111111" }
```
    Response (200) updated user object.

#### Input Validation Rules

- **email**: must have an @ and domain name

- **password**: minimum 8 characters, at least one uppercase, one lowercase, one digit, one special char (recommend bcrypt with cost >= 12).

- **first_name/last_name**: non-empty strings, max length 100.

- **phone_number**: E.164 format if provided.

- **role**: one of guest, host, admin (admin creation restricted to system ops).

#### Error Responses (examples)

    400 Bad Request — invalid input (JSON body contains error field with details).

    409 Conflict — email already registered.

    401 Unauthorized — invalid credentials or token.

    403 Forbidden — role-based access denied (e.g., host-only endpoint).

    429 Too Many Requests — rate-limiting.

#### Security & Non-functional Requirements

    Password storage: hashed using bcrypt/argon2; never return raw password.

    JWT: RS256 recommended (asymmetric) with 1h expiry; short-lived tokens; refresh tokens opaque and stored server-side (or use rotating refresh tokens).

    Brute-force protection: lockout or exponential backoff after 5 failed attempts per account or IP.

    Response time: Auth endpoints should respond < 300ms under normal load (p99 < 500ms).

    Logging: Log failed/successful auth attempts with IP and timestamp (PII handling policies).

    Account verification email: Use background worker (Celery, Sidekiq, etc.) with retry.

## Feature 2 — Booking System

### Functional Requirements

- Guests search listings, select dates, and create bookings.

- System prevents double-booking via availability checks.

- Bookings have lifecycle: pending → confirmed → cancelled → completed.

- Hosts can view and manage bookings for their properties.

- Guests can view and cancel bookings subject to policy (cancellation windows).

- Booking must interact with Payment service to confirm payment before finalizing.

### REST API Endpoints

#### 2.1 Create Booking

    POST /api/v1/bookings

    Header: Authorization: Bearer <access_token> (guest)

    Request
```json
{
  "property_id": "7a8b9c...-uuid",
  "start_date": "2025-12-01",
  "end_date": "2025-12-05",
  "guests": 2,
  "payment_method_id": "pm_abc123"    // optional placeholder; actual payment may be processed via separate /payments endpoint
}
```

Processing Steps (server logic)

- Validate input (dates, guests count).

- Verify user is authenticated and not the host of the property.

-  Check availability:

    Query bookings for overlapping confirmed or pending bookings for the property in range [start_date, end_date).

    Apply availability rules (e.g., buffer days).

- Reserve booking slot by creating booking record with status = "pending_payment" (or pending).

- Return booking id and next action (e.g., proceed_to_payment).

Response (201 Created)
```json
{
  "booking_id": "b1c2d3...-uuid",
  "status": "pending_payment",
  "total_price": 4800.00,
  "currency": "KES",
  "next_step": "/api/v1/payments/charge?booking_id=b1c2d3..."
}
```

#### 2.2 Get Booking Details

    GET /api/v1/bookings/{booking_id}

    Authorization: Access allowed for booking guest, host (owner of property), or admin.

    Response (200)
```json
{
  "booking_id": "...",
  "property_id": "...",
  "user_id": "...",
  "start_date": "2025-12-01",
  "end_date": "2025-12-05",
  "status": "confirmed",
  "total_price": 4800.00,
  "currency": "KES",
  "created_at": "2025-10-20T10:00:00Z"
}
```
2.3 Cancel Booking

    PUT /api/v1/bookings/{booking_id}/cancel

    Header: Authorization: Bearer <access_token>

    Rules: Can be invoked by guest (subject to cancellation policy) or host/admin (subject to business logic).

    Response (200) updated booking object with status: "cancelled" and refund instructions if applicable.

2.4 List Bookings (User)

    GET /api/v1/users/me/bookings?page=1&page_size=20

    Response: Paginated list of bookings for the authenticated user.

2.5 List Bookings (Host)

    GET /api/v1/hosts/me/bookings?property_id=<id>&from=&to=

    Authorization: Host-only; returns bookings for host's properties.

Request / Response Examples

(See above endpoints; responses include booking_id, status, price fields.)
Validation Rules & Business Logic

__Date ranges__: end_date > start_date; minimum stay rules (e.g., min 1 night).

__Availability check__: Overlap detection must treat bookings as half-open intervals [start_date, end_date) to avoid date collisions.

__Guests limit__: Must not exceed property max_guests.

__Host cannot book own property__: If user_id == property.host_id, return 403 Forbidden.

__Idempotency__: For payment-triggered booking creation, support idempotency keys to avoid duplicate bookings due to retries.

__Concurrency__: Use transactional locks or serialized checks when creating bookings to prevent race conditions (e.g., SELECT ... FOR UPDATE, optimistic locking with unique constraints on date ranges where feasible).

__Cancellation policy__: Implement business rules (full refund if canceled within X days, partial refund, no refund).

__Booking lifecycle enforcement__: Only certain transitions allowed (e.g., pending → confirmed or cancelled; confirmed → completed after checkout).

Error Responses

    400 Bad Request — invalid dates or missing fields.

    403 Forbidden — unauthorized actor for the booking or host booking own property.

    409 Conflict — availability conflict (double-booking).

    404 Not Found — property or booking not found.

    429 Too Many Requests — rate limiting.

Security & Performance

__Auth__: Auth required for creating bookings.

__ACID guarantees__: Booking creation should be atomic with availability check and booking insert/update; use DB transactions.

__Latency target__: Booking creation (including availability check, DB write) should complete within < 500ms under typical load; p95 < 800ms.

__Scalability__: Availability checks should use indexed queries (index on property_id, start_date, end_date) and caching for read-heavy operations (e.g., property calendar snapshots).

__Background tasks__: Send notifications (email/SMS) asynchronously; do not block booking API response.

__Audit__: Record actor, IP, user agent for all booking actions.

## Feature 3 — Payment & Billing
### Functional Requirements

1. Securely accept payments for bookings via external payment providers (Stripe/PayPal) or local payment rails.

1. Record payment attempts and final transaction status.

1. Update booking status upon successful payment.

1. Support refunds and partial refunds where policy allows.

1. Maintain transaction history for audit and reconciliation.

### REST API Endpoints
#### 3.1 Initiate Payment (Create Payment Intent)

    POST /api/v1/payments/initiate

    Header: Authorization: Bearer <access_token>

    Request
```json
{
  "booking_id": "b1c2d3...-uuid",
  "payment_method": "stripe_card",   // or "mpesa", "paypal"
  "currency": "KES",
  "amount": 4800.00,
  "return_url": "https://client.app/booking/confirm"   // for redirect-based methods
}
```
Processing

- Verify booking exists and is pending_payment.

- Create a payment record in payments table with status = "initiated".

- Create a provider-specific payment intent (call provider API).

- Return provider response/details to client (e.g., client_secret for Stripe).

Response (201 Created)
```json
{
  "payment_id": "pay_abc123-uuid",
  "status": "initiated",
  "provider_client_secret": "...",   // when applicable
  "provider_redirect_url": null
}
```
3.2 Confirm/Complete Payment (Webhook / Callback)

    POST /api/v1/payments/webhook (public endpoint used by payment provider)

    Processing

       { Validate webhook signature.

        Update payment record (status: succeeded / failed).

        On succeeded:

            Update booking status to confirmed.

            Store provider transaction id, amount, fees.

            Enqueue confirmation notifications.

        On failed:

            Update payment status = failed; booking remains pending or payment_failed.}

Response: 200 OK to acknowledge webhook.
#### 3.3 Refund Payment

    POST /api/v1/payments/{payment_id}/refund

    Header: Authorization: Bearer <access_token> (admin or system)

    Request
```json
{ "amount": 4800.00, "reason": "guest_cancellation" }

    Processing: Call provider refund API; record refund transaction.

Response (200)

{ "refund_id": "refund_xyz", "status": "processing" }
```
#### 3.4 Get Payment Details

    GET /api/v1/payments/{payment_id}

    Authorization: Booking owner, host (if appropriate), or admin.

    Response
```json
{
  "payment_id": "...",
  "booking_id": "...",
  "status": "succeeded",
  "amount": 4800.00,
  "currency": "KES",
  "provider_transaction_id": "...",
  "created_at": "2025-10-22T12:00:00Z"
}
```
Validation Rules & Business Logic

1. __Amount verification__: Payment amount must match booking total (server-side authoritative).

1. __Idempotency__: All payment creation endpoints must accept idempotency keys to prevent duplicate charges.

1. __Webhook security__: Validate provider signatures and replay protections.

1. __Refund policy__: Enforce business rules and partial refunds limits.

1. __Settlement__: For host payouts, record transfer records separately; payouts may be scheduled and executed via provider-managed transfers.

Error Responses

    400 Bad Request — invalid request fields.

    402 Payment Required — user attempted action but payment required/failed.

    409 Conflict — payment mismatch or duplicate payment.

    422 Unprocessable Entity — provider-specific validation errors.

    500 Internal Server Error — provider connectivity issues (retry with backoff).

Security & Non-functional Requirements

1. __PCI Compliance__: Do not store raw card data; use tokenization and provider SDKs. Follow best practices for PCI-DSS where applicable.

1. __Response Times__: Initiating payment should return the provider client_secret / redirect URL within < 800ms (network dependent).

1. __Reliability__: Webhook handling must be idempotent and durable (use acknowledgements & retries).

1. __Audit Trail__: Store provider transaction ids, fees, and payout links for reconciliation.

1. __Retries & Backoff__: For transient provider errors, implement exponential backoff; do not retry indefinitely.

1. __Encryption__: All storage of payment-related sensitive metadata must be encrypted at rest.

### Cross-Cutting Concerns
#### Authentication & Authorization

All protected endpoints require Authorization header with valid JWT.

Role checks enforced on endpoints (e.g., property creation: host role).

Logging & Monitoring

- Log structured events for important actions: registration, login attempts, booking create/update/cancel, payment events.

- Instrument metrics: request counts, latencies (p50/p95/p99), error rates per endpoint.

- Integrate with monitoring tools (Prometheus / Grafana / Cloud provider equivalents).

Rate Limiting & Abuse Prevention

- Auth endpoints: max 5 login attempts per minute per IP with progressive backoff.

- General API: 100 requests/min per IP as baseline.

- Burst allowances via token buckets supported.

Data Retention & Privacy

- Comply with data retention policies: personal data retention & deletion flows (GDPR-style: "right to be forgotten" endpoints).

- Anonymize logs to avoid PII leakage.

Testing & Validation

- Provide unit tests for business logic (availability checks, payment reconciliation).

- Integration tests for end-to-end booking + payment flows (use sandbox provider APIs).

- Contract tests for webhooks (signature and payload validation).

Appendix: HTTP Status & Error Codes (Guideline)

    200 OK — successful GET/PUT/POST (when no resource created)

    201 Created — successful resource creation

    204 No Content — successful operation with no body (e.g., logout)

    400 Bad Request — validation or malformed request

    401 Unauthorized — missing/invalid auth

    403 Forbidden — insufficient permissions

    404 Not Found — resource not found

    409 Conflict — resource conflict (duplicate, availability collisions)

    422 Unprocessable Entity — semantic errors with correct format

    429 Too Many Requests — rate limiting