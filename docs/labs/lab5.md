# Lab 5 – Next.js Portfolio: Hero CRUD & Deployment Prep

Editable hero content in the protected dashboard, data-URL avatars, and Vercel-ready envs.

---

## 🎯 Lab 5 Objectives
1. Persist a hero profile (avatar + copy) in Neon Postgres.
2. Expose authenticated CRUD for the hero at `/api/hero`.
3. Build a protected `/dashboard` with a Hero editor form (file upload → data URL).
4. Wire the homepage hero to live DB data with safe fallbacks.
5. Prepare for Vercel deploy: env vars, Auth0 callback/origin updates, and smoke tests.

---

## 📚 What You’ll Learn
- Auth0-protected API routes in the App Router.
- Converting uploaded files to data URLs on the server (no external storage).
- FormData + zod validation with `withApiAuthRequired`.
- Reusable DB helpers for single-row content.
- Mapping localhost OAuth settings to a hosted Vercel domain.

---

## ✅ End Result (By the End of Lab 5)
- `/dashboard` shows a Hero editor for signed-in users; anonymous users see a login prompt.
- `/api/hero` supports GET (public) and PUT (authenticated) with text fields + avatar upload.
- Homepage hero renders DB content; placeholders only when empty.
- Vercel deployment with working Auth0 login/logout on the live URL.

---

## 🧰 Prerequisites
- Lab 4 completed on the same repo/branch.
- Auth0 tenant + Regular Web App from Lab 4.
- Neon DB URL available.
- Vercel account (free tier OK) with GitHub repo access.
- `.env.local` ready to mirror into Vercel:
  - `AUTH0_SECRET`, `AUTH0_BASE_URL`, `AUTH0_ISSUER_BASE_URL`, `AUTH0_CLIENT_ID`, `AUTH0_CLIENT_SECRET`
  - `NEON_DB_URL`

---

## 🗂️ Suggested Folder Additions

```text
src/
├── app/
│   ├── api/
│   │   └── hero/route.js          # GET/PUT with Auth0 guard + file→data URL
│   └── dashboard/page.jsx         # protected dashboard + hero form
├── components/
│   └── hero-editor-form.jsx       # client form for hero CRUD
└── lib/
    └── db.js                      # hero table helpers (ensure/get/upsert)
```

---

## 🧭 Step-by-Step Instructions

### 1) Hero Table & Helpers (server)
- Add hero helpers to `src/lib/db.js` (similar shape to projects). Target a single-row table; merge defaults → current → updates.

```js
const HERO_PLACEHOLDER_AVATAR = "data:image/gif;base64,R0lGODlhAQABAAAAACw=";
const defaultHeroContent = {
  avatar: HERO_PLACEHOLDER_AVATAR,
  fullName: "...",
  shortDescription: "...",
  longDescription: "...",
};

export async function ensureHeroTable() {
  await sql`
    create table if not exists hero (
      id uuid primary key,
      avatar text not null default '',
      full_name text not null,
      short_description text not null check (char_length(short_description) <= 120),
      long_description text not null,
      created_at timestamptz not null default now(),
      updated_at timestamptz not null default now()
    );
  `;
  const [{ count }] = await sql`select count(*)::int as count from hero`;
  if (Number(count) === 0) await seedHeroTable();
}

export async function getHero() {
  await ensureHeroTable();
  const [row] = await sql`
    select id, avatar, full_name, short_description, long_description,
           created_at as "createdAt", updated_at as "updatedAt"
    from hero
    order by created_at asc
    limit 1;
  `;
  return row ? mapHeroRow(row) : null;
}

export async function upsertHero(updates = {}) {
  await ensureHeroTable();
  const current = await getHero();
  // merge defaults → current → updates, normalize avatar/lengths, then UPDATE/INSERT
  // return mapped row with fallbacks for empty fields
}
```

Goal: single-row table with `getHero()` for reads and `upsertHero()` for PUTs.

---

### 2) Authenticated API: `/api/hero`
- Create `src/app/api/hero/route.js` with public GET and guarded PUT (`withApiAuthRequired`).
- Convert file uploads to data URLs; validate with zod.

```js
import { NextResponse } from "next/server";
import { z } from "zod";
import { auth0 } from "@/lib/auth0";
import { getHero, upsertHero } from "@/lib/db";
import image2uri, { extTypeMap } from "image2uri";
import fs from "fs";
import os from "os";
import path from "path";
import { randomUUID } from "crypto";

const heroSchema = z.object({
  avatar: z.string().trim().min(1).refine((v) => v.startsWith("data:"), "Avatar must be a data URL"),
  fullName: z.string().trim().min(2).max(200),
  shortDescription: z.string().trim().min(2).max(120),
  longDescription: z.string().trim().min(10).max(5000),
});

export async function GET() {
  const hero = await getHero();
  return NextResponse.json({ data: hero });
}

export const PUT = auth0.withApiAuthRequired(async (request) => {
  const session = await auth0.getSession();
  if (!session?.user?.email) {
    return NextResponse.json({ message: "You must be logged in to edit the hero section" }, { status: 401 });
  }

  const formData = await request.formData();
  const avatarFile = formData.get("avatarFile");
  const avatarFromForm = formData.get("avatar");
  const avatarDataUrl = await toDataUrl(avatarFile, avatarFromForm);

  const payload = heroSchema.parse({
    avatar: avatarDataUrl ?? "",
    fullName: formData.get("fullName") ?? "",
    shortDescription: formData.get("shortDescription") ?? "",
    longDescription: formData.get("longDescription") ?? "",
  });

  const hero = await upsertHero(payload);
  return NextResponse.json({ message: "Hero updated", data: hero }, { status: 200 });
});

async function toDataUrl(file, fallbackString) {
  const fallback = typeof fallbackString === "string" ? fallbackString.trim() : "";
  if (file && typeof file.arrayBuffer === "function") {
    const arrayBuffer = await file.arrayBuffer();
    const buffer = Buffer.from(arrayBuffer);
    const ext = path.extname(file.name || "") || ".bin";
    const mime = extTypeMap[ext] ?? file.type ?? "application/octet-stream";
    const tmp = path.join(os.tmpdir(), `${randomUUID()}${ext}`);
    fs.writeFileSync(tmp, buffer);
    try {
      const uri = await image2uri(tmp, { ext });
      return uri.startsWith("data:") ? uri : `data:${mime};base64,${uri}`;
    } finally {
      fs.rmSync(tmp, { force: true });
    }
  }
  return fallback;
}
```

Notes: GET is public for the homepage; PUT is guarded; avatar files become data URLs—no external storage needed.

---

### 3) Dashboard: protected Hero editor
- Gate with `useUser()`; show login prompt when anonymous; render form when authenticated (`src/app/dashboard/page.jsx`).

```jsx
"use client";
import { useEffect } from "react";
import { redirect } from "next/navigation";
import { useUser } from "@auth0/nextjs-auth0/client";
import { toast } from "sonner";
import HeroEditorForm from "@/components/hero-editor-form";

export default function DashboardPage() {
  const { user, error, isLoading } = useUser();
  useEffect(() => { if (error) toast.error(error.message); }, [error]);
  if (error) redirect("/auth/login");

  return (
    <div className="flex flex-col min-h-screen items-center bg-zinc-50 dark:bg-black">
      <h1 className="mt-8 text-4xl font-bold">Dashboard</h1>
      {isLoading && <p className="mt-4">Loading...</p>}
      {!isLoading && !user && <p className="mt-4 text-lg">Log in to update your portfolio content.</p>}
      {user && (
        <div className="mt-6 w-full max-w-5xl px-4 pb-10">
          <p className="mb-4 text-lg">Welcome to your dashboard, {user.nickname}!</p>
          <HeroEditorForm />
        </div>
      )}
    </div>
  );
}
```

---

### 4) Hero Editor UI (client form)
- `src/components/hero-editor-form.jsx` should:
  - Fetch current hero via `/api/hero` on mount; fall back to defaults on error.
  - Use zod + React Hook Form for validation and controlled fields.
  - Support avatar file upload → hidden avatar input (data URL) → FormData PUT to `/api/hero`.
  - Show preview image and toast messages on success/error.

```js
const onSubmit = async (values) => {
  const formData = new FormData();
  formData.append("avatar", values.avatar);
  formData.append("fullName", values.fullName);
  formData.append("shortDescription", values.shortDescription);
  formData.append("longDescription", values.longDescription);
  if (avatarFile) formData.append("avatarFile", avatarFile);

  const response = await fetch("/api/hero", { method: "PUT", body: formData });
  if (!response.ok) throw new Error((await response.json()).message || "Failed to update hero");
  const { data } = await response.json();
  form.reset({
    avatar: data.avatar,
    fullName: data.fullName,
    shortDescription: data.shortDescription,
    longDescription: data.longDescription,
  });
  toast.success("Hero section updated");
};
```

---

### 5) Homepage hero wired to DB
- Keep `src/components/my-hero-section.jsx` as a server component that calls `getHero()`.
- Render DB values when present; otherwise use `defaultHeroContent`; fall back to `HERO_PLACEHOLDER_AVATAR` when avatar is empty/transparent.

---

### 6) Deploy Prep on Vercel
- Import GitHub repo into Vercel (“Add New Project”).
- Environment Variables (Project Settings → Environment Variables): copy all `.env.local` values:
  - `NEON_DB_URL`
  - `AUTH0_SECRET`
  - `AUTH0_BASE_URL` → use your Vercel domain, e.g., `https://your-app.vercel.app`
  - `AUTH0_ISSUER_BASE_URL` → `https://<tenant>.us.auth0.com`
  - `AUTH0_CLIENT_ID`, `AUTH0_CLIENT_SECRET`
- Auth0 Application Settings (include Vercel domain and localhost):
  - Allowed Callback URLs: `https://your-app.vercel.app/auth/callback`, `http://localhost:3000/auth/callback`
  - Allowed Logout URLs: `https://your-app.vercel.app`, `http://localhost:3000`
  - Allowed Web Origins: `https://your-app.vercel.app`, `http://localhost:3000`
  - Allowed Login URLs: `https://your-app.vercel.app/auth/login`, `http://localhost:3000/auth/login`
- Redeploy from Vercel to pick up envs.
- Smoke test on live URL:
  - `/auth/login` redirects to Auth0 and back.
  - `/dashboard` loads; hero edits persist and reflect on homepage.

---

### 7) Local Dev vs Vercel URLs
- Local: `AUTH0_BASE_URL=http://localhost:3000`; callbacks/logouts use localhost.
- Prod: `AUTH0_BASE_URL=https://your-app.vercel.app`; ensure both localhost and prod URLs are in Auth0 settings for seamless switching.

---

## 🧪 Test Checklist
- Auth: `/auth/login`/`logout` work locally and on Vercel.
- Dashboard: logged-out users see login prompt; logged-in users see Hero editor.
- API: GET `/api/hero` returns data; PUT returns 200 + updated payload when authenticated, 401 otherwise.
- Avatar upload: file updates preview and saves as data URL; refresh shows persisted avatar.
- Homepage: hero reflects latest DB values; placeholders only when DB empty.
- Env: Vercel build succeeds; live site uses correct Auth0 callbacks.

---

## 🧯 Troubleshooting & Notes
- Homepage still shows old hero copy: The homepage is likely statically pre-rendered and baked in the old hero content. Make it dynamic (or revalidate immediately) so it fetches fresh data from Neon. Add at the top of `src/app/page.js`:

```js
export const revalidate = 0;
```

Redeploy, and the homepage will read the latest hero content on each request.
- 401 on PUT: confirm login and `auth0.withApiAuthRequired` wrapping.
- Avatar issues: ensure image/* uploads; data-URL conversion uses `image2uri`; keep files <1MB.
- Short description errors: zod caps at 120 chars; trim before submit.
- Auth0 console noise when logged out: The client may log harmless 401s. Ignore them or instantiate `Auth0Client` with `noContentProfileResponseWhenUnauthenticated: true` to return 204 instead of 401.
- Prod callback issues: Vercel domain must exactly match Auth0 Allowed URLs (https, no trailing slash).

---

## 📌 Next Steps (Optional Enhancements)
- Add versioning/history for hero edits (who changed what and when).
- Store data URL → upload to object storage (S3/R2) later for optimization.
- Add drafts vs published hero content.
- Add rate limiting or role checks (e.g., only allow certain emails).

---

## Final checklist before submission
- [ ] `/api/hero` GET/PUT implemented; PUT guarded by Auth0.
- [ ] `HeroEditorForm` loads existing data and saves changes (avatar + text).
- [ ] `/dashboard` gated; shows editor only when authenticated.
- [ ] Homepage hero reads DB and shows updated content.
- [ ] Vercel env vars set; Auth0 callback/logout/login/web origins updated for prod domain.
- [ ] Live URL tested for login + hero edit round-trip.

---

## Submission
- Link to your GitHub repo.
- Include screenshots:
  - `/dashboard` while logged in (hero editor visible).
  - Screenshots of modifying hero section from `/dashboard`.
  - Hero editor before/after saving; avatar upload preview.
  - Screenshot of Homepage showing updated hero content.
  - Screenshot of Vercel-deployed portfolio
  - Screenshots of working login steps on deployed portfolio.
