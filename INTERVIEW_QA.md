# AcxiomCRM — Technical Interview Questions & Answers

This document prepares you for technical interviews, viva voce, and architectural defense sessions regarding the AcxiomCRM project.

---

### 1. Why did you choose React.js for the frontend?
**Answer:**
React provides a component-based architecture where UI elements (such as `StatCard`, `StatusBadge`, and `Pagination`) are encapsulated, reusable, and maintainable. Its Virtual DOM efficiently updates only the portions of the DOM that changed, ensuring snappy navigation across large datasets like customer tables and pipeline metrics without full-page reloads.

---

### 2. Why Node.js and Express.js for the backend?
**Answer:**
Node.js offers an asynchronous, non-blocking I/O event-loop architecture that excels at handling concurrent HTTP requests with minimal overhead. Express.js is a minimalist, flexible web framework that allows us to construct a clean layered REST API, organize modular routers, and apply reusable middleware for authentication, logging, and error handling.

---

### 3. Why PostgreSQL instead of a NoSQL database like MongoDB?
**Answer:**
CRM applications are inherently relational. A customer possesses multiple opportunities, an opportunity is tied to leads and activities, and audit logs link back to users. PostgreSQL enforces ACID compliance (Atomicity, Consistency, Isolation, Durability), strict foreign key constraints, and cascading integrity, ensuring critical sales pipeline data is never orphaned or corrupted.

---

### 4. Why Prisma ORM?
**Answer:**
Prisma provides a declarative schema modeling language (`schema.prisma`) and auto-generates a type-safe database client (`@prisma/client`). It eliminates SQL injection vulnerabilities through automated query parameterization, handles database migrations cleanly, and simplifies complex table joins (`include: { customer: true, assignedTo: true }`) into simple, readable JavaScript objects.

---

### 5. What is a REST API and what conventions did you follow?
**Answer:**
A REST (Representational State Transfer) API is an architectural style for networked applications using standard HTTP verbs:
* `GET` for fetching data (e.g., `GET /api/customers`)
* `POST` for creating new records (e.g., `POST /api/leads`)
* `PUT` for full record updates (e.g., `PUT /api/opportunities/:id`)
* `PATCH` for partial status changes (e.g., `PATCH /api/followups/:id/complete`)
* `DELETE` for removing records (e.g., `DELETE /api/customers/:id`)

All endpoints return predictable status codes (`200 OK`, `201 Created`, `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`) with a uniform JSON envelope (`{ success, message, data, errors }`).

---

### 6. What is Express middleware and how is it used in AcxiomCRM?
**Answer:**
Middleware functions are functions that have access to the request object (`req`), response object (`res`), and the `next` function in the application's request-response cycle. In AcxiomCRM, middleware is used for:
1. `authMiddleware`: Extracts the JWT token from the Authorization header or HttpOnly cookie, verifies its signature, checks user status, and attaches `req.user`.
2. `roleMiddleware`: Checks if `req.user.role` is permitted to access the route.
3. `rateLimiter`: Protects authentication endpoints against brute-force attacks.
4. `errorHandler`: Centrally catches and formats unexpected errors without exposing stack traces.

---

### 7. What is the difference between Authentication and Authorization?
**Answer:**
* **Authentication (AuthN)**: Verifying **who** you are. For example, logging in with an email and password to prove identity, resulting in a signed JWT token.
* **Authorization (AuthZ)**: Verifying **what** you have permission to do. For example, checking if the logged-in user with role `SALES_EXECUTIVE` is permitted to access `DELETE /api/customers/:id` or view the Admin User Management panel.

---

### 8. What is Role-Based Access Control (RBAC) and how did you implement it?
**Answer:**
RBAC restricts system access based on assigned user roles. We implemented three roles:
* `ADMIN`: Full access to user management, audit logs, and all CRM entities.
* `MANAGER`: Full access to CRM data, reports, and team pipeline, but restricted from user administration and compliance audit logs.
* `SALES_EXECUTIVE`: Scoped access only to assigned leads, customers, and opportunities.

On the backend, `authorizeRoles('ADMIN', 'MANAGER')` guards route endpoints. On the frontend, `ProtectedRoute allowedRoles={['ADMIN']}` guards client-side routes.

---

### 9. Why bcrypt/bcryptjs and how does password hashing work?
**Answer:**
Storing passwords in plain text is a severe security vulnerability. `bcrypt` uses an adaptive cryptographic hashing algorithm incorporating a salt (random data) and cost factor (work factor). Even if an attacker gains unauthorized access to the database, they cannot reverse the hashes or use precomputed rainbow tables to discover user passwords.

---

### 10. Why use JWT and how is it secured in AcxiomCRM?
**Answer:**
JSON Web Tokens (JWT) allow stateless authentication, eliminating the need to store session IDs in server memory. To maximize security:
1. The token is transmitted inside a secure **HttpOnly cookie** (or `Authorization: Bearer` header) so client-side JavaScript cannot be compromised by XSS attacks to steal the token.
2. Tokens have a strict expiration time (24 hours).
3. The server validates the token's cryptographic signature on every request using a secure environment secret (`JWT_SECRET`).

---

### 11. How does the Dashboard retrieve and render its data?
**Answer:**
The dashboard data is **not** hardcoded. When the Dashboard component mounts, it calls `GET /api/dashboard`. The backend `dashboardService` executes parallel database queries using Prisma (`prisma.customer.count()`, `prisma.opportunity.findMany()`, `prisma.lead.groupBy()`). It dynamically sums open deals to compute Total Pipeline, computes Weighted Pipeline, aggregates lead statuses, and groups won revenue by month. The frontend feeds these arrays directly into Chart.js (`<Doughnut />`, `<Bar />`, `<Line />`).

---

### 12. How does Lead Conversion work in AcxiomCRM?
**Answer:**
Lead conversion represents a prospect becoming a paying account:
1. **Pre-condition**: The lead must be in the `Qualified` status. If it is in any other status (`New`, `Contacted`, `Lost`), the server rejects conversion with a 400 Bad Request.
2. **Transaction**: Using a Prisma transaction (`prisma.$transaction`), the backend atomically:
   * Creates a new `Customer` record using the lead's contact information.
   * Creates an active `Opportunity` in the `Qualification` stage.
   * Updates the lead's status to `Converted`, linking the new `convertedCustomerId` and `convertedOpportunityId`.
3. An audit log is immediately written capturing the conversion event.

---

### 13. How is Weighted Pipeline calculated and why is it important?
**Answer:**
**Formula:**
$$\text{Weighted Pipeline} = \text{Amount} \times \left(\frac{\text{Probability}}{100}\right)$$

*Example*: If a deal is worth $100,000 but is currently in the Proposal stage with a 40% win probability, its weighted pipeline forecast is $40,000.
This metric prevents executive sales leaders from overestimating revenue by accounting for the realistic risk profile of active negotiations.

---

### 14. How is the Audit Logging system designed?
**Answer:**
Audit logging tracks system mutations and security events. The `AuditLog` table records:
* `userId`, `userEmail`, `action`, `entityName`, `recordId`, `oldValue`, `newValue`, `ipAddress`, and `createdAt`.
* **Append-only**: There are no update or delete routes for audit logs.
* **Sensitive Data Sanitation**: A utility function (`sanitizePayload`) scrubs fields like `password`, `token`, and `secret` before saving to ensure credentials never leak into compliance records.

---

### 15. How did you handle Account Lockout?
**Answer:**
To prevent credential stuffing and brute-force attacks:
* The `User` model stores `failedLoginAttempts` and `lockoutUntil`.
* Each consecutive failed password check increments `failedLoginAttempts`.
* When the counter reaches 5, `lockoutUntil` is set to 15 minutes in the future.
* Subsequent attempts return `403 Forbidden` until the lockout expires or an Administrator resets the user's password.
* A successful login resets `failedLoginAttempts` to 0.

---

### 16. What would you improve if given additional development time?
**Answer:**
1. Automated email notifications via SendGrid/Nodemailer for follow-up reminders.
2. Drag-and-drop Kanban view for visual opportunity stage transitions.
3. Two-factor authentication (2FA/TOTP).
4. Automated background cron jobs checking for overdue follow-ups daily.
