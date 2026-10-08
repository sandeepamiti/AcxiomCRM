# AcxiomCRM — Enterprise Full-Stack Role-Based CRM

[![Node.js](https://img.shields.io/badge/Node.js-v18%2B-green.svg)](https://nodejs.org/)
[![React](https://img.shields.io/badge/React-v18-blue.svg)](https://react.dev/)
[![Express](https://img.shields.io/badge/Express-v4.21-lightgrey.svg)](https://expressjs.com/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-v14%2B-blue.svg)](https://www.postgresql.org/)
[![Prisma](https://img.shields.io/badge/Prisma-v5.22-darkblue.svg)](https://www.prisma.io/)
[![Bootstrap](https://img.shields.io/badge/Bootstrap-v5.3-purple.svg)](https://getbootstrap.com/)

**AcxiomCRM** is a complete, role-based Customer Relationship Management web application engineered for sales tracking, lead qualification, deal pipeline visualization, touchpoint scheduling, and compliance auditing.

---

## 🌟 Key Features

* **Strict Role-Based Access Control (RBAC)**:
  * **ADMIN**: Unrestricted system administration, staff account management, role allocation, password reset, and compliance audit trail inspection.
  * **MANAGER**: Department-wide visibility over customers, leads, opportunities, follow-up schedules, and team performance reports.
  * **SALES EXECUTIVE**: Scoped access focused on assigned leads, customer accounts, opportunities, and personal follow-ups.
* **Authentication & Security**:
  * JSON Web Tokens (JWT) stored in secure HttpOnly cookies / Bearer headers.
  * Adaptive password hashing with **bcryptjs** (10 salt rounds).
  * Enforced password complexity: 8+ characters, uppercase, lowercase, digit, and special character.
  * Automated account lockout protection after 5 consecutive failed login attempts (15-minute freeze).
  * Express Rate Limiting against brute-force attacks.
* **Pipeline Forecasting & Opportunities**:
  * Real-time calculation of **Weighted Pipeline** (`Amount × Probability / 100`).
  * Deal stages: `Qualification`, `Proposal`, `Negotiation`, `Won`, and `Lost`.
  * Date guard prevents active opportunities from having target close dates in the past.
* **Lead Qualification & Conversion Engine**:
  * Progression through `New`, `Contacted`, `Qualified`, `Unqualified`, `Converted`, and `Lost`.
  * Atomic transactional conversion turning **Qualified** leads into active Customer accounts and initial pipeline Opportunities.
* **Smart Follow-Up Scheduling**:
  * Schedule touchpoints across `Call`, `Meeting`, `Email`, and `Task` channels.
  * Date validation rejects scheduling follow-ups before the current date.
  * Filter tabs for `Pending`, `Upcoming`, and `Overdue` tasks with quick "Mark Complete" and "Cancel" actions.
* **Dynamic Executive Analytics & Chart.js**:
  * Dynamic KPIs: Total Customers, Total Leads, Open/Won/Lost Deals, Pending Follow-Ups, Total Pipeline, and Weighted Forecast.
  * Real database Chart.js charts: Lead Status Distribution (Doughnut), Pipeline Stages (Bar), and Monthly Closed Revenue (Line).
* **Multi-Tab Reports & CSV Export**:
  * Pipeline, Sales, Lead Conversion, and Customer reports with instant CSV downloads.
* **Compliance Audit Trail**:
  * Append-only security audit log recording every login, logout, create, update, delete, and conversion event.
  * Sanitizes payloads to ensure passwords, hashes, and tokens are never stored.

---

## 💻 Technology Stack

* **Frontend**: HTML5, CSS3, JavaScript, React.js (v18), Bootstrap 5, Bootstrap Icons, React Router DOM (v6), Axios, Chart.js, React-Chartjs-2.
* **Backend**: Node.js, Express.js, REST API, Helmet, CORS, Cookie-Parser, Express-Rate-Limit.
* **Database & ORM**: PostgreSQL, Prisma ORM.
* **Authentication**: JWT, bcryptjs, Role-Based Access Control.
* **Tools**: VS Code, Git, GitHub, Postman.

---

## 🏗️ Architecture & Layered Structure

```
AcxiomCRM/
│
├── client/                     # React Frontend
│   ├── src/
│   │   ├── api/                # Axios client with credentials & interceptors
│   │   ├── components/         # Common UI: StatCard, Pagination, Modals, Badges
│   │   ├── context/            # AuthContext (session state & role helpers)
│   │   ├── hooks/              # useAuth hook
│   │   ├── layouts/            # DashboardLayout (Sidebar & TopNavbar)
│   │   ├── pages/              # Auth, Dashboard, Customers, Leads, Deals, Reports, Admin
│   │   ├── routes/             # AppRoutes & ProtectedRoute (RBAC)
│   │   ├── services/           # Frontend API consumer modules
│   │   ├── App.jsx             # Root application
│   │   ├── index.css           # Design tokens, custom badges & scrollbars
│   │   └── main.jsx            # Entry point mounting Bootstrap & Router
│   ├── package.json
│   └── vite.config.js
│
├── server/                     # Node.js + Express REST API
│   ├── config/                 # db.js (Prisma client) & config.js (env options)
│   ├── controllers/            # Request/Response handlers
│   ├── middleware/             # authMiddleware, roleMiddleware, rateLimiter, errorHandler
│   ├── prisma/
│   │   ├── schema.prisma       # Prisma relational database schema
│   │   └── seed.js             # Realistic demo data seed script
│   ├── routes/                 # REST API endpoints
│   ├── scripts/
│   │   └── switch-db.js        # Helper to toggle between PostgreSQL and SQLite
│   ├── services/               # Core business logic & database queries
│   ├── utils/                  # AuditLogger, PasswordHelper, CsvExporter, ResponseHandler
│   ├── validators/             # Domain and schema validation functions
│   ├── app.js                  # Express assembly
│   ├── server.js               # Entry point listening on port 5000
│   └── package.json
│
├── API_DOCUMENTATION.md        # Comprehensive Postman-ready REST API specs
├── DATABASE_DESIGN.md          # ERD diagram, tables, keys, and constraint rules
├── PROJECT_EXPLANATION.md      # In-depth architectural & conceptual guide
├── INTERVIEW_QA.md             # Key interview questions & answers
├── .env.example                # Sample environment configuration
├── .gitignore
└── package.json                # Monorepo concurrency & convenience scripts
```

---

## 🚀 Prerequisites

1. **Node.js**: v18.0.0 or higher ([Download](https://nodejs.org/))
2. **npm**: v9.0.0 or higher (packaged with Node.js)
3. **PostgreSQL**: v14 or higher ([Download](https://www.postgresql.org/download/)) or a free cloud database like [Neon](https://neon.tech/) or [Supabase](https://supabase.com/).

---

## 🛠️ Step-by-Step Installation

### Step 1: Install Dependencies
Open a terminal in the project root directory and run:
```bash
npm run install:all
```
*(This automatically installs root, backend, and frontend packages).*

---

### Step 2: Database Configuration

#### Option A: Using PostgreSQL (Standard Production & Local Setup)
1. Ensure your PostgreSQL service is running.
2. Create a database named `acxiomcrm` in PostgreSQL:
   ```sql
   CREATE DATABASE acxiomcrm;
   ```
3. Update `server/.env` (or set `DATABASE_URL`):
   ```env
   DATABASE_URL="postgresql://postgres:your_password@localhost:5432/acxiomcrm?schema=public"
   ```
4. Push the schema and apply demo seed data:
   ```bash
   npm run prisma:generate
   npm run prisma:push
   npm run prisma:seed
   ```

#### Option B: Instant Zero-Setup Mode (SQLite)
If you are evaluating this project on a computer that does not currently have PostgreSQL installed, you can switch the Prisma engine with one single command:
```bash
npm run use:sqlite
npm run prisma:push
npm run prisma:seed
```
*(You can switch back to PostgreSQL at any time using `npm run use:postgres`)*.

---

### Step 3: Run the Full Application

Start both the backend server and React frontend concurrently:
```bash
npm run dev
```

* **Frontend Web Application**: [http://localhost:5173](http://localhost:5173)
* **Backend REST API**: [http://localhost:5000](http://localhost:5000)
* **API Health Check**: [http://localhost:5000/api/health](http://localhost:5000/api/health)

---

## 🔑 Demo Login Accounts

| Role | Email Address | Password | Permissions |
|---|---|---|---|
| **Admin** | `admin@acxiomcrm.com` | `Admin@12345` | Complete system access, user administration, role assignment, audit log inspection. |
| **Manager** | `manager@acxiomcrm.com` | `Manager@12345` | Department-wide customer, lead, and deal management; team analytics and reports. |
| **Sales Executive 1** | `sales1@acxiomcrm.com` | `Sales@12345` | Scoped to assigned leads, customers, deals, and personal schedule. |
| **Sales Executive 2** | `sales2@acxiomcrm.com` | `Sales@12345` | Scoped to assigned leads, customers, deals, and personal schedule. |

> 💡 **Quick Login:** The Login screen features 1-Click Demo Buttons that automatically populate these credentials for convenient demonstration.

---

## 🧪 Postman API Verification

Import the endpoints documented in [API_DOCUMENTATION.md](file:///c:/Users/RAKESH/OneDrive/Desktop/mini%20project/sundeep/API_DOCUMENTATION.md):

1. **Authentication**: `POST http://localhost:5000/api/auth/login`
2. **Customers**: `GET http://localhost:5000/api/customers`
3. **Leads**: `GET http://localhost:5000/api/leads`
4. **Lead Conversion**: `POST http://localhost:5000/api/leads/1/convert`
5. **Opportunities**: `GET http://localhost:5000/api/opportunities`
6. **Follow-Ups**: `GET http://localhost:5000/api/followups`
7. **Dashboard Metrics**: `GET http://localhost:5000/api/dashboard`
8. **Reports & CSV Export**: `GET http://localhost:5000/api/reports/pipeline?format=csv`

---

## 🐙 Git & GitHub Setup

To push this project to your GitHub profile:

```bash
git init
git add .
git commit -m "Initial AcxiomCRM enterprise full-stack release"
git branch -M main
git remote add origin https://github.com/your-username/acxiom-crm.git
git push -u origin main
```

*(Notice that `.env`, `node_modules`, and local build artifacts are safely ignored by `.gitignore`).*

---

## 🌐 Production Deployment

### 1. Database (PostgreSQL)
* Deploy a managed PostgreSQL database using [Neon](https://neon.tech), [Supabase](https://supabase.com), or [Render](https://render.com).
* Obtain the pooled connection string (e.g. `postgresql://user:pass@ep-host.region.aws.neon.tech/acxiomcrm?sslmode=require`).

### 2. Backend (Render / Railway / Heroku)
* Set environment variables in the host dashboard:
  * `DATABASE_URL`: Your production PostgreSQL URL
  * `JWT_SECRET`: A secure 64-character random string
  * `CLIENT_URL`: Your production frontend URL (e.g., `https://acxiomcrm.vercel.app`)
  * `NODE_ENV`: `production`
* Start command: `node server.js`

### 3. Frontend (Vercel / Netlify)
* Build command: `npm run build`
* Output directory: `dist`
* Set API proxy / rewrite rule or configure `VITE_API_BASE_URL` pointing to the deployed backend.
