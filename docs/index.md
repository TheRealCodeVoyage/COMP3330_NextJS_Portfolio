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

✅ Objectives:
- 🔐 Build a login page (email + password).
- 🔒 Protect the /dashboard route using middleware or session check.
- 🧑‍💻 Start dashboard layout with tabs or cards for:
- Project Management (CRUD)
- Hero Content Editing
- 🔑 Implement .env setup for secrets (Resend key, DB credentials, etc.)

💡 Outcome: Authenticated users gain access to a styled, protected dashboard with placeholder content editors.

---

## 🧠 Session 5: CRUD + DB Integration + Deployment

✅ Objectives:
- 🧠 Connect frontend to API routes for full-stack CRUD operations.
- 🗃️ Integrate a simple database (e.g., SQLite, Supabase, or Prisma + PostgreSQL).
- Store: projects and hero section content
- Read: on homepage, projects page
- Write/update/delete: from dashboard
- 🚀 Deploy entire site on Vercel (or Netlify as fallback)
- Final review:
- 🔍 Clean up code
- ⛳ Test all flows
- 🎯 Showcase possible optional extensions

💡 Outcome: You walk away with a deployed, dynamic portfolio site with full-stack features and admin capabilities.
