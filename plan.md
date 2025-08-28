Below is the detailed implementation plan in markdown with step‐by‐step instructions, including changes or additions to dependent files, error handling, and UI/UX considerations.

---

## Overall Architecture and Technology Stack

- **Frontend:** Next.js (React) using the app directory, TypeScript, Tailwind CSS (via globals.css), and our in‑house modular UI components (in src/components/ui).  
- **Backend/API:** Next.js API Routes (inside src/app/api) with RESTful endpoints.  
- **Database:** Supabase free tier (PostgreSQL behind the scenes) integrated via Prisma ORM (new prisma folder + schema.prisma).  
- **Authentication:** NextAuth.js using local database with role‑based access (Admin, HR, Manager, Employee).  
- **Mobile Compatibility:** Progressive Web App (PWA) support (via a manifest.json in public) with instructions for a separate React Native client if needed.  
- **Advanced AI Features (Optional):** Attendance anomaly detection and a chatbot for HR queries integrated with OpenRouter (default: anthropic/claude-sonnet-4) via separate endpoints and UI modals.

---

## File-by-File and Module Changes

### Environment and Configuration Files

1. **.env.local (new file)**
   - Add keys for `DATABASE_URL` (Supabase/PostgreSQL connection), `NEXTAUTH_SECRET`, and any SMTP details needed.
   - Example entry:
     ```
     DATABASE_URL="your_supabase_postgres_connection_string"
     NEXTAUTH_SECRET="a_strong_secret_value"
     ```
2. **package.json**
   - Add dependencies:
     - `"prisma"` and `"@prisma/client"` for ORM.
     - `"next-auth"` for authentication.
     - `"react-hook-form"` for form management.
     - `"axios"` for API calls if needed.
   - Update scripts if necessary (e.g., add a script for running prisma migrations).
3. **tsconfig.json**
   - Verify baseUrl and paths if adding new aliases (e.g., for @components or @api).
4. **next.config.ts**
   - Ensure environment variables are exposed to the server only (or via runtime config if needed).
   - Enable PWA settings if you decide to add a manifest later.

---

## Prisma and Database Setup

1. **/prisma/schema.prisma (new file)**
   - Define models for:
     - `Employee` (fields: id, personalInfo, professionalInfo, documents, createdAt, updatedAt)
     - `Attendance` (fields: id, employeeId, clockIn, clockOut, location, late, overtime, earlyExit)
     - `Leave` (fields: id, employeeId, leaveType, startDate, endDate, status)
     - `Payroll` (fields: id, employeeId, salary, deductions, bonuses, netPay, generatedAt)
     - `Performance` (fields: id, employeeId, kpis, reviews, goals, feedback, reviewedAt)
     - `Candidate` (fields for recruitment activity)
     - `Compliance` (fields: id, documentName, expiryDate, reminderSent)
   - Include relations and validations.
   - Run: `npx prisma migrate dev --name init` to set up the database schema.
2. **Database error handling:**  
   - Use try/catch blocks in API endpoints during database queries.
   - Return error responses in a consistent JSON schema with a status code and error message.

---

## API Endpoints (inside src/app/api)

Create a folder structure under `src/app/api/` with the following routes. All endpoints must follow RESTful practices and include proper error handling:

1. **Employee Management (src/app/api/employees/route.ts)**
   - GET: List all employees.
   - POST: Create a new employee profile (with file upload for documents).
   - PUT: Update employee details.
   - DELETE: Remove an employee.
   - Use multipart support for file uploads via next middleware.
2. **Attendance & Time Tracking (src/app/api/attendance/route.ts)**
   - POST: Register clock-in/clock-out.  
   - Validate geolocation by comparing the submitted coordinates with the office location (hardcode/configure allowed latitude, longitude, and radius).
   - Include checks for duplicate clock-ins and calculate timestamps.
3. **Leave Management (src/app/api/leaves/route.ts)**
   - GET: Retrieve leave requests (filtered by role).
   - POST: Submit new leave request.
   - PUT: Approve/reject a leave request (by managers/HR).
4. **Payroll Management (src/app/api/payroll/route.ts)**
   - GET: List payroll records for employees.
   - POST: Trigger salary calculation based on attendance, leave, and other factors; auto-generate a payslip.
5. **Performance Management (src/app/api/performance/route.ts)**
   - GET/POST: Create and update performance reviews, track KPIs, set goals and collect feedback.
6. **Recruitment & Onboarding (src/app/api/recruitment/route.ts)**
   - GET: Retrieve job postings and candidate applications.
   - POST: Create job postings and track candidate statuses.
7. **Compliance & Policy Management (src/app/api/compliance/route.ts)**
   - GET: List company policy documents.
   - POST: Upload/update compliance documents along with auto-reminder settings.
8. **Reports & Analytics (src/app/api/reports/route.ts)**
   - GET: Generate reports (attendance trends, payroll summaries, performance analysis) with filters (daily, weekly, monthly).

*Each API file should use async functions with try/catch error handling and return a consistent JSON structure.*

---

## Authentication and Authorization

1. **NextAuth.js Integration**
   - Create `src/app/api/auth/[...nextauth]/route.ts` to set up authentication routes.
   - Use credentials, Google OAuth, or email sign‑in based on configuration.
   - Secure API endpoints using middleware that checks user roles before processing sensitive requests.
2. **Middleware**
   - Create a shared middleware (in src/lib/utils.ts or a new file under src/middleware/) to verify JWT or session data.
   - Use this middleware in each critical API route.

---

## Frontend & UI Components

### Dashboard and Pages (inside src/app/dashboard)

1. **Employee Management Page (src/app/dashboard/employees/page.tsx)**
   - Display a data table with employee list (Name, Department, Role, etc.) using our existing table component.
   - Include an “Add Employee” button that opens a modal (use our dialog component) with the `EmployeeForm` component.
   - Error handling: Display user-friendly error messages on API failure.
2. **Attendance Page (src/app/dashboard/attendance/page.tsx)**
   - Modern UI with two large buttons: “Clock In” and “Clock Out”.  
   - On click, prompt for geolocation (using the browser’s `navigator.geolocation`) and send captured coordinates to the API.  
   - Show status messages (e.g., “You are within the allowed area” or “Your location could not be verified”).
3. **Leave Management Page (src/app/dashboard/leaves/page.tsx)**
   - Display calendar and leave request table.  
   - Add a form for submitting leave requests (using react-hook-form for validation).
4. **Payroll Management Page (src/app/dashboard/payroll/page.tsx)**
   - Render payroll summaries using card components from our UI kit.  
   - Provide an option to download payslips (triggering a payroll API endpoint).
5. **Performance Management Page (src/app/dashboard/performance/page.tsx)**
   - Provide forms for performance reviews, KPI tracking, and 360° feedback.
6. **Recruitment & Onboarding Page (src/app/dashboard/recruitment/page.tsx)**
   - Display current job postings and integrate a candidate tracking list.
7. **Compliance Page (src/app/dashboard/compliance/page.tsx)**
   - List company policy documents and deadlines for renewals.
8. **Reports & Analytics Page (src/app/dashboard/reports/page.tsx)**
   - Use the existing chart component (src/components/ui/chart.tsx) to display analytics.
   - Provide filters for date ranges and department-wise breakdowns.

### Shared UI Components (inside src/components)

1. **EmployeeForm.tsx (new file)**
   - Create a form with sections for personal details, professional details, and file uploads.
   - Use built‑in Input, Label, and Button components from src/components/ui.
2. **AttendanceClock.tsx (new file)**
   - Component to capture current geolocation and render “Clock In/Clock Out” buttons.
3. **LeaveForm.tsx (new file)**
   - A responsive form for leave application that includes date pickers and reason fields.
4. **PayrollSummary.tsx, PerformanceReview.tsx, RecruitmentList.tsx, ComplianceDocuments.tsx, and ReportsChart.tsx (new components)**
   - Each component to be built modularly using our existing UI primitives.
   - Ensure error handling (e.g., fallback messages if API data is missing).

---

## File Upload and Document Handling

- Use Next.js API routes to accept file uploads (e.g., employee documents, compliance files). Files may be stored in a dedicated `/public/uploads` folder.
- Validate file types, sizes, and include error messages for invalid uploads.

---

## Mobile & PWA Enhancements

1. **PWA Setup**
   - Create a `public/manifest.json` that defines the app icon (use a locally hosted asset URL if needed), short name, background color, and display options.
2. **Responsive Design**
   - Ensure that all page layouts in the dashboard use responsive CSS classes (Tailwind) and the useMobile hook (src/hooks/use-mobile.ts) to render mobile‑friendly UIs.
3. **Future React Native Mobile App**
   - Document API endpoints and data contracts so a separate React Native project can consume the same backend.

---

## Advanced AI Features (Optional)

1. **Attendance Anomaly Detection**
   - Create a new API endpoint (src/app/api/attendance/anomaly/route.ts) which uses an LLM integration to detect potential proxy attendance or anomalous patterns.
   - In the endpoint, send activity details to the LLM API (using OpenRouter’s endpoint `https://openrouter.ai/api/v1/chat/completions` with model "anthropic/claude-sonnet-4") and return analysis.
2. **HR Chatbot for Queries**
   - Create `src/app/api/hr/chatbot/route.ts` that receives HR questions, calls the LLM API, and returns responses.
   - UI Integration: Add a chatbot modal in the dashboard that accepts text queries from employees.
   - Ensure the message structure uses the required array format (role and content using type “text”).

---

## Deployment & Documentation

1. **README.md**
   - Update with full instructions on setting up environment variables, installing dependencies, database migration (Prisma commands), and running the development server.
   - Include curl commands to test key API endpoints.
2. **Deployment Instructions**
   - Provide guidelines for deploying on Vercel (for the Next.js frontend/API) and instructions for setting up Supabase as the database.
   - Mention steps for setting up auto‑backup of the database.
3. **Error Handling & Testing**
   - Document how errors are logged both on the client and server.
   - Provide sample curl tests for critical endpoints as listed in the testing protocol.

---

## Summary

- The plan covers setting up environment variables, adding Prisma for a Supabase database, and NextAuth for authentication.  
- New API routes are created for employee, attendance, leave, payroll, performance, recruitment, compliance, and reporting, each with robust error handling.  
- Dashboard pages for each HR module are built using modern, responsive UI components, ensuring mobile-friendliness with PWA support.  
- Shared components (forms, tables, modals) are designed with our existing UI kit, adhering to modern design principles without external icon libraries.  
- Advanced AI features (attendance anomaly detection and an HR chatbot) are integrated via additional API endpoints using an LLM provider.  
- Comprehensive deployment and documentation instructions, including testing with curl commands, ensure a smooth production rollout.
