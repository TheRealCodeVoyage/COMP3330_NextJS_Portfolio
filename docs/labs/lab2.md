# Lab 2 – Next.js Portfolio: Projects Page & Dynamic Routing

Projects Index, Dynamic Detail, App Router API, and “New Project” Form

This lab continues your portfolio build with Next.js App Router. You’ll render a projects list, support dynamic detail pages, move data behind API routes, and ship a /projects/new form with validation that POSTs to your API.

---

## 🎯 Lab 2 Objectives

1.	App Router data flow: Fetch projects from /app/api/projects (Shifting from static hardcoded array).
2.	Routing:
- /projects lists every project.
- /projects/[slug] shows one project based on a URL-friendly slug derived from its title.
3.	UI/UX: Keep a sticky, always-on-top navbar (z-index fix so cards don’t overlap).
4.	Create form: Build /projects/new using react-hook-form, zod, and shadcn/ui.
5.	Server POST: Handle POST /api/projects/new with formData, prepare for DB later.
6.	Keywords field: Add/remove keyword “chips” in the form (using badges from shadcn/ui).

---

## 📚 What You’ll Learn

- App Router file conventions: app/(routes)/**/page.jsx and app/api/**/route.js.
- Dynamic segments and params in server components.
- Fetching data from API routes in server components.
- Client-side form validation with zod + react-hook-form.
- Submitting FormData to an App Router API endpoint.
- Managing tag-like inputs (keywords) with local UI state.

---

## ✅ End Result (By the End of Lab 2)

- /projects — grid of cards loaded from GET /api/projects.
- /projects/[slug] — detail page for a single project (slug from title).
- /projects/new — validated form for creating a new project (title, description, image URL, link, keywords) that POSTs to /api/projects/new.
- Sticky Navbar that stays above content (fixed stacking context).

---

## 🧰 Prerequisites

- Next.js App Router project (JS is fine).
- Tailwind CSS configured.
- shadcn/ui installed with: Button, Card, Form, Input, Form, Badge, Skeleton,
- Packages:

```bash
npm i react-hook-form @hookform/resolvers zod
```

- Add this to .env.local for server-side fetching during local dev:

```plaintext
NEXT_PUBLIC_BASE_URL=http://localhost:3000
```

---

## 🗂️ Final Folder Structure (Recap)

>```bash
>src/
>├── app/
>│   ├── api/
>│   │   └── projects/
>│   │       ├── new/
>│   │       │   └── route.js    # POST /api/projects/new -> create
>│   │       └── route.js        # GET /api/projects -> returns list
>│   ├── layout.js               # navbar lives here (sticky + z-50)
>│   ├── page.js
>│   └── projects/
>│       ├── [slug]/
>│       │   └── page.jsx        # /projects/:slug (detail)
>│       ├── new/
>│       │   └── page.jsx        # /projects/new (form)
>│       └── page.jsx            # /projects (index)
>└── components/
>    ├── my-hero-section.jsx
>    ├── my-nav.jsx
>    ├── project-preview-card.jsx
>    └── ui/                     # shadcn ui components
>```

---

## 🧭 Step-by-Step Instructions

### 1) Move the Navbar into the App Layout (and fix overlap)

You may notices when navigating to other pages, navbar is not appearing anymore, that's becuase we have only imported our Navbar in to our home page and not the root layout. now we will move the navbar to the **App Root Layout** in: File: `src/app/layout.js` above the future children (nested pages/component that may render there.)

```js
export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>
        <MyNavBar />
        {children}
      </body>
    </html>
  );
}
```

**Note:**
Your Navbar needs a z-index attribute to avoid any other component cover it.
`z-50` prevents project cards (which may scale on hover) from visually covering the nav.

---

### 2) Create the Projects API (GET)

In `src/app/api`, which we might consider it to be our backend, we can define our API Endpoints! Creating a `/projects/` directory in there, directly adds to the API endpoint handle; this is because Next.js App Router works in a file-based manner.
In order to register your HTTP Method handler such as `GET`, `POST`, `PUT`, ..., we need to add a `route.js` at the end of this API route/path.

In this File: `src/app/api/projects/route.js` we will define our `GET` request, which returned back out Projects data. For now we will write them hard-coded in an array but later we will connect out DB to handle the data.

```js
// GET /api/projects
export async function GET() {
  const projects = [
    {
      title: "Conway's Game of Life",
      description: "Cellular automaton visualizer.",
      image: "/images/placeholder-300x300.png",
      link: "https://example.com/game-of-life",
      keywords: ["algorithms", "simulation"]
    },
    // ...add more seed items
  ];

  return Response.json({ projects });
}
```

This centralizes your data. You’ll swap it with a DB later.

You can test it by running a curl command in a seperate terminal or simply navigate to the address by your browser

```bash
curl http://localhost:3000/api/projects
```

---

### 3) Build the Projects Index Page

In this page we will show all saved projects. In `src/app/` any new directories created such as `/projects/` could be considered as our frontend path.

>A good mental model for the App Router:
>
>- Folders under `app/api/**/route.js` define HTTP endpoints (**your “backend” surface**). Files there export handlers like `GET`, `POST`, etc., and they run on the server—perfect for DB access, auth, etc.
>- Other folders under `app/` (like `app/projects`) define UI routes. Each `page.js` (or `page.jsx`) becomes a **frontend entry point**, and you can nest layouts, loading states, etc.

```js
import Link from "next/link";
import Image from "next/image";
import { Card } from "@/components/ui/card";
import { Button } from "@/components/ui/button";

const createSlug = (str) =>
  str.toLowerCase().trim()
    .replace(/\s+/g, '-') // Replace spaces with hyphens
    .replace(/[^\w-]+/g, '') // Remove non-word characters
    .replace(/--+/g, '-'); // Replace multiple hyphens with a single hyphen
}

export default async function ProjectsPage() {
  const res = await fetch(`${process.env.NEXT_PUBLIC_BASE_URL}/api/projects`, { cache: "no-store" });
  const { projects } = await res.json();

  return (
    <div className="flex items-center justify-center">
    {projects.map((p) => {
        const slug = createSlug(p.title);
        return (
        <Card key={slug} className="group hover:scale-105 transition-transform">
            <h3>{p.title}</h3>
            <div className="space-y-3">
            <Image
                src={p.image}
                alt={p.title}
                width={300}
                height={300}
                className="w-full h-48 object-cover rounded-md"
            />
            <p className="text-sm text-muted-foreground line-clamp-2">{p.description}</p>
            <div className="flex gap-2">
                <Button asChild size="sm" variant="secondary">
                <a href={p.link} target="_blank" rel="noreferrer">Open</a>
                </Button>
                <Button asChild size="sm">
                <Link href={`/projects/${slug}`}>Details</Link>
                </Button>
            </div>
            </div>
        </Card>
        );
    })}
    </div>
  );
}
```

Image paths: put assets under public/ and reference them as /images/... (no public prefix). Next.js recognizes images in `public/` directory; when a request to `/image/<an image file name>` is recevied, Next.js automatically searches in the `public/` directory for it.

You may have noticed `createSlug()` function in the code above. this function helps us slugify the title of each project and create a unique address for each project. Later, we will define each proejct's page.

---

### 4) Dynamic Detail Project Page: /projects/[slug]

>**Reference: [Dynamic Route Segments](https://nextjs.org/docs/app/api-reference/file-conventions/dynamic-routes)**

Now, we wan to create a detail page for each of our projects. we want to be able to navigate directly to each project page via their unique link. this is where slug comes to play. we can easily search out project for their slugified title.

In this page we want to show:
- Project's Title
- Project's Description
- Project's Link as a Button
- Project's Thumbnail
- Project's Tags/Keywords in form of Chips


By creating a directory wrapped with squared brackets such as `/[slug]/` we are basically creating a dynamic route where as of the page's parameters, `slug` will be accessable in said page. As Next.js structure we need to have a `page.js` (or `page.jsx`) at the end of our new API path.

#### ⚙️ Modularize the repeating function!
Since we are going to use `createSlug()` function again and it's not just limited to one spesific component, it's better to modularize it and integrate it into `src/lib/utils.js` and import it where ever we need to use it rather than redefining the function.



Now we can proceed to write our first Dynamic Route page: File: src/app/projects/[slug]/page.jsx

```js
import { createSlug } from "@/lib/utils"
// import rest of components needed.

export default async function ProjectDetailPage({ params }) {
    const { slug } = await params
    return <div>My Post's sluq: {slug}</div>
}
```

This is just an example of how to extract `slug` out of `params`. use this example to form your detail project page fully. 

The `slug` is derived from the title (no need to store it separately yet).

---

### 5) New Project Form (Client Component)

- Using [shadcn/ui form component](https://ui.shadcn.com/docs/components/form), we need to build this page to be able to create new project.
- The Form should `POST` to Route: `api/projects/new`
- New Project Form Page need to be accessable from `http:/localhost:3000/projects/new`
- Create a directory and the `page.jsx` file to start building this Form Page.
- We will use `useForm` hook from `react-hook-form` we have installed eariler.
- When creating a `form` using `useForm` we will employ `zodResolver` from `@hookform/resolvers/zod` to check and validate the data we are receiving agasint the _Schema_ (zod Object) we define.


**Below is the boiletplate of what you need in regard to useForm, Validatoin, Schema, onSubmit Function, [FormData](https://developer.mozilla.org/en-US/docs/Web/API/FormData), ...** Make sure you complete the code.

```js
"use client";

import { z } from "zod"
import { zodResolver } from "@hookform/resolvers/zod"
import { useForm } from "react-hook-form"
import { useState } from "react"
// Import rest of the components needed from shadcn/ui

const newProjectSchema = z.object({
    title: z.string().min(2, { message: "Your title is too short", }).max(200),
    ... // define the rest
})

export default function NewPage() {

    const form = useForm({
        resolver: zodResolver(newProjectSchema),
        defaultValues: {
            title: "Write your project title here...",
            description: "Write your project description here...",
            img: "https://placehold.co/300.png",
            link: "https://your-project-link.com",
            keywords: [],
        },
    })

    function onSubmit(values) {
        const formData = new FormData()
        formData.append('title', values.title)
        ... // complete the rest

        fetch('/api/projects/new', {
            method: 'POST',
            body: formData,
        })
        ... // handle the response retunred and catch the possible error


        // TODO: in future we will write the data to DB 
    }

    return (
        <Form {...form}>
            <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-2 m-4">
                <FormField
                    control={form.control}
                    name="title"
                    render={({ field }) => (
                        <FormItem>
                            <FormLabel>Title</FormLabel>
                            <FormControl>
                                <Input placeholder="A title of your project" {...field} />
                            </FormControl>
                            <FormDescription>
                                This is the title of your project.
                            </FormDescription>
                            <FormMessage />
                        </FormItem>
                    )}
                />
                <FormField
                    control={form.control}
                    name="description"
                    render={({ field }) => (
                        <FormItem>
                            <FormLabel>Description</FormLabel>
                            <FormControl>
                                <Input placeholder="A brief description of your project" {...field} />
                            </FormControl>
                            <FormDescription>
                                This is a brief description of your project.
                            </FormDescription>
                            <FormMessage />
                        </FormItem>
                    )}
                />
                <FormField
                    control={form.control}
                    name="img"
                    render={({ field }) => (
                        <FormItem>
                            <FormLabel>Image URL</FormLabel>
                            <FormControl>
                                <Input placeholder="https://your-image-link.com/image.png" {...field} />
                            </FormControl>
                            <FormDescription>
                                This is the image URL of your project.
                            </FormDescription>
                            <FormMessage />
                        </FormItem>
                    )}
                />
                <FormField
                    control={form.control}
                    name="link"
                    render={({ field }) => (
                        <FormItem>
                            <FormLabel>Project Link</FormLabel>
                            <FormControl>
                                <Input placeholder="https://your-project-link.com" {...field} />
                            </FormControl>
                            <FormDescription>
                                This is the link to your project.
                            </FormDescription>
                            <FormMessage />
                        </FormItem>
                    )}
                />
                <FormField
                    control={form.control}
                    name="keywords"
                    render={({ field }) => {
                        const currentKeywords = field.value ?? [];

                        const handleAddKeyword = () => {
                            const value = draftKeyword.trim();
                            if (!value || currentKeywords.includes(value)) return;

                            const updated = [...currentKeywords, value];
                            field.onChange(updated);
                            setDraftKeyword("");
                        };

                        const handleRemoveKeyword = (keyword) => {
                            const updated = currentKeywords.filter((k) => k !== keyword);
                            field.onChange(updated);
                        };

                        const handleKeyDown = (event) => {
                            if (event.key === "Enter") {
                                event.preventDefault();
                                handleAddKeyword();
                            }
                        };
                        return (
                            <FormItem className="flex">
                                <div className="flex flex-col gap-2 flex-1">
                                    <FormLabel>Keywords</FormLabel>
                                    <FormControl>
                                        <div className="flex gap-2">
                                            <Input
                                                value={draftKeyword}
                                                onChange={(e) => setDraftKeyword(e.target.value)}
                                                onKeyDown={handleKeyDown}
                                                placeholder="Add a keyword and press Enter"
                                            />
                                            <Button type="button" onClick={handleAddKeyword}>
                                                Add
                                            </Button>
                                        </div>
                                    </FormControl>
                                    <FormDescription>
                                        Tag your project so it is easier to filter later.
                                    </FormDescription>
                                </div>
                                <div className="flex flex-1 flex-wrap gap-2 pt-6">
                                    {currentKeywords.map((keyword) => (
                                        <Badge
                                            key={keyword}
                                            variant="outline"
                                            className="flex items-center gap-1 bg-stone-600 text-stone-200"
                                        >
                                            {keyword}
                                            <button
                                                type="button"
                                                className="ml-1 text-xs"
                                                onClick={() => handleRemoveKeyword(keyword)}
                                                aria-label={`Remove ${keyword}`}
                                            >
                                                ×
                                            </button>
                                        </Badge>
                                    ))}
                                </div>
                                <FormMessage />
                            </FormItem>
                        );
                    }}
                />
                <Button type="submit" className="mt-2">Submit</Button>
            </form>
        </Form >
    )
}
```

---

### 6) Handle the Form POST on the Server


Since we are posting to `/api/projects/new`, we need to register this route by adding a `route.js` file at the of this path. `src/app/api/projects/new/route.js` will take care of our `POST` request at the server level.

```js
export async function POST(req) {
  try {
    const formData = await req.formData();
    const title = formData.get("title");
    ... // complete the rest 

    // FUTURE CONCERNS - you can ignore them now
    // TODO: (recommended) validate here again with Zod
    // TODO: persist to DB (Prisma/Drizzle/etc.)
    // TODO: revalidatePath("/projects") after write (if using Next cache)

    console.log({project: { title, description, img, link, keywords }})

    return Response.json({ ok: true, project: { title, description, img, link, keywords } }, { status: 201 });
  } catch (err) {
    console.error(err);
    return Response.json({ ok: false, error: "Invalid payload" }, { status: 400 });
  }
}
```

---

## 🧪 Test Checklist
- Navbar shows on every page and never hides behind cards (thanks to z-50).
- GET /api/projects returns an array of projects.
- /projects fetches and renders cards for all items.
- Clicking Details navigates to /projects/[slug] and shows the right project.
- /projects/new loads with default values.
- Adding a keyword via Enter or the Add button appends a chip; clicking the chip removes it.
- Submitting the form issues a POST to /api/projects/new; on success, server logs the request.

---

### 🧯 Troubleshooting & Notes
- Images 404 / invalid URL:
    - Local images go under /public/** and are referenced as /images/….
    - Remote images require next.config.js images.domains allowlist.
- zod .url() deprecation warning:
    - Some editors show a stale warning; z.string().url() continues to work for client-side validation.
- Slug mismatches:
    - If detail pages 404, confirm createSlug(title) is identical wherever it’s used (list and detail).
- Avoid legacy routes:
    - Ensure all data flows through App Router APIs; and not Page Router. When reading docs on next.js docs, on the top left corner, it let you choose which type of router you would like to see.

---

📌 Next Steps (Optional Enhancements)
- Add server-side Zod validation to POST /api/projects/new.
- Persist to a real DB and call revalidatePath("/projects") after writes.
- Show success/error toasts on submit.
- Add tag-based filtering on /projects.
- Generate static params for known slugs if you switch to SSG.

---

### You’re set!
Lab 2 now delivers a complete, API-backed projects section with listing, details, and a creation form—cleanly structured for future DB integration and UI polish.

### Final checklist before submission

- [ ] `/Projects/*` routes works fine.
    - [ ] Navigating to `/projetcs/` shows full list of projects responsively (Mobile and Desktop)
    - [ ] Navigating to `/projects/[slug]` shows the correct project responsively
    - [ ] Navigating to `/projects/new`:
        - [ ]  renders form correctly
        - [ ]  keywords form works correctly
        - [ ]  when submitting the form, server receives it and logs it in the terminal. it's ready for DB integration.
- [ ] Navbar shows on every page and never hides behind cards.


## Submission

- Please provide full page (beginning to the end) screenshot of your lab 2 rendered page, in both Mobile and Desktop view.
- Commit and Push all the changes made for this Lab in a new Github Repo; and provide your Repo's link in your submission.