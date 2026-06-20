# PeopleHub HRMS — Production Build

A full-stack HRMS for Internovo Group (Interface Projects LLP, Indihomes, One Bharat, Investments), built with React + Vite + TypeScript + Supabase.

This replaces the static single-file HTML prototype with a real backend: Postgres database, Supabase Auth, and role-based access control enforced at the database layer (Row Level Security) — not just hidden in the UI.

## What changed vs. the prototype
- **Persistence**: all data (employees, leave, attendance, payroll) lives in Postgres, not in-memory JS arrays that reset on refresh.
- **Auth**: real login via Supabase Auth (email/password). No more "logged in as whoever the dropdown says."
- **RBAC**: four roles — `super_admin`, `hr_admin`, `manager`, `employee` — enforced via Postgres RLS policies, so even a direct API call can't bypass permissions.
- **Multi-company**: every employee/payroll/leave/attendance row is scoped to a company (`interface`, `indihomes`, `onebharat`, `investments`).

## Setup

### 1. Create a Supabase project
Go to supabase.com → New Project.

### 2. Run the schema
Open the SQL Editor in your Supabase dashboard and run `supabase/schema.sql`. This creates all tables, RLS policies, and seeds the 4 Internovo Group companies.

### 3. Configure environment
```
cp .env.example .env
```
Fill in `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY` from Project Settings → API.

### 4. Install & run
```
npm install
npm run dev
```

### 5. Create your first user
Easiest path: Supabase Dashboard → Authentication → Add User (set email + password). A `profiles` row is auto-created via trigger with role `employee`.

Then promote yourself to admin by running in SQL Editor:
```sql
update profiles set role = 'super_admin' where email = 'you@yourcompany.com';
```

### 6. Add employees
Log in → Employees → Add Employee. To let an employee log in and see their own leave/attendance/payslips, link their `profiles.employee_id` to the employee row (and optionally `employees.user_id` back to the profile).

## Roles
| Role | Can do |
|---|---|
| `super_admin` | Everything, including managing companies |
| `hr_admin` | Full HR operations across all companies |
| `manager` | View/approve leave & attendance for direct reports |
| `employee` | Self-service: apply leave, view own attendance & payslips |

## Module status
- ✅ Dashboard (role-aware stats)
- ✅ Employees (full CRUD)
- ✅ Leave (apply, approve/reject, RLS-scoped visibility)
- ✅ Attendance (daily marking by HR, self-view for employees)
- ✅ Payroll (Indian salary breakdown: Basic/HRA/PF/PT, process & mark-paid)
- ✅ Reports (headcount by company/department, leave status breakdown)

## Not yet built (flag if you want these next)
- Holiday calendar UI (table exists in schema, no page yet)
- Performance reviews UI (table exists in schema, no page yet)
- Bulk CSV import for employees
- Payslip PDF export
- Email notifications on leave decisions
