# AcxiomCRM REST API Documentation

AcxiomCRM exposes a clean, predictable, and secure RESTful API. All API responses follow a standardized JSON envelope structure with appropriate HTTP status codes.

---

## 1. API Architecture & Standards

### Base URL
```
http://localhost:5000/api
```

### Standard Response Envelopes

#### Success Response
```json
{
  "success": true,
  "message": "Operation successful description",
  "data": { ... },
  "meta": {
    "page": 1,
    "limit": 10,
    "total": 45,
    "totalPages": 5
  }
}
```

#### Error Response
```json
{
  "success": false,
  "message": "User-friendly description of the error",
  "errors": ["Specific validation or domain violation detail"]
}
```

### HTTP Status Codes
| Code | Name | Description |
|---|---|---|
| `200` | OK | Request succeeded, returns requested resource or status. |
| `201` | Created | New record successfully created. |
| `400` | Bad Request | Validation failure, invalid business rule, or malformed payload. |
| `401` | Unauthorized | Missing, invalid, or expired JWT token. |
| `403` | Forbidden | Authenticated, but role does not have authorization, or account is locked. |
| `404` | Not Found | Requested entity was not found in the database. |
| `409` | Conflict | Unique constraint violation (e.g., duplicate email address). |
| `500` | Internal Server Error | Unexpected server runtime exception. |

### Authentication Header
For protected endpoints, supply the JWT token in the `Authorization` header or through the secure HttpOnly cookie `acxiom_token`:
```http
Authorization: Bearer <YOUR_JWT_TOKEN>
```

---

## 2. Authentication Endpoints

### 2.1 User Login
* **Endpoint:** `POST /auth/login`
* **Authentication:** None (Public)
* **Role Requirement:** None
* **Rate Limit:** 20 attempts per 15 minutes per IP
* **Account Lockout:** 5 consecutive failures triggers a 15-minute lockout.

#### Request Body
```json
{
  "email": "admin@acxiomcrm.com",
  "password": "Admin@12345"
}
```

#### Success Response (`200 OK`)
```json
{
  "success": true,
  "message": "Login successful",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "user": {
      "id": 1,
      "name": "System Administrator",
      "email": "admin@acxiomcrm.com",
      "role": "ADMIN",
      "roleId": 1,
      "status": "Active"
    }
  }
}
```

#### Error Responses
* `401 Unauthorized`: `"Invalid email or password."`
* `403 Forbidden`: `"Account is temporarily locked due to multiple failed attempts. Try again in 15 minute(s)."`

---

### 2.2 User Registration
* **Endpoint:** `POST /auth/register`
* **Authentication:** None (Public)
* **Role Requirement:** Assigns `SALES_EXECUTIVE` by default (Admin and Manager roles cannot be selected publicly).

#### Request Body
```json
{
  "name": "Alexander Pierce",
  "email": "alex.pierce@acxiomcrm.com",
  "password": "SecurePassword@2026"
}
```

#### Success Response (`201 Created`)
```json
{
  "success": true,
  "message": "Registration successful. Welcome to AcxiomCRM!",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "user": {
      "id": 5,
      "name": "Alexander Pierce",
      "email": "alex.pierce@acxiomcrm.com",
      "role": "SALES_EXECUTIVE",
      "roleId": 3,
      "status": "Active"
    }
  }
}
```

---

### 2.3 Current User Profile
* **Endpoint:** `GET /auth/me`
* **Authentication:** Required (`Bearer <token>`)
* **Role Requirement:** Any authenticated role

#### Success Response (`200 OK`)
```json
{
  "success": true,
  "message": "User profile retrieved",
  "data": {
    "user": {
      "id": 1,
      "name": "System Administrator",
      "email": "admin@acxiomcrm.com",
      "role": "ADMIN",
      "roleId": 1,
      "status": "Active",
      "createdAt": "2026-10-08T15:00:00.000Z"
    }
  }
}
```

---

### 2.4 User Logout
* **Endpoint:** `POST /auth/logout`
* **Authentication:** Required
* **Action:** Logs `LOGOUT` in the audit log and clears the `acxiom_token` HttpOnly cookie.

#### Success Response (`200 OK`)
```json
{
  "success": true,
  "message": "Logged out successfully",
  "data": null
}
```

---

## 3. Customer Management Endpoints

### 3.1 Get All Customers
* **Endpoint:** `GET /customers`
* **Authentication:** Required
* **Role Scoping:**
  * `ADMIN` & `MANAGER`: All company customers.
  * `SALES_EXECUTIVE`: Only customers created by or assigned to the user.
* **Query Parameters:**
  * `page` (default: 1)
  * `limit` (default: 10)
  * `search` (e.g., `acme`)
  * `status` (`Active` or `Inactive`)

#### Success Response (`200 OK`)
```json
{
  "success": true,
  "message": "Customers retrieved successfully",
  "data": [
    {
      "id": 1,
      "customerCode": "CUST-1001",
      "customerName": "Acme Corporation",
      "email": "contact@acmecorp.com",
      "phone": "+1-555-0199",
      "companyName": "Acme Corporation Inc.",
      "city": "Chicago",
      "state": "IL",
      "status": "Active",
      "createdAt": "2026-10-08T15:00:00.000Z",
      "_count": {
        "opportunities": 2,
        "followUps": 1,
        "activities": 1
      }
    }
  ],
  "meta": {
    "page": 1,
    "limit": 10,
    "total": 1,
    "totalPages": 1
  }
}
```

---

### 3.2 Create Customer
* **Endpoint:** `POST /customers`
* **Authentication:** Required
* **Audit Action:** `CREATE`

#### Request Body
```json
{
  "customerName": "Zenith Global Tech",
  "email": "procurement@zenithglobal.com",
  "phone": "+1-555-8822",
  "companyName": "Zenith Global Enterprises LLC",
  "address": "400 Tech Plaza, Suite 900",
  "city": "Seattle",
  "state": "WA",
  "status": "Active"
}
```

#### Success Response (`201 Created`)
```json
{
  "success": true,
  "message": "Customer created successfully",
  "data": {
    "id": 6,
    "customerCode": "CUST-1006",
    "customerName": "Zenith Global Tech",
    "email": "procurement@zenithglobal.com",
    "phone": "+1-555-8822",
    "companyName": "Zenith Global Enterprises LLC",
    "status": "Active",
    "createdAt": "2026-10-08T15:30:00.000Z"
  }
}
```

---

### 3.3 Update Customer
* **Endpoint:** `PUT /customers/:id`
* **Authentication:** Required
* **Audit Action:** `UPDATE`

#### Request Body
```json
{
  "customerName": "Zenith Global Tech Inc.",
  "phone": "+1-555-8823",
  "status": "Active"
}
```

#### Success Response (`200 OK`)
```json
{
  "success": true,
  "message": "Customer updated successfully",
  "data": {
    "id": 6,
    "customerName": "Zenith Global Tech Inc.",
    "phone": "+1-555-8823"
  }
}
```

---

### 3.4 Delete Customer
* **Endpoint:** `DELETE /customers/:id`
* **Authentication:** Required
* **Role Requirement:** `ADMIN` or `MANAGER` (Sales Executives receive `403 Forbidden`)
* **Audit Action:** `DELETE`

#### Success Response (`200 OK`)
```json
{
  "success": true,
  "message": "Customer deleted successfully",
  "data": null
}
```

---

## 4. Lead Management Endpoints

### 4.1 Get All Leads
* **Endpoint:** `GET /leads`
* **Query Parameters:** `page`, `limit`, `search`, `status` (`New`, `Contacted`, `Qualified`, `Unqualified`, `Converted`, `Lost`), `source` (`Website`, `Referral`, `Cold Call`, `Advertisement`, `Partner`, `Other`)

---

### 4.2 Create Lead
* **Endpoint:** `POST /leads`
* **Request Body:**
```json
{
  "leadName": "Robert Vance",
  "email": "robert@vancerefrigeration.com",
  "phone": "+1-555-9011",
  "companyName": "Vance Refrigeration",
  "source": "Referral",
  "status": "New",
  "expectedValue": 45000
}
```
* **Success Response (`201 Created`)**

---

### 4.3 Convert Lead
* **Endpoint:** `POST /leads/:id/convert`
* **Business Rule:** Lead MUST have status `Qualified`. Rejects if lead is not qualified or already converted.
* **Action:** Atomically creates Customer, creates Opportunity, sets Lead status to `Converted`, and records an audit log entry.

#### Request Body
```json
{
  "opportunityName": "Vance Refrigeration - Commercial Fitout",
  "amount": 45000,
  "expectedCloseDate": "2026-11-15"
}
```

#### Success Response (`200 OK`)
```json
{
  "success": true,
  "message": "Lead successfully converted to Customer and Opportunity!",
  "data": {
    "customer": {
      "id": 7,
      "customerCode": "CUST-1007",
      "customerName": "Robert Vance",
      "companyName": "Vance Refrigeration"
    },
    "opportunity": {
      "id": 6,
      "opportunityName": "Vance Refrigeration - Commercial Fitout",
      "amount": 45000,
      "stage": "Qualification",
      "probability": 20
    },
    "lead": {
      "id": 3,
      "status": "Converted",
      "convertedAt": "2026-10-08T15:35:00.000Z"
    }
  }
}
```

---

## 5. Opportunity Management Endpoints

### 5.1 Get All Opportunities
* **Endpoint:** `GET /opportunities`
* **Query Parameters:** `page`, `limit`, `search`, `stage` (`Qualification`, `Proposal`, `Negotiation`, `Won`, `Lost`), `status` (`Open`, `Won`, `Lost`)
* **Computed Field:** `weightedAmount = amount * (probability / 100)`

---

### 5.2 Create Opportunity
* **Endpoint:** `POST /opportunities`
* **Business Rules:**
  * `amount > 0`
  * `0 <= probability <= 100`
  * Active opportunities cannot have `expectedCloseDate` in the past.

#### Request Body
```json
{
  "opportunityName": "Enterprise Data Lake Implementation",
  "customerId": 1,
  "amount": 75000,
  "stage": "Proposal",
  "probability": 60,
  "expectedCloseDate": "2026-11-30"
}
```

---

## 6. Follow-Up Endpoints

### 6.1 Schedule Follow-Up
* **Endpoint:** `POST /followups`
* **Business Rule:** Scheduled date CANNOT be before today!
* **Request Body:**
```json
{
  "customerId": 1,
  "followUpDate": "2026-10-12T10:00:00.000Z",
  "followUpType": "Meeting",
  "remarks": "Product demonstration with department heads",
  "status": "Planned"
}
```

### 6.2 Complete / Cancel Follow-Up
* **Endpoint:** `PATCH /followups/:id/complete` or `PATCH /followups/:id/cancel`

---

## 7. Activity Management Endpoints

* `GET /activities`: List activities with filtering.
* `POST /activities`: Log new activity (`Call`, `Meeting`, `Email`, `Task`).
* `PUT /activities/:id`: Update activity.
* `DELETE /activities/:id`: Remove activity.

---

## 8. User Management (Admin Only)

* `GET /users`: List users with status & lockout state.
* `GET /users/roles`: List all system roles (`ADMIN`, `MANAGER`, `SALES_EXECUTIVE`).
* `POST /users`: Create staff user with password validation.
* `PUT /users/:id`: Edit user details, activate/deactivate account.
* `POST /users/:id/reset-password`: Reset user password and clear lockout.

---

## 9. Audit Logs (Admin Only)

* `GET /audit-logs`: Search append-only audit records by action, entity name, user, or date range.

---

## 10. Dashboard & Analytics

* `GET /dashboard`: Dynamic role-tailored KPIs, Chart.js datasets (Lead distribution, Pipeline stages, Monthly revenue), and recent activity feed.

---

## 11. Reports & CSV Export

* `GET /reports/pipeline` (Append `?format=csv` for downloadable CSV file)
* `GET /reports/sales` (Append `?format=csv`)
* `GET /reports/conversion` (Append `?format=csv`)
* `GET /reports/customers` (Append `?format=csv`)
