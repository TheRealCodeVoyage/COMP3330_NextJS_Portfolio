# Lab 3 – Next.js Portfolio: Contact, Notifications & GitHub

Dynamic 404, Contact Form → API Email, Toast UX, and GitHub Integration￼

---

## 🎯 Lab 3 Objectives
1.	Dynamic 404: Add a generic yet dynamic 404 page that can display a custom message.
2.	Contact Form: Build a page section with Name, Email, Message (validated).
3.	Email API: Wire the form to a Next.js API route that sends email (Resend or Nodemailer).
4.	Toasts: Show non-blocking success/error toasts for:
    - contact form submission
    - project creation
5.	GitHub: Create a GitHub component that shows either recent contributions or public repos, and place it on the homepage or a sidebar.

---

## 📚 What You’ll Learn
- App Router file conventions for error/404 UIs.
- Client-side validation and accessible form UX.
- Building a POST API route that accepts payloads and returns JSON.
- Integrating third-party email providers with .env secrets.
- Non-blocking UI feedback with toasts (optimistic UX patterns).
- Lightweight GitHub integration (external script or REST API) within the App Router.

---

## ✅ End Result (By the End of Lab 3)
- Two custom 404 pages that renders a friendly UI
    - Generic 404 page which can accept an optional message.
    - Spesific 404 page for slugified project page if project is not found.
- A styled contact form that validates inputs, submits to `/api/contact-me`, and reports status via toasts.
- A working email sender ([Resend](https://resend.com/)) via your API route.
- A GitHub widget/component build base on the [3rd-Party GitHub Contribution Calendar Widget](https://github.com/imananoosheh/github-contributions-widget?tab=readme-ov-file#github-contribution-calendar-widget) visible in your site (homepage/aside), showing contributions.

---

## 🧰 Prerequisites
- Your Lab 2 codebase (App Router, Tailwind, shadcn/ui installed).
- Create an account in [Resend](https://resend.com), get an API Key
- Create you `.env` file and populate as below:

```.env
RESEND_API_KEY=your_key_here
RESEND_FROM=your-resend-username@resend.dev
RESEND_TO=your-email-address-to-receive-contact-message
```

Tip: Keep secrets only in `.env` or `.env.local` and never commit that file.

---

## 🗂️ Suggested Folder Additions

```text
src/
├── app/
│   ├── contact-me/
│   │       └── route.jsx         # Backend handling contact form request
│   ├── api/
│   │   └── contact-me/
│   │       └── route.js          # POST /api/contact-me -> send email
│   ├── not-found.jsx             # App-level custom 404
│   └── projects/
│       └── not-found.jsx         # project specific 404
└── components/
   ├── contact-form.jsx           # Reusable form component
   └── github-calendar.jsx        # Github calendar component
```

---

## 🧭 Step-by-Step Instructions

### 1) Dynamic 404 (App-Level)

#### Generic 404 Page

- Create src/app/not-found.jsx. Next.js will render this for unknown routes.
- Keep it simple and friendly:
- A bold heading (“Page not found”)
- A short description that can show a message if passed
- A button link back to `/`
- Do not over-engineer: focus on composable UI (e.g., shadcn Card, Button).
- Hint: You can design your component to accept a message, even though App-level not-found doesn’t receive route params. You’ll still reuse the same UI in route-level 404s later (if you decide to have per-segment not-found.jsx).

✅ Test: Manually type a nonsense URL (e.g., /xyz) and confirm your UI renders properly.

#### Project Specific 404 Page

- Create src/app/projects/not-found.jsx. This is a segement-level _not-found_. Next.js will prioritize this 404 over the generic one when user is asking for anything below `/projects/` route.
- To display more intractive and customized message, you can use `useParams` from `next/navigation` to capture the last part of the router where we have no corespondance for. you can capture the `slug` from `params` and let user know this is not a valid project.

---

### 2) Contact Form (Client Component + Validation)
- Build `components/contact-form.jsx` using:
    - shadcn/ui: `Form`, `FormField`, `FormItem`, `FormLabel`, `FormControl`, `Input`, `Textarea`, `Button`, `FormMessage`
    - `useForm` from `react-hook-form`
    - `zod` + `zodResolver` from `@hookform/resolvers/zod`
- Define an Schema for the `message` (along with user's `email` and `name`)
- UX details to cover:
- Required labels, helpful placeholders.
- Mount it in `src/app/contact/page.jsx`. Keep layout consistent with your design system.

✅ Test: Trigger validation (empty fields, invalid email) and confirm messages show near fields.

---

### 3) Contact API Route (POST `/api/contact-me`)
- Create `src/app/api/contact-me/route.js`.
- Your handler should:
    - Guard non-POST methods (return 405).
    - Parse input: You can send FormData or JSON from the client. Choose one and stay consistent.
    - If you use FormData, extract values via await req.formData() and formData.get("name"), etc.
    - If you use JSON, await req.json().
- **Optionally**: You can move the schema into a shared, server-safe module (no 'use client') and import it in route.js. In this way, we will not need to redefine the schema. Now you can validate server-side (lightweight repeat of zod is recommended).
- Send the email using Resend object: create a client with `RESEND_API_KEY` and call its `emails.send()` function.
- Return JSON { ok: true, message: "Email sent" } on success; on failure return { ok: false, message: "..." } with 4xx/5xx status.
- You message could be crafted using `data` and `error` outputs:

```js
const { data, error } = await resend.emails.send({
        from: '',
        ...
    });
```

🔒 Security: Never trust client input—always validate on server too.

✅ Test: Open DevTools → Network. Submit the form and confirm:
- Request hits /api/contact-me
- Response { ok: true } on success (or meaningful error body on failure)

Minimal method guard (only if stuck):

```js
export async function POST(req) {
  // parse + send email + return JSON
}

export function GET() {
  return Response.json({ error: "Method Not Allowed" }, { status: 405 });
}
```

(Stick to the outline above; implement the email call yourself.)

---

### 4) Toast Notifications (Contact + Project Create)
- Install a toast library if you haven’t already
    - note: Toast have been merged into **Sonner**

```bash
npx shadcn@latest add sonner
```

- Follow the installation and Setup on [shadcn/ui Sonner Docs](https://ui.shadcn.com/docs/components/sonner) and add its provider once in `app/layout.js`.
- Contact: On submit:
    - Show a loading toast while awaiting the API.
    - On success → “Message sent!”
    - On error → “Failed to send message. Try again.”
- Projects → New (from Lab 2):
    - After successful `POST` to `/api/projects/new`, show a success toast: “Project received successfully” (you’ll wire real DB later).

✅ Test: Manually trigger success & failure (e.g., temporarily force your API to throw) to see both toast paths.

---

### 5) GitHub Activity Calendar Component 

You’ll surface GitHub activity on your site.

Repo to use: [GitHub Calendar Activity](https://github.com/imananoosheh/github-contributions-widget)

Contributions Calendar (external script)
- Create `components/github-calendar.jsx`.
- Use Next’s `<Script />` to load the widget script from a CDN and render a placeholder `<div>` that the script populates.
- Keep the component server-safe by not passing event handlers through props from a server component to client-only script tags (avoid onLoad on server surfaces).
- Style the surrounding container (title, caption/link to your profile).

✅ Test: Confirm the component renders on the homepage or a sidebar section. Keep it responsive, with sensible spacing and headings.

---

## 🧪 Test Checklist
- 404 page:
- Visiting an unknown path shows your custom 404 UI.
- “Back home” button navigates correctly.
- Contact form:
- Client validation prevents empty/invalid submissions.
- Submitting calls POST `/api/contact-me`.
- Successful response → success toast; failure → error toast.
- Email service:
- Resend keys live in `.env`.
- API route returns meaningful JSON + status codes.
- Project creation toast:
- Visiting `/projects/new`, submit a mock project → see success toast on success.
- GitHub:
- A GitHub component (calendar or repos) renders on home/aside.
- Layout remains responsive and consistent with the design system.

---

## 🧯 Troubleshooting & Notes
- 404 doesn’t show? Ensure the file is exactly `app/not-found.jsx` (App Router). Avoid Page Router patterns.
- Event handler warnings with external scripts: don’t pass client-only event props from server components. If needed, make the component "use client" and scope script handling appropriately.
- Email not arriving:
- Check provider dashboard/logs.
- Try sending to your own inbox first; check spam.
- Toasts not visible:
- Provider not mounted? Ensure `<Toaster />` (or library equivalent) is placed once in app/layout.js.

---

## 📌 Next Steps (Optional Enhancements)
- Add server-side zod validation to `/api/contact-me.`
- BCC the sender so they receive a copy/receipt.
- Persist contact requests to a DB (audit trail) in addition to sending email.
- Add loading states (skeletons/spinners) for the GitHub component and contact submission.
- Centralize toast patterns into a small helper (loading → success/error).

---

## Final checklist before submission
- [ ] `app/not-found.jsx` exists and renders a friendly UI with a “Back Home” button.
- [ ] Contact UI:
    - [ ] Name, Email, Message fields with client validation.
    - [ ] Submit disabled during request.
- [ ] API:
    - [ ] POST `/api/contact-me` implemented (FormData or JSON—be consistent).
    - [ ] Server-side validation (at least minimal) and proper status codes.
    - [ ] Email service (Resend) wired with .env secrets.
- [ ] Toasts:
    - [ ] Contact form success and error toasts appear.
    - [ ] Project creation success toast triggered on /projects/new after submit.
- [ ] GitHub:
    - [ ] A visible GitHub component (calendar or repos) is integrated and responsive.
- [ ] Code quality:
    - [ ] No secrets committed.
    - [ ] Components are modular; UI consistent with shadcn/tailwind style.
    - [ ] Meaningful error handling and empty states.

---

## Submission
- Provide full-page screenshots of:
    - The 404 pages on an invalid route
        - Generic 404 Page
        - Specific Project 404 page (segment-level 404)
    - The Contact page/component (before and after a successful submit) + toast message included
    - The GitHub component in place (home/aside)
- Commit and push all Lab 3 changes to your GitHub repo and submit the repository link.