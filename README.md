# Smart Leads Dashboard

A full-stack Lead Management Dashboard built with the MERN stack (MongoDB, Express, React, Node.js) using TypeScript throughout.

---

## Tech Stack

**Frontend:** React 18, TypeScript, TailwindCSS, React Router v6, Axios, React Hot Toast  
**Backend:** Node.js, Express.js, TypeScript, MongoDB + Mongoose, JWT, bcryptjs  
**DevOps:** Docker, Docker Compose, Nginx

---

## Features

- **JWT Authentication** – Register, login, protected routes, bcrypt password hashing
- **Role-Based Access Control** – `admin` and `sales` roles with different permissions
- **Leads CRUD** – Create, read, update, delete leads (delete restricted to admin)
- **Advanced Filtering** – Filter by status, source, search by name/email, sort asc/desc — all combinable
- **Debounced Search** – 400ms debounce on search input
- **Backend Pagination** – 10 leads per page with full metadata
- **CSV Export** – Export filtered leads to CSV
- **Analytics Page** – Visual breakdown of leads by status and source
- **Dark Mode** – Toggle with system preference detection
- **Responsive Design** – Mobile-first, works on all screen sizes
- **Loading & Empty States** – Spinner, empty state messages, error handling UI
- **Form Validation** – Client-side and server-side validation

---

## Project Structure

```
smart-leads/
├── backend/
│   ├── src/
│   │   ├── config/         # DB connection
│   │   ├── controllers/    # Route handlers (auth, leads)
│   │   ├── middleware/      # Auth, error handler, validation
│   │   ├── models/         # Mongoose models (User, Lead)
│   │   ├── routes/         # Express routers
│   │   ├── types/          # TypeScript interfaces
│   │   ├── utils/          # JWT helpers, response helpers
│   │   └── index.ts        # Entry point
│   ├── Dockerfile
│   ├── package.json
│   └── tsconfig.json
│
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── auth/       # ProtectedRoute
│   │   │   ├── layout/     # Navbar, Layout
│   │   │   ├── leads/      # LeadForm, LeadTable, LeadFiltersBar, LeadDetail
│   │   │   └── ui/         # Badge, Modal, Spinner, Pagination, ConfirmDialog
│   │   ├── context/        # AuthContext, ThemeContext
│   │   ├── hooks/          # useDebounce, useLeads
│   │   ├── pages/          # LoginPage, RegisterPage, DashboardPage, LeadsPage, StatsPage, UsersPage
│   │   ├── services/       # api.ts, authService.ts, leadsService.ts
│   │   ├── types/          # TypeScript types
│   │   ├── App.tsx
│   │   └── main.tsx
│   ├── Dockerfile
│   ├── nginx.conf
│   ├── package.json
│   └── vite.config.ts
│
├── docker-compose.yml
└── README.md
```

---

## Setup Instructions

### Prerequisites

- Node.js 18+
- MongoDB (local) or MongoDB Atlas URI
- Docker & Docker Compose (optional)

---

### Option 1 — Local Development (without Docker)

#### 1. Clone & install

```bash
git clone <your-repo-url>
cd smart-leads
```

#### 2. Backend setup

```bash
cd backend
cp .env.example .env
# Edit .env with your values
npm install
npm run dev
```

#### 3. Frontend setup

```bash
cd frontend
cp .env.example .env
npm install
npm run dev
```

Frontend runs at `http://localhost:5173`, Backend at `http://localhost:5000`.

---

### Option 2 — Docker Compose

```bash
# From project root
cp .env.example .env
# Edit .env — set JWT_SECRET

docker-compose up --build
```

App will be available at `http://localhost`.

---

## Environment Variables

### Backend (`backend/.env`)

| Variable       | Description                        | Example                                      |
|----------------|------------------------------------|----------------------------------------------|
| `PORT`         | Server port                        | `5000`                                       |
| `MONGODB_URI`  | MongoDB connection string          | `mongodb://localhost:27017/smart-leads`      |
| `JWT_SECRET`   | Secret key for JWT signing         | `supersecretkey`                             |
| `JWT_EXPIRES_IN` | JWT expiry                       | `7d`                                         |
| `NODE_ENV`     | Environment                        | `development` or `production`                |
| `CLIENT_URL`   | Frontend URL (for CORS)            | `http://localhost:5173`                      |

### Frontend (`frontend/.env`)

| Variable       | Description                        | Example  |
|----------------|------------------------------------|----------|
| `VITE_API_URL` | API base URL                       | `/api`   |

---

## API Documentation

Base URL: `http://localhost:5000/api`

### Auth Endpoints

| Method | Endpoint          | Auth | Description            |
|--------|-------------------|------|------------------------|
| POST   | `/auth/register`  | ❌   | Register new user      |
| POST   | `/auth/login`     | ❌   | Login user             |
| GET    | `/auth/me`        | ✅   | Get current user       |
| GET    | `/auth/users`     | ✅ Admin | List all users     |

#### POST `/auth/register`
```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "password123",
  "role": "sales"
}
```

#### POST `/auth/login`
```json
{
  "email": "john@example.com",
  "password": "password123"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Login successful",
  "data": {
    "user": { "_id": "...", "name": "John Doe", "email": "...", "role": "sales" },
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

---

### Leads Endpoints

All leads endpoints require `Authorization: Bearer <token>` header.

| Method | Endpoint              | Role        | Description              |
|--------|-----------------------|-------------|--------------------------|
| GET    | `/leads`              | All         | Get leads (with filters) |
| GET    | `/leads/:id`          | All         | Get single lead          |
| POST   | `/leads`              | All         | Create lead              |
| PUT    | `/leads/:id`          | All         | Update lead              |
| DELETE | `/leads/:id`          | Admin only  | Delete lead              |
| GET    | `/leads/export`       | All         | Export CSV               |
| GET    | `/leads/stats`        | All         | Get analytics stats      |

#### GET `/leads` — Query Parameters

| Param    | Type   | Values                              | Description       |
|----------|--------|-------------------------------------|-------------------|
| `status` | string | `New`, `Contacted`, `Qualified`, `Lost` | Filter by status |
| `source` | string | `Website`, `Instagram`, `Referral`  | Filter by source  |
| `search` | string | any                                 | Search name/email |
| `sort`   | string | `latest`, `oldest`                  | Sort order        |
| `page`   | number | default `1`                         | Page number       |
| `limit`  | number | default `10`, max `100`             | Records per page  |

**Response:**
```json
{
  "success": true,
  "data": { "leads": [...] },
  "meta": {
    "total": 45,
    "page": 1,
    "limit": 10,
    "totalPages": 5,
    "hasNextPage": true,
    "hasPrevPage": false
  }
}
```

#### POST `/leads`
```json
{
  "name": "Jane Smith",
  "email": "jane@example.com",
  "status": "New",
  "source": "Instagram",
  "notes": "Interested in premium plan"
}
```

---

## Role Permissions

| Action              | Admin | Sales |
|---------------------|-------|-------|
| View own leads      | ✅    | ✅    |
| View all leads      | ✅    | ❌    |
| Create lead         | ✅    | ✅    |
| Update lead         | ✅    | ✅ (own) |
| Delete lead         | ✅    | ❌    |
| Export CSV          | ✅    | ✅    |
| View users list     | ✅    | ❌    |

---

## Git Commit Convention

This project follows conventional commits:

```
feat: add CSV export functionality
fix: resolve pagination off-by-one error
refactor: extract lead filters to custom hook
chore: update dependencies
```

---

## Evaluation Criteria Coverage

| Criterion           | Implementation                                              |
|---------------------|-------------------------------------------------------------|
| TypeScript          | Strict mode, interfaces everywhere, minimal `any`           |
| Code Quality        | Clean separation of concerns, single responsibility        |
| Project Structure   | Feature-based, scalable folder layout                       |
| API Design          | RESTful, consistent response format, proper status codes    |
| UI/UX               | Responsive, loading/empty states, dark mode, animations     |
| Error Handling      | Global error middleware, client-side error boundaries       |
| Reusability         | Shared components, custom hooks, service layer              |
| Scalability         | Mongoose indexes, pagination, lazy-loaded routes            |
| Debounced Search    | 400ms debounce via custom hook                             |
| CSV Export          | Filtered export with auth headers                          |
| RBAC                | Middleware-enforced admin/sales roles                      |
| Docker              | Multi-stage builds, docker-compose with Nginx              |
| Dark Mode           | Tailwind `dark:` classes, localStorage persistence         |
