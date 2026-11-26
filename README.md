# Employee Onboarding Automation Portal

A full-stack employee onboarding management application built with **React**, **FastAPI**, **SQLAlchemy**, and **PostgreSQL**. It provides an HR-facing workspace to manage employees, departments, onboarding tasks, required documents, and onboarding progress.

> **Project status:** Learning/demo project. Authentication is currently disabled in the backend. Do not deploy this application to the public internet or use it with real employee data until the security items in [Security Notes](#security-notes) have been addressed.

## Features

- HR dashboard with onboarding overview and employee progress
- Employee creation, listing, details, updates, and deletion
- Department management
- Automatic creation of eight default onboarding tasks for each new employee
- Automatic creation of three required document slots: ID proof, PAN, and education certificate
- Task completion tracking and dynamically calculated progress
- Document upload, review, verification, and rejection workflow
- Onboarding completion check based on **all tasks completed** and **all required documents verified**
- FastAPI interactive API documentation

## Technology Stack

| Layer | Technology |
| --- | --- |
| Frontend | React 18, React Router, Axios |
| Frontend tooling | Vite |
| Backend | Python, FastAPI, Uvicorn |
| ORM / validation | SQLAlchemy, Pydantic |
| Database | PostgreSQL |
| File handling | FastAPI UploadFile and local server-side storage |

## Application Architecture

The frontend communicates with the backend through REST endpoints. Backend responsibilities are separated into routers, services, repositories, and database models.

```text
React UI (Vite)
     |
     | Axios / REST (/api)
     v
FastAPI Routers
     |
     v
Service Layer
     |
     v
Repository Layer
     |
     v
SQLAlchemy
     |
     v
PostgreSQL
```

- **Routers** define HTTP endpoints and request/response handling.
- **Services** implement onboarding, employee, task, department, document, and dashboard logic.
- **Repositories** encapsulate database queries.
- **Models and schemas** represent database entities and validate API data.

## Repository Structure

```text
employee-onboarding-portal/
├── backend/
│   ├── app/
│   │   ├── core/          # Configuration and upload handling
│   │   ├── database/      # SQLAlchemy connection/session
│   │   ├── models/        # Database models
│   │   ├── repositories/  # Data access layer
│   │   ├── routers/       # FastAPI endpoints
│   │   ├── schemas/       # Pydantic schemas
│   │   └── services/      # Business logic
│   ├── .env.example
│   └── requirements.txt
├── database/
│   ├── 01_create_database.sql
│   ├── 02_create_tables.sql
│   └── 03_seed_data.sql
└── frontend/
    ├── src/
    │   ├── components/
    │   ├── pages/
    │   ├── services/
    │   └── routes.js
    ├── package.json
    └── vite.config.js
```

## Prerequisites

Install the following before running the project:

- Python 3.10 or newer
- Node.js and npm
- PostgreSQL
- Git

## Setup and Run

### 1. Clone the repository

```bash
git clone https://github.com/G534767/employee-onboarding-portal.git
cd employee-onboarding-portal
```

### 2. Create and initialize the database

Run the SQL scripts in this order:

1. Connect to PostgreSQL's administrative `postgres` database and execute `database/01_create_database.sql`.
2. Connect to the new `employee_onboarding` database and execute `database/02_create_tables.sql`.
3. Execute `database/03_seed_data.sql` to insert sample departments, employees, and related records.

**Important:** `02_create_tables.sql` drops existing application tables before recreating them. Running it against a database with data will delete that data. Use it only for a fresh setup or after backing up anything important.

You can run the scripts using psql or a PostgreSQL client such as DBeaver.

### 3. Configure and start the backend

Open a terminal in the `backend` directory:

```bash
cd backend
python -m venv .venv
```

Activate the virtual environment:

**Windows PowerShell**
```powershell
.venv\\Scripts\\Activate.ps1
```

**macOS / Linux**
```bash
source .venv/bin/activate
```

Install dependencies and create your local environment file:

```bash
pip install -r requirements.txt
```

Copy `.env.example` to `.env`, then set `DATABASE_URL` to match your local PostgreSQL username, password, host, port, and database. The example file uses port `5433`; change it if your PostgreSQL server listens on a different port.

Start FastAPI from the `backend` directory:

```bash
uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
```

Backend URLs:

- API root: http://127.0.0.1:8000/
- Health check: http://127.0.0.1:8000/health
- Interactive API docs: http://127.0.0.1:8000/docs

### 4. Install and start the frontend

Open another terminal:

```bash
cd frontend
npm install
npm run dev
```

Open the frontend at:

http://localhost:5173/onboarding-portal/hr/dashboard

Vite proxies requests beginning with `/api` to the backend at `http://127.0.0.1:8000`. Start the backend before using the frontend.

## Onboarding Workflow

1. HR creates an employee and assigns a department and relevant details.
2. The backend creates eight default onboarding tasks and three required document slots.
3. The employee's onboarding tasks are tracked as pending or completed.
4. Required documents can be uploaded and reviewed by HR.
5. Progress is calculated from completed tasks divided by total tasks.
6. Onboarding is marked **COMPLETED** only when every task is completed and all three required document types are verified.

Document uploads currently accept PDF, PNG, JPG/JPEG, DOC, and DOCX files, with a maximum size of 5 MB per file.

## Main API Groups

The backend mounts these route groups under `/api`:

| Resource | Base path | Purpose |
| --- | --- | --- |
| Employees | `/api/employees` | Employee CRUD |
| Departments | `/api/departments` | Department CRUD |
| Tasks | `/api/tasks` | Task tracking and completion |
| Documents | `/api/documents` | Document submission and review |
| Dashboard | `/api/dashboard` | HR, manager, and employee dashboard data |

See the interactive API documentation at `/docs` while the backend is running for request schemas and endpoint details.

## Configuration and Uploaded Files

- Backend settings are loaded from `backend/.env`.
- Start with `backend/.env.example`; never commit your own `.env` file or real credentials.
- Uploaded employee documents are stored under `backend/uploads/` by default. This directory is ignored by Git and should never be committed.
- The repository's `.gitignore` excludes virtual environments, secrets, caches, frontend build output, logs, and uploaded documents.

## Security Notes

This repository reflects the supplied project implementation. The current backend explicitly does **not** register the authentication router, and the frontend API client does not attach authentication credentials. As a result, the API routes—including employee records, document access, uploads, verification, and rejection—are not protected by an authenticated user session or role-based authorization.

Before any production use, implement and test at least:

- Authentication and role-based authorization for HR, managers, and employees
- Authorization checks on every employee and document resource
- Secure handling of uploaded files, including malware scanning and access controls
- Production-safe secrets, database permissions, HTTPS, and CORS configuration
- Audit logging for sensitive employee and document actions
- Automated backend and frontend tests, plus deployment and backup procedures

Use only synthetic sample data in a local development environment until these controls are implemented.

## Testing Status

The supplied archive did not include automated test files. The project should be treated as not yet covered by an automated test suite. Validate the database setup and run the backend and frontend locally before making changes or demonstrations.

## Future Improvements

- Add authentication and role-based access control
- Add automated unit and integration tests
- Add deployment configuration and CI checks
- Add audit history and notifications
- Add UI screenshots and a short demo walkthrough

## License

No license file was included with the supplied project. All rights remain with the respective copyright holder unless a license is added.
