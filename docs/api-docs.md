# GateVault — API Documentation

**Base URL (Production):** `https://gatevault-agps.vercel.app`  
**Base URL (Development):** `http://localhost:3000`  
**API Prefix:** `/api`  
**Format:** All requests and responses use `application/json` unless noted otherwise.

---

## Table of Contents

1. [Overview](#overview)
2. [Authentication](#authentication)
3. [User Roles](#user-roles)
4. [Error Handling](#error-handling)
5. [Endpoints](#endpoints)
   - [Auth](#auth)
   - [User Registration](#user-registration)
   - [Gate Passes](#gate-passes)
   - [QR Verification](#qr-verification)
   - [Analytics](#analytics)
   - [Profile](#profile)

---

## Overview

GateVault's backend is built on **Next.js 16 App Router API routes**, which means all server-side logic lives inside `app/api/`. Each route is a TypeScript file exporting named HTTP method handlers (`GET`, `POST`, `PUT`, `DELETE`).

The API follows a straightforward RESTful structure. Resources are organized around two core entities — **Users** and **Gate Passes** — and access to every endpoint is gated by the requester's session role. All authentication is handled through **NextAuth.js v4**, which issues a signed JWT stored in a secure HTTP-only cookie.

---

## Authentication

GateVault uses session-based authentication managed by NextAuth.js. After a successful login — either via Google OAuth or email/password — a session cookie (`next-auth.session-token`) is set automatically by the browser.

Every protected API route validates this session on the server side using `getServerSession()`. There are no API keys or manual `Authorization` headers involved for the standard client flow.

**Public routes** (no session required):

| Route | Description |
|---|---|
| `/login` | Email/password and Google sign-in |
| `/signup` | Student self-registration |
| `/admin-signup` | Admin account creation |
| `/hod-signup` | HOD account creation |
| `/warden-signup` | Warden account creation |
| `/security-signup` | Security personnel account creation |
| `/api/auth/*` | All NextAuth internal handlers |

All other routes redirect unauthenticated requests to `/login` with a `callbackUrl` parameter preserving the original destination.

---

## User Roles

The application defines five distinct roles. Each role determines which endpoints a user can access and what actions they can perform.

| Role | Description |
|---|---|
| `student` | Submits gate pass or leave requests and tracks their status |
| `admin` | Full system access — manages users, passes, and reports |
| `hod` | Head of Department — reviews and approves/rejects pass requests for their department |
| `warden` | Hostel warden — reviews and approves/rejects outstation or hostel-related passes |
| `security` | Gate-level personnel — scans QR codes to verify passes at entry/exit points |

Role is assigned at the time of registration and encoded in the user's session token.

---

## Error Handling

All errors follow a consistent JSON structure:

```json
{
  "error": "A human-readable description of what went wrong"
}
```

Standard HTTP status codes are used throughout:

| Status Code | Meaning |
|---|---|
| `200 OK` | Request succeeded |
| `201 Created` | Resource was created successfully |
| `400 Bad Request` | Missing or invalid fields in the request body |
| `401 Unauthorized` | No valid session found |
| `403 Forbidden` | Authenticated but insufficient role/permissions |
| `404 Not Found` | The requested resource does not exist |
| `409 Conflict` | A resource with the same unique identifier already exists |
| `500 Internal Server Error` | Unexpected server-side failure |

---

## Endpoints

---

### Auth

NextAuth.js handles all authentication flows through a catch-all route. These endpoints are consumed automatically by the NextAuth client SDK and are not typically called directly.

---

#### `GET/POST /api/auth/[...nextauth]`

**Description:** Handles all authentication actions — sign-in, sign-out, session retrieval, CSRF token generation, and OAuth callbacks.

**Supported Providers:**
- Google OAuth 2.0
- Credentials (email + bcrypt-hashed password)

**Common sub-routes used internally:**

| Path | Purpose |
|---|---|
| `/api/auth/signin` | Initiates the sign-in flow |
| `/api/auth/signout` | Terminates the current session |
| `/api/auth/session` | Returns the active session data |
| `/api/auth/csrf` | Returns a CSRF token required for POST requests |
| `/api/auth/callback/google` | OAuth redirect callback from Google |
| `/api/auth/callback/credentials` | Handles credential-based login |

---

### User Registration

Each role has its own dedicated registration endpoint. This design keeps validation and role assignment explicit rather than relying on a single generic signup with a role parameter.

---

#### `POST /api/signup`

Registers a new student account.

**Request Body:**

```json
{
  "name": "Ananda Saikia",
  "email": "et22bthcs072@kazirangauniversity.in",
  "password": "securepassword@123456",
  "rollNumber": "ET22BTHCS072",
  "department": "Computer Science and Engineering",
  "year": 4
}
```

**Success Response — `201 Created`:**

```json
{
  "message": "Student account created successfully",
  "userId": "64f3a2bc9e1d4c001e8b4512"
}
```

**Error Responses:**

`400 Bad Request` — Required fields are missing or the email format is invalid.  
`409 Conflict` — An account with this email already exists.

---

#### `POST /api/admin-signup`

Registers a new admin account. In production, this endpoint should be restricted to internal use or protected by an invite token.

**Request Body:**

```json
{
  "name": "Bitupan Bhuyan",
  "email": "admin@kazirangauniversity.in",
  "password": "admin@123456"
}
```

**Success Response — `201 Created`:**

```json
{
  "message": "Admin account created successfully",
  "userId": "64f3a2bc9e1d4c001e8b4521"
}
```

---

#### `POST /api/hod-signup`

Registers a new Head of Department account.

**Request Body:**

```json
{
  "name": "Mousoomi Bora",
  "email": "mousoomi@kazirangauniversity.in",
  "password": "Hod@123456",
  "department": "Computer Science and Engineering"
}
```

**Success Response — `201 Created`:**

```json
{
  "message": "HOD account created successfully",
  "userId": "64f3a2bc9e1d4c001e8b4530"
}
```

---

#### `POST /api/warden-signup`

Registers a new warden account.

**Request Body:**

```json
{
  "name": "Mr. Prasanta Bora",
  "email": "prasanta@kazirangauniversity.in",
  "password": "wardenpassword@123456",
  "hostelBlock": "Orang"
}
```

**Success Response — `201 Created`:**

```json
{
  "message": "Warden account created successfully",
  "userId": "64f3a2bc9e1d4c001e8b4541"
}
```

---

#### `POST /api/security-signup`

Registers a new security personnel account.

**Request Body:**

```json
{
  "name": "Bikash Das",
  "email": "security1@kazirangauniversity.in",
  "password": "security1password@123456",
  "gateNumber": "Gate 1"
}
```

**Success Response — `201 Created`:**

```json
{
  "message": "Security account created successfully",
  "userId": "64f3a2bc9e1d4c001e8b4550"
}
```

---

### Gate Passes

Gate passes are the core resource of the application. The behaviour of each endpoint changes depending on the caller's role — a student sees only their own passes, while an admin sees all passes across the system.

---

#### `POST /api/passes`

Creates a new gate pass request. Only accessible by users with the `student` role.

**Request Body:**

```json
{
  "reason": "Family function",
  "destination": "Guwahati",
  "exitDate": "2026-05-15T08:00:00.000Z",
  "returnDate": "2026-05-17T20:00:00.000Z",
  "passType": "leave"
}
```

> `passType` can be `"gatepass"` (within city, same-day return) or `"leave"` (overnight/multi-day).

**Success Response — `201 Created`:**

```json
{
  "message": "Leave request submitted successfully",
  "passId": "64f4b3cd0e2d5d002f9c5623",
  "status": "pending"
}
```

**Error Responses:**

`400 Bad Request` — Missing required fields or invalid date range.  
`403 Forbidden` — Caller does not have the `student` role.

---

#### `GET /api/passes`

Retrieves gate passes. The result set is scoped by role:

- **Student:** Returns only the authenticated student's own passes.
- **HOD / Warden:** Returns passes pending their review, filtered by department or hostel block.
- **Admin:** Returns all passes in the system.
- **Security:** Returns currently active/approved passes for verification purposes.

**Query Parameters (optional):**

| Parameter | Type | Description |
|---|---|---|
| `status` | `string` | Filter by pass status: `pending`, `approved`, `rejected`, `expired` |
| `page` | `number` | Page number for pagination (default: `1`) |
| `limit` | `number` | Results per page (default: `10`) |

**Example Request:**

```
GET /api/passes?status=pending&page=1&limit=10
```

**Success Response — `200 OK`:**

```json
{
  "passes": [
    {
      "_id": "64f4b3cd0e2d5d002f9c5623",
      "studentName": "Ananda Saikia",
      "rollNumber": "ET22BTHCS072",
      "reason": "Family function",
      "destination": "Guwahati",
      "exitDate": "2026-05-15T08:00:00.000Z",
      "returnDate": "2026-05-17T20:00:00.000Z",
      "passType": "leave",
      "status": "pending",
      "createdAt": "2026-05-10T12:34:56.000Z"
    }
  ],
  "total": 1,
  "page": 1,
  "totalPages": 1
}
```

---

#### `GET /api/passes/:id`

Retrieves a single gate pass by its ID. Students can only fetch their own passes; admin, HOD, and warden can fetch any pass.

**URL Parameter:**

`id` — The MongoDB ObjectId of the gate pass.

**Success Response — `200 OK`:**

```json
{
  "_id": "64f4b3cd0e2d5d002f9c5623",
  "studentId": "64f3a2bc9e1d4c001e8b4512",
  "studentName": "Ananda Saikia",
  "rollNumber": "ET22BTHCS072",
  "department": "Computer Science and Engineering",
  "reason": "Family function",
  "destination": "Guwahati",
  "exitDate": "2026-05-15T08:00:00.000Z",
  "returnDate": "2026-05-17T20:00:00.000Z",
  "passType": "leave",
  "status": "approved",
  "approvedBy": "64f3a2bc9e1d4c001e8b4530",
  "approvedAt": "2026-05-11T09:15:00.000Z",
  "qrToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "createdAt": "2026-05-10T12:34:56.000Z"
}
```

**Error Responses:**

`404 Not Found` — No pass exists with the given ID.  
`403 Forbidden` — Student attempting to access another student's pass.

---

#### `PUT /api/passes/:id`

Updates the status of a gate pass. Used by HOD, Warden, or Admin to approve or reject a pending request. Students may use this endpoint to cancel their own pending passes.

**URL Parameter:**

`id` — The MongoDB ObjectId of the gate pass.

**Request Body (HOD / Warden / Admin):**

```json
{
  "status": "approved",
  "remarks": "Approved. Student must return by the stated date."
}
```

> `status` accepts `"approved"` or `"rejected"`.

**Request Body (Student — cancellation only):**

```json
{
  "status": "cancelled"
}
```

**Success Response — `200 OK`:**

```json
{
  "message": "Leave status updated successfully",
  "passId": "64f4b3cd0e2d5d002f9c5623",
  "status": "approved"
}
```

**Error Responses:**

`403 Forbidden` — Caller lacks the permission to update this pass.  
`404 Not Found` — Pass does not exist.  
`400 Bad Request` — Attempting to approve/reject a pass that is not in `pending` status.

---

#### `DELETE /api/passes/:id`

Permanently deletes a gate pass record. Restricted to `admin` only.

**URL Parameter:**

`id` — The MongoDB ObjectId of the gate pass.

**Success Response — `200 OK`:**

```json
{
  "message": "Gate pass deleted successfully"
}
```

---

### QR Verification

When a leave request is approved, a signed QR token is generated and embedded into the pass record. Security personnel use this endpoint to validate the token scanned at the gate.

---

#### `POST /api/verify`

Verifies a scanned QR code and returns the corresponding pass details. Accessible only by users with the `security` role.

**Request Body:**

```json
{
  "qrToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Success Response — `200 OK` (Pass is valid and active):**

```json
{
  "valid": true,
  "pass": {
    "_id": "64f4b3cd0e2d5d002f9c5623",
    "studentName": "Ananda Saikia",
    "rollNumber": "ET22BTHCS072",
    "department": "Computer Science and Engineering",
    "destination": "Guwahati",
    "exitDate": "2026-05-15T08:00:00.000Z",
    "returnDate": "2026-05-17T20:00:00.000Z",
    "status": "approved"
  }
}
```

**Error Responses:**

`200 OK` with `"valid": false` — Token is expired, already used, or has been tampered with.

```json
{
  "valid": false,
  "reason": "Pass has expired"
}
```

`403 Forbidden` — Caller does not have the `security` role.  
`400 Bad Request` — QR token is missing from the request body.

---

### Analytics

The analytics endpoint powers the admin and HOD dashboards, providing aggregated data for charts and summary statistics rendered using Recharts.

---

#### `GET /api/analytics`

Returns aggregated pass statistics. Accessible by `admin` and `hod` roles.

**Query Parameters (optional):**

| Parameter | Type | Description |
|---|---|---|
| `range` | `string` | Time range: `"7d"`, `"30d"`, `"90d"` (default: `"30d"`) |
| `department` | `string` | Filter by department name (HOD access is scoped to their own department automatically) |

**Example Request:**

```
GET /api/analytics?range=30d&department=Computer+Science
```

**Success Response — `200 OK`:**

```json
{
  "totalPasses": 148,
  "approved": 112,
  "rejected": 21,
  "pending": 15,
  "byPassType": {
    "local": 89,
    "outstation": 59
  },
  "byDepartment": [
    { "department": "Computer Science and Engineering", "count": 48 },
    { "department": "Electronic Engineering", "count": 35 }
  ],
  "trend": [
    { "date": "2026-04-15", "count": 5 },
    { "date": "2025-04-16", "count": 8 }
  ]
}
```

---

### Profile

---

#### `GET /api/profile`

Returns the authenticated user's profile information. Accessible by all roles.

**Success Response — `200 OK`:**

```json
{
  "_id": "64f3a2bc9e1d4c001e8b4512",
  "name": "Ananda Saikia",
  "email": "et22bthcs072@kazirangauniversity.in",
  "role": "student",
  "rollNumber": "ET22BTHCS072",
  "department": "Computer Science and Engineering",
  "year": 4,
  "createdAt": "2026-05-01T10:00:00.000Z"
}
```

---

#### `PUT /api/profile`

Updates the authenticated user's profile. Users can update their own name, contact information, and password. Role and email cannot be changed through this endpoint.

**Request Body:**

```json
{
  "name": "Ananda Saikia",
  "currentPassword": "securepassword@123456",
  "newPassword": "newpassword@123456"
}
```

> `currentPassword` and `newPassword` are only required when changing the password.

**Success Response — `200 OK`:**

```json
{
  "message": "Profile updated successfully"
}
```

**Error Responses:**

`400 Bad Request` — `currentPassword` is incorrect.  
`400 Bad Request` — `newPassword` does not meet minimum length requirements.

---

## Notes for Developers

**Database IDs** — All `_id` fields are MongoDB ObjectIds serialized as 24-character hexadecimal strings.

**Dates** — All date-time values are ISO 8601 strings in UTC (e.g., `2026-05-15T08:00:00.000Z`). Use Day.js on the client side to format them for local display.

**QR Token Lifetime** — QR tokens are tied to the pass's `returnDate`. A token becomes invalid automatically once the return date has passed, preventing misuse of old passes.

**Session Expiry** — NextAuth sessions expire after 30 days by default. The client SDK handles silent token refresh transparently.

**Pagination** — All list endpoints that return multiple records support `page` and `limit` query parameters. The default page size is `10`.

---

*This document reflects the API structure of GateVault as of the current `main` branch. Update this file whenever new endpoints are added or existing ones are modified.*
