# 📚 University Document Management System (UDMS)

> A Web-based Predictive Analytics University Document Management System for Academic Accreditation Using Deep Learning — developed for the University of the Immaculate Conception (UDM).

---

## Table of Contents

- [Overview](#overview)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [User Roles & Permissions](#user-roles--permissions)
- [Permission Helpers (Frontend)](#permission-helpers-frontend)
- [Route Protection](#route-protection)
- [Backend API Protection](#backend-api-protection)
- [PermissionGate Component](#permissiongate-component)
- [Permission Cheat Sheet](#permission-cheat-sheet)
- [Docker Setup](#docker-setup)

---

## Overview

UDMS is a full-stack web application designed to help universities manage documents for academic accreditation. It uses deep learning for predictive analytics to support accreditation workflows, with a role-based access control system to manage what each user can see and do.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React + Vite |
| Backend | Python (Flask) |
| Auth | JWT (JSON Web Tokens) |
| Cache | Redis |
| ML/AI | Jupyter Notebook / Python (Deep Learning) |
| Containerization | Docker / Docker Compose |
| Deployment | Railway |

**Language breakdown:** JavaScript (54.8%), Python (22.4%), Jupyter Notebook (21.4%), CSS (1.0%), HTML (0.3%), Dockerfile (0.1%)

---

## Project Structure

```
Capstone-Project/
├── frontend/                  # React + Vite frontend
│   └── src/
│       ├── components/
│       │   └── PermissionGate.jsx     # Reusable permission wrapper
│       ├── pages/
│       │   └── Users.jsx              # User management page
│       └── utils/
│           └── auth_utils.jsx         # Auth & permission helpers
├── backend/                   # Flask backend
│   └── .env                   # Environment variables
├── uploads/                   # Uploaded document storage
├── docker-compose.yml
├── railway.json
├── QUICK_START.md
├── USER_MANAGEMENT_GUIDE.md
├── PERMISSION_SUMMARY.md
├── PERMISSION_BASED_FETCHING_GUIDE.md
├── ROUTE_PROTECTION_GUIDE.md
├── ROUTE_PROTECTION_SUMMARY.md
├── USAGE_EXAMPLES.md
└── IMPLEMENTATION_CHECKLIST.md
```

---

## Getting Started

### Prerequisites

- [Node.js](https://nodejs.org/) (v18+)
- [Python](https://www.python.org/) (3.10+)
- [Docker](https://www.docker.com/) & Docker Compose
- [Redis](https://redis.io/)

### Run with Docker (Recommended)

```bash
# Clone the repository
git clone https://github.com/am-not-a-coder/Capstone-Project.git
cd Capstone-Project

# Start all services
docker compose up
```

| Service | URL |
|---------|-----|
| Frontend | http://localhost:5173 |
| Backend API | http://localhost:5000 |
| Redis | localhost:6379 |

### Run Manually

**Backend:**
```bash
cd backend
pip install -r requirements.txt
flask run
```

**Frontend:**
```bash
cd frontend
npm install
npm run dev
```

---

## User Roles & Permissions

The system uses a three-tier permission hierarchy.

### Permission Fields (from `models.py`)

```python
isAdmin           = Boolean  # Full system administrator
isCoAdmin         = Boolean  # Limited administrator
isRating          = Boolean  # Can rate/evaluate documents
isEdit            = Boolean  # Can edit users
crudFormsEnable   = Boolean  # Can create/update/delete forms
crudProgramEnable = Boolean  # Can create/update/delete programs
crudInstituteEnable = Boolean # Can create/update/delete institutes
```

### Role Hierarchy

```
Admin (isAdmin: true)
├── Full access to everything
├── Can create/edit/delete all users
├── Can assign Co-Admin status
└── Can assign all permissions to others

Co-Admin (isCoAdmin: true)
├── Access to user management
├── Can edit users (if isEdit is true)
├── Can rate documents (if isRating is true)
├── Can manage forms (if crudFormsEnable is true)
├── Can manage programs (if crudProgramEnable is true)
├── Can manage institutes (if crudInstituteEnable is true)
└── Cannot create other admins or co-admins

Regular User
├── isRating          → Can rate/evaluate documents
├── isEdit            → Can edit user information
├── crudFormsEnable   → Can manage forms
├── crudProgramEnable → Can manage programs
└── crudInstituteEnable → Can manage institutes
```

---

## Permission Helpers (Frontend)

Located in `frontend/src/utils/auth_utils.jsx`.

### Available Functions

```js
import {
  isLoggedIn,          // Returns true if user is authenticated
  getCurrentUser,      // Returns the full user object
  fetchCurrentUser,    // Fetches latest user data from server
  adminHelper,         // Returns true if user is admin
  coAdminHelper,       // Returns true if user is co-admin
  canRate,             // Returns true if user can rate documents
  canEditUsers,        // Returns true if user can edit users
  canManageForms,      // Returns true if user can manage forms
  canManagePrograms,   // Returns true if user can manage programs
  canManageInstitutes, // Returns true if user can manage institutes
  hasAdminPrivileges,  // Returns true if admin OR co-admin
  logoutAcc,           // Logs out the current user
} from '../utils/auth_utils'
```

### Usage Examples

**Protect a whole page:**
```jsx
import { Navigate } from 'react-router-dom'
import { adminHelper } from '../utils/auth_utils'

const AdminPage = () => {
  if (!adminHelper()) return <Navigate to="/Dashboard" replace />
  return <div>Admin-only content</div>
}
```

**Conditionally render a button:**
```jsx
import { adminHelper, getCurrentUser } from '../utils/auth_utils'

const MyComponent = () => {
  const isAdmin = adminHelper()
  const user = getCurrentUser()

  return (
    <div>
      <button>View</button>                              {/* Everyone */}
      {isAdmin && <button>Delete</button>}               {/* Admin only */}
      {(isAdmin || user?.isRating) && <button>Rate</button>} {/* Admin or rater */}
    </div>
  )
}
```

**Conditional sidebar links:**
```jsx
import { adminHelper, coAdminHelper, getCurrentUser } from '../utils/auth_utils'

const Sidebar = () => {
  const isAdmin = adminHelper()
  const isCoAdmin = coAdminHelper()
  const user = getCurrentUser()

  return (
    <nav>
      <SidebarLink to="/Dashboard" text="Dashboard" />                        {/* Everyone */}
      {isAdmin && <SidebarLink to="/Settings" text="Settings" />}             {/* Admin only */}
      {(isAdmin || isCoAdmin) && <SidebarLink to="/Reports" text="Reports" />} {/* Admin/Co-Admin */}
      {(isAdmin || user?.isRating) && <SidebarLink to="/Accreditation" text="Accreditation" />}
      {(isAdmin || user?.crudProgramEnable) && <SidebarLink to="/Programs" text="Programs" />}
    </nav>
  )
}
```

---

## Route Protection

Wrap routes in your router configuration to protect entire pages:

```jsx
import { Navigate } from 'react-router-dom'
import { adminHelper, coAdminHelper } from './utils/auth_utils'

// Admin-only route
const AdminRoute = ({ children }) => {
  return adminHelper() ? children : <Navigate to="/Dashboard" replace />
}

// Admin or Co-Admin route
const PrivilegedRoute = ({ children }) => {
  return (adminHelper() || coAdminHelper()) ? children : <Navigate to="/Dashboard" replace />
}

// In your router:
<Route path="/settings" element={<AdminRoute><Settings /></AdminRoute>} />
<Route path="/reports"  element={<PrivilegedRoute><Reports /></PrivilegedRoute>} />
```

---

## Backend API Protection

Located in `backend/routes.py`. All protected routes use `@jwt_required()`.

**Admin only:**
```python
@app.route('/api/user', methods=['POST'])
@jwt_required()
def create_user():
    current_user_id = get_jwt_identity()
    user = Employee.query.filter_by(employeeID=current_user_id).first()
    if not user or not user.isAdmin:
        return jsonify({'success': False, 'message': 'Admins only'}), 403
    # ...
```

**Admin or Co-Admin:**
```python
@app.route('/api/reports', methods=['GET'])
@jwt_required()
def get_reports():
    current_user_id = get_jwt_identity()
    user = Employee.query.filter_by(employeeID=current_user_id).first()
    if not user or (not user.isAdmin and not user.isCoAdmin):
        return jsonify({'success': False, 'message': 'Admin or Co-Admin only'}), 403
    # ...
```

**Specific permission check:**
```python
@app.route('/api/forms', methods=['POST'])
@jwt_required()
def create_form():
    current_user_id = get_jwt_identity()
    user = Employee.query.filter_by(employeeID=current_user_id).first()
    if not user or (not user.isAdmin and not user.crudFormsEnable):
        return jsonify({'success': False, 'message': 'Permission denied'}), 403
    # ...
```

---

## PermissionGate Component

Located in `frontend/src/components/PermissionGate.jsx`. A reusable wrapper for conditional rendering based on permissions.

```jsx
import { PermissionGate } from './components/PermissionGate'

// Admin only
<PermissionGate requireAdmin>
  <button>Admin Button</button>
</PermissionGate>

// Admin or Co-Admin
<PermissionGate requireCoAdmin>
  <button>Privileged Button</button>
</PermissionGate>

// Specific permission
<PermissionGate requires="crudFormsEnable">
  <button>Create Form</button>
</PermissionGate>

// Any of multiple permissions
<PermissionGate requireAny={['isRating', 'isEdit']}>
  <button>Rate or Edit</button>
</PermissionGate>
```

### Component Source

```jsx
import { getCurrentUser, adminHelper, coAdminHelper } from '../utils/auth_utils'

export const PermissionGate = ({ requires, requireAdmin = false, requireCoAdmin = false, requireAny = [], children }) => {
  const user = getCurrentUser()
  const isAdmin = adminHelper()
  const isCoAdmin = coAdminHelper()

  if (!user) return null
  if (requireAdmin && !isAdmin) return null
  if (requireCoAdmin && !isAdmin && !isCoAdmin) return null
  if (requires && !user[requires] && !isAdmin) return null
  if (requireAny.length > 0) {
    const hasAny = requireAny.some(perm => user[perm])
    if (!hasAny && !isAdmin) return null
  }

  return children
}
```

---

## Permission Cheat Sheet

| Goal | Code |
|------|------|
| Admin only button | `adminHelper() && <Component />` |
| Admin or Co-Admin | `hasAdminPrivileges() && <Component />` |
| Rating permission | `canRate() && <Component />` |
| Edit users permission | `canEditUsers() && <Component />` |
| Forms management | `canManageForms() && <Component />` |
| Programs management | `canManagePrograms() && <Component />` |
| Custom permission check | `getCurrentUser()?.yourField && <Component />` |
| Redirect non-admins | `if (!adminHelper()) return <Navigate to="/Dashboard" />` |

---

## Docker Setup

The `docker-compose.yml` defines three services:

```yaml
services:
  frontend:
    build: ./frontend
    ports:
      - "5173:5173"
    environment:
      - REACT_APP_API_URL=http://localhost:5000/api
    depends_on:
      - backend

  backend:
    build: ./backend
    ports:
      - "5000:5000"
      - "8888:8888"
    env_file:
      - ./backend/.env

  redis:
    image: redis:alpine
    ports:
      - "6379:6379"
    command: redis-server --appendonly yes
```

Hot-reloading is enabled via Docker Compose `watch` — changes to `./frontend` or `./backend` are synced into the container automatically during development.

---

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Commit your changes (`git commit -m 'Add your feature'`)
4. Push to the branch (`git push origin feature/your-feature`)
5. Open a Pull Request

---

## Repository

**GitHub:** [am-not-a-coder/Capstone-Project](https://github.com/am-not-a-coder/Capstone-Project)
