# COMP 3330: JavaScript Frameworks and Servers

## Next.js Portfolio Lab Series


# 📘 Five-Session Next.js Portfolio Course (3 Hours Each)

📌 General Guidelines:
- Each session includes coding and short testing/debugging.
- Next.js is the main platform for routing, API handling, and SSR/SSG.
- Backend can be built using Next.js API routes or an external server.
- Mock data first, then replace with real APIs where applicable.
- Keep the dashboard and deployment for the final session, to end on a high note.

---

## 🧱 Session 1: Project Setup & Homepage Foundations

### 👉[Navigate to Lab's Page](/labs/lab1.md)

✅ Objectives:
- Initialize a new Next.js app and install TailwindCSS.
- Install Shadcn/ui.
- Create base folder structure (components, pages, lib, etc.).
- Build and style:
    - 🧭 Navbar
    - 🧱 Hero section
    - 🔗 Project preview cards (hardcoded)
- Set up global layout and reusable components.
- Practice routing between pages.

💡 Outcome: You will have a styled homepage with working navigation and preview cards, ready for expansion.

---

## 📂 Session 2: Projects Page & Dynamic Routing

### 👉[Navigate to Lab's Page](/labs/lab2.md)

✅ Objectives:
- 📁 Create /projects page with a dynamic list of all projects.
- 🔗 Add individual project detail pages via [slug].js or [id].js.
- 🧪 Use mock JSON or local array as temporary project data source.
- Introduce getStaticProps or getServerSideProps.
- Optional: Add tag/category support for filtering projects.

💡 Outcome: You have full navigation from homepage → projects list → individual project details.

---

## 📤 Session 3: Contact Page, GitHub API & Notifications

### 👉[Navigate to Lab's Page](/labs/lab3.md)

✅ Objectives:
- 🚫 Add a generic yet Dynamic 404 Page, displaying a message
- 📤 Build a contact form with Name, Email, Message fields.
- 📬 Set up a Next.js API route to send emails (e.g., via Resend or Nodemailer).
- 🪄 Add toast notifications for:
- Form submission success/failure
- Project creation (preview for upcoming dashboard work)
- 🛠️ Create GitHub component:
- Fetch recent contributions or public repos using GitHub API
- Display them on homepage or sidebar

💡 Outcome: Functional contact flow and GitHub integration, with responsive feedback through toasts.

---

## 🔐 Session 4: Authentication & Protected Dashboard

### 👉[Navigate to Lab's Page](/labs/lab4.md)

✅ Objectives:
- Auth0: configure tenant, wire SDK, and expose helpers to read the session in server components and API routes.
- Protected routes: gate `/dashboard` (and future admin areas) so unauthenticated visitors are redirected to Auth0 login.
- Database: connect to Neon Postgres, seed projects, and build reusable CRUD helpers.
- API: ship authenticated CRUD endpoints under `/api/projects`.
- UI wiring: show/hide Add/Edit/Delete controls based on session and fetch projects through API routes.
- Editing: add `/projects/[uuid]/edit` with a prefilled form that updates or deletes DB rows.

💡 Outcome: Authenticated users gain access to a styled, protected dashboard with placeholder content editors.

---

## 🧠 Session 5: CRUD + DB Integration + Deployment

### 👉[Navigate to Lab's Page](/labs/lab5.md)

✅ Objectives:
- Persist hero profile (avatar + copy) in Neon Postgres.
- Authenticated hero CRUD at `/api/hero` (public GET, guarded PUT).
- Protected `/dashboard` with a hero editor form (file upload → data URL).
- Homepage hero pulls live DB data with safe fallbacks.
- Deploy-ready: Vercel envs + Auth0 callback/origin updates; smoke test live login/logout.

💡 Outcome: Live site with a protected hero editor, `/api/hero` wired to Neon, homepage reading fresh DB content, and Vercel deployment with working Auth0 login/logout.
