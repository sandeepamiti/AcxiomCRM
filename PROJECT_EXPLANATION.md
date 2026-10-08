# AcxiomCRM — Comprehensive Project Explanation

Welcome to the architectural and conceptual walkthrough for **AcxiomCRM**, an enterprise-grade Customer Relationship Management system built using modern web standards. This guide explains every facet of the system in clean, approachable language suitable for academic viva, project demonstrations, and engineering interviews.

---

## 1. Project Objective

The primary objective of AcxiomCRM is to streamline the end-to-end sales and customer lifecycle for enterprise organizations. It replaces unstructured spreadsheets and disjointed communication channels with a centralized, role-based platform that enforces business validation, maintains a clear sales pipeline, schedules follow-ups, and records an immutable security audit trail.

---

## 2. Problem Statement

Organizations without a formal CRM face critical operational bottlenecks:
1. **Lead Leakage**: Inbound leads from websites, referrals, and ads get lost or neglected without tracking.
2. **Lack of Pipeline Visibility**: Sales leaders cannot accurately forecast expected revenue or see which stage deals are stalling in.
3. **Accountability Gaps**: Without audit logs, changes to sensitive records (like customer contacts or deal amounts) cannot be traced back to who made them.
4. **Security & Access Control Vulnerabilities**: Junior representatives often have excessive access to company-wide financial data, or managers lack team oversight.

---

## 3. The AcxiomCRM Solution

AcxiomCRM resolves these challenges through:
* **Role-Based Access Control (RBAC)**: Enforces three distinct permission levels (`ADMIN`, `MANAGER`, `SALES_EXECUTIVE`).
* **Structured Pipeline Workflows**: Tracks deals through qualification, proposal, negotiation, won, and lost phases, computing weighted pipeline forecasts (`Amount × Probability / 100`).
* **Lead-to-Deal Conversion Engine**: A transactional converter turning qualified leads into permanent customer accounts and opportunities.
* **Proactive Follow-Up Scheduler**: Filters for pending, upcoming, and overdue tasks to guarantee timely customer touchpoints.
* **Append-Only Audit Logging**: Automatically tracks logins, logouts, creation, modification, and deletion events with IP addresses.

---

## 4. Technology Stack & Rationale

| Layer | Technology | Why Chosen? |
|---|---|---|
| **Frontend** | React.js (v18) | Component-based reactivity, virtual DOM for performance, and clean modular UI state. |
| **Styling** | Bootstrap 5 + CSS3 | Accessible, responsive grid system, battle-tested utility classes, enhanced with custom design tokens. |
| **Routing** | React Router DOM (v6) | Declarative client-side routing with role-guarded `ProtectedRoute` wrappers. |
| **Data Fetching** | Axios | Interceptors for seamless JWT attachment and automated 401 unauthenticated redirect handling. |
| **Visualizations** | Chart.js & React-Chartjs-2 | High-performance HTML5 canvas charting for pipeline stages, lead statuses, and sales trends. |
| **Backend** | Node.js & Express.js | Asynchronous event-driven I/O model ideal for RESTful microservices and JSON APIs. |
| **ORM** | Prisma ORM | Type-safe database queries, automated migrations, declarative schema, and relationship safety. |
| **Database** | PostgreSQL | Enterprise ACID compliance, relational integrity, robust indexing, and transaction guarantees. |
| **Security** | JWT + bcryptjs + Helmet | Industry-standard password hashing, stateless tokens, HTTP security headers, and rate limiting. |

---

## 5. Architectural Layers

The system follows a clean **layered architecture** ensuring separation of concerns:

```
[ React Client ]
       │  (Axios HTTP / REST JSON)
       ▼
[ Express Router ] ──────> [ Middleware (Auth, RBAC, RateLimit, Validator) ]
       │
       ▼
[ Controllers ] (Extracts params, cookies, and formats standard HTTP responses)
       │
       ▼
[ Services ] (Pure Business Logic: pipeline calculations, qualification checks, audit logging)
       │
       ▼
[ Prisma ORM Client ] (Constructs parameterized SQL queries)
       │
       ▼
[ PostgreSQL Database ] (ACID Transactions & Relational Tables)
```

---

## 6. Authentication & Security Implementation

1. **Password Encryption**: Passwords are never stored in plain text. They are hashed using **bcryptjs** with 10 salt rounds.
2. **Password Complexity**: Enforces at least 8 characters, an uppercase letter, a lowercase letter, a number, and a special character.
3. **Stateless JWT**: JSON Web Tokens signed with a secret key expire after 24 hours. The token is transmitted via an **HttpOnly cookie** or `Authorization: Bearer <token>` header, mitigating Cross-Site Scripting (XSS) risks.
4. **Account Lockout Mechanism**: After 5 consecutive failed login attempts, the account is temporarily locked for 15 minutes, neutralizing brute-force dictionary attacks.
5. **Rate Limiting**: Express Rate Limit blocks abusive IP traffic on authentication endpoints.

---

## 7. Role-Based Access Control (RBAC)

* **ADMIN**:
  * Unrestricted access across the application.
  * Can manage user accounts, assign roles, activate/deactivate staff, and reset passwords.
  * Can inspect system-wide audit logs.
* **MANAGER**:
  * Manages team CRM records (Customers, Leads, Opportunities, Follow-Ups, Activities).
  * Can view team dashboard analytics and performance reports.
  * Restricted from user administration and compliance audit logs.
* **SALES EXECUTIVE**:
  * Scoped to assigned records (or records created by them).
  * Can qualify leads, create opportunities, log activities, and schedule follow-ups.
  * Cannot access admin panels or company-wide managerial configurations.

---

## 8. Core CRM Modules

### 8.1 Customer Management
* **Entity**: Stores company name, primary contact, telephone, email, and address.
* **Code Generation**: Generates sequential identifiers (e.g., `CUST-1001`).
* **Relationships**: Linked to multiple opportunities, follow-ups, and engagement activities.

### 8.2 Lead Management & Conversion
* **Lifecycle**: `New` ➔ `Contacted` ➔ `Qualified` ➔ `Converted` (or `Unqualified` / `Lost`).
* **Conversion Guard**: Leads must reach the `Qualified` status before conversion is permitted.
* **Atomic Transaction**: Conversion automatically provisions a new `Customer` record, creates a corresponding `Opportunity`, marks the lead as `Converted`, and timestamps the conversion.

### 8.3 Opportunity & Pipeline Management
* **Stages**: `Qualification` ➔ `Proposal` ➔ `Negotiation` ➔ `Won` / `Lost`.
* **Rules**: Deal amount must exceed 0, win probability is bound between 0% and 100%, and active opportunities cannot have target close dates in the past.
* **Weighted Value Formula**: `Weighted Pipeline = Amount × (Probability / 100)`.

### 8.4 Follow-Up Scheduler
* **Touchpoints**: Scheduled via `Call`, `Meeting`, `Email`, or `Task`.
* **Date Guard**: Disallows scheduling a new follow-up with a date prior to today.
* **Status Views**: Dedicated views for pending, upcoming, and overdue actions.

### 8.5 Activity Logging
* History of customer interactions, demos, and correspondence.

### 8.6 Executive Dashboard & Reports
* Dynamic KPIs (Total Customers, Leads, Open/Won/Lost Deals, Pipeline values).
* Chart.js visualizations driven by live database queries.
* Multi-tab reporting with CSV export capabilities.

---

## 9. Audit Logging System

Every security-sensitive or mutating CRM operation automatically generates an immutable record in `AuditLog`:
* Logs `userId`, `userEmail`, `action`, `entityName`, `recordId`, `oldValue`, `newValue`, and `ipAddress`.
* Sanitizes payloads to guarantee passwords and tokens are **never** logged.

---

## 10. Future Enhancements

* **Email Integration**: Automated transactional email notifications via SMTP or SendGrid.
* **Kanban Board**: Drag-and-drop pipeline stage transitions for opportunities.
* **Two-Factor Authentication (2FA)**: Time-based One-Time Password (TOTP) support.
* **Cloud Storage**: Attachment uploads (contracts, proposals) directly to AWS S3.
