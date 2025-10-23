# Lab 1 – Next.js Portfolio: Project Setup & Homepage Foundation

Lab Length: ~180 minutes
Goal: Initialize a new Next.js app (JavaScript), add Tailwind CSS + shadcn/ui, set a global layout, and build a reusable NavBar plus a starter Hero/Projects preview area. Outcome: a styled homepage with working navigation ready to expand. 

## What You’ll Build
- A new Next.js app using JavaScript (not TypeScript) with App Router and Tailwind CSS preconfigured. 
- shadcn/ui installed with a base color (Choose the base color pallete of your choice, You can check the reference here: [Tailwind Colors](https://ui.shadcn.com/colors)). 
- A reusable NavBar component using shadcn’s Navigation Menu with links: Home, Resume, Projects, Login. 
- Modular page structure (components in src/). 
- Google Fonts via Next.js font integration. 

## Prerequisites
- Node.js installed. Verify with `node -v`. 
- VS Code or another editor. 

**Note:** We’ll run the dev server using `npm run dev` (NOT `npm start`), since `npm start` expects a <u>**production build**</u>. 

### Step 1: Create the Next.js App

1.	Open Terminal in your desired folder.
2.	Run the official CLI:

    ```bash
    npx create-next-app@latest
    ```
    This uses `npx` to execute the latest Next.js app generator. 

3. Answer the prompts (choose as below):
    - Project name: e.g., iman-portfolio (your choice). 
    - Use the recommended default? No (we will customize to avoid TypeScript). 
    - TypeScript? No. 
    - ESLint? Yes. 
    - React Compiler? Yes. 
    - Tailwind CSS? Yes. 
    - src/ directory? Yes. 
    - App Router? Yes. 
    - Turbopack? Yes (recommended). ([What is Turbopack?!](https://nextjs.org/docs/app/api-reference/turbopack))
    - Customize import alias? No (keep @ default). 

    Why Tailwind here? Next.js can generate Tailwind setup automatically so you’re ready to style immediately. 

    After creation, notice files like `next.config.mjs`, `package.json`, `postcss.config.js`, `tailwind.config.js`, `public/`, `src/`, `.gitignore`, etc. 

### Step 2: Run the Dev Server (and fix the common gotcha)

1.	cd into the new project folder.
2.	Start the server in development mode:

    ```bash
    npm run dev
    ```

Open http://localhost:3000 to see the Next.js starter. If you accidentally try `npm start` first, you’ll see an error like “_could not find a production build_”—that’s expected until you run npm run build. Use `npm run dev` for this lab. 

### Step 3: Install and Configure shadcn/ui

We’ll use shadcn/ui for high-quality headless components styled with Tailwind.
1.	Follow the shadcn installer (run their init script per [docs](https://ui.shadcn.com/docs/installation/next)), then choose a desired base color.
2.	The installer will wire up global CSS and add a lib/components structure for you. 
3.	We’ll add the following components now: navigation-menu, card, button, and typography. (Toast and skeleton are also handy later.)
- You can add multiple components as below:

    ```bash
    npx shadcn@latest add [component]
    ```

- You can always add more later (e.g., dialog, sheet, etc.). 

### Step 4: Global Layout & Fonts

Next.js App Router uses a root `layout.js` where you can set *metadata*, *fonts*, and wrap all pages.
- Confirm Google Fonts integration (e.g., using next/font/google) and set base metadata like title and description. 
- Ensure children is rendered inside <body> so pages appear correctly. 

### Step 5: Build a Reusable NavBar Component

We’ll move nav code into a dedicated component for cleanliness and reuse.

1.	Create the component file (e.g., `src/components/MyNavBar.js`). 
2.	Implement a Navigation Menu using shadcn’s exports (e.g., `NavigationMenu`, `NavigationMenuList`, `NavigationMenuItem`, `NavigationMenuLink`). Be sure to import all required subcomponents. (check out the [docs](https://ui.shadcn.com/docs/components/navigation-menu)) 
3.	Add links:
    - Home → `/`
    - Projects → `/projects`
    - Resume (placeholder for now; you can later point to a PDF) → e.g., `/resume` 
    - Login → `/login `
4.	Gotcha: making links clickable!
- If you wrap with shadcn’s asChild, ensure the child is a `<Link>` that provides an href. With Next.js, you can use `next/link` package or a plain `<a href="/">.`
5. Feel free to customize your navigation bar to your liking.
6.	Use the NavBar on your homepage by importing and placing <MyNavBar /> in `src/app/page.js`. 

Modularizing keeps `page.js` minimal and future changes isolated to `<MyNavBar />`. 

Sticky Nav & Page Flow
- Make the navbar sticky: add `sticky top-0`.
- Keep the main content starting at the top (justify-start) unless you explicitly desire centered layouts. 

---

### Step 6: Build the Hero Component

Create a dedicated Hero component and render it on the homepage. 
1.	Create src/components/MyHeroSection.jsx. 
2.	Scaffold a minimal component:

```bash
export default function MyHero() {
    return (<div><div/>)
    }
```

Start simple, then grow. 

3.	Import essentials:
- `Card` from `shadcn/ui`. 
- `Image` from `next/image`. 
4.	Place your image in `/public` (e.g., /public/profile.jpg). Reference with `src="/profile.jpg"` in `<Image>` to avoid Invalid URL errors. 
5.	Compose the card with header/content areas and text beside the image. Add an h1 for the title. 
6.	Typography (optional): try shadcn Typography or import a Google Font (e.g., Roboto Mono) with next/font/google. 
7.	Mount on the homepage by importing and rendering `<MyHero/>` beneath the `NavBar`. 

Layout tips / fixes you may need
- If the image and text sit side‑by‑side unexpectedly, ensure the parent is flex flex-col or adjust flex-row/flex-col deliberately. 
- Remove inherited center‑alignment on the page wrapper if it forces everything to the middle. 
- Page scaffolding like className="flex flex-col w-full" plus h-screen where appropriate keeps flow predictable. 
- Make the nav persist with sticky top-0 (and background/z‑index as needed).
- Equal card heights: put items-stretch on the flex/grid container and h-full (or self-stretch) on each card. 
- Prefer top‑alignment by using justify-start instead of centering if you want content to start from the top. 

### Step 7: Build the Projects Preview Card Component (hard‑coded first)

Create a simple Projects Preview Card component that renders a fixed number of preview cards; we’ll make it dynamic later. 
1.	Create src/components/project-preview-card.jsx. 
2.	Scaffold with a count prop:

```bash
export default function ProjectPreviewCard({ count = 3 }) {
  return <div/>;
}
```

This lets you choose how many projects to display. 

3.	Prepare temp data (inside the component for now):
    - Each project should consist of:
        - Title: string
        - Description: string
        - Thumbnail Image: URL Address: string
        - Link: an address navigating to the project

```js
const projects = [
  {
    title: "Project One",
    desc: "Short blurb.",
    img: "https://placehold.co/300.png",
    link: "#"
    },

    ...

];
```

Include title, description, and a placeholder image. 

4. Render with shadcn/ui `Card` + `Button` + `Skeleton` as a placeholder:
- Import `Card` and use `Skeleton` for each project's thumbnail.
- Use `Button` to render the link to the project.
5. Layout
- Make the layout responsive using `grid` or `flex` (Your choice)
6. Mount on the homepage under the Hero:
- Import `ProjectPreviewCard` in `src/app/page.jsx` and render `<ProjectPreviewCard count={3} />`. 
- If it doesn’t show up, inspect the DOM and verify the Hero’s height or a parent container isn’t obscuring it. 

Checklist (Projects)
- Images are in `/public` or a configured remote domain if your images are coming from a external sources (check [this](https://nextjs.org/docs/pages/api-reference/components/image#remote-images) section for more detail on `remotePatterns` configuration). 
- Cards share consistent padding/radius/shadows.
- Titles/descriptions use chosen typography utilities. 
- Containers stretch evenly (items-stretch + h-full). 

### Troubleshooting (Common Errors You May See)
- “could not find a production build” after npm start → Use npm run dev during development; npm start is for running a built app. 
- Nav items not appearing → Ensure you imported all needed shadcn Navigation Menu subcomponents and used the right JSX structure: NavigationMenu > NavigationMenuList > NavigationMenuItem > NavigationMenuLink. 
- Link not clickable → When using asChild, provide an element that supports href (e.g., Next Link or an `<a>`). 
- Font sizing mismatch → Confirm consistent classes per item, and avoid mixing stray elements outside NavigationMenuList. 

---

### You’re done!

You now have a clean, modular Next.js foundation with Tailwind + shadcn/ui, a global layout, and a reusable NavBar, your Hero section, and Project preview cards ready for future labs (dynamic content, data fetching, auth, etc.). 


### Final checklist before submission

- [ ] Sticky Navigation Bar
    - [ ] Mandatory Links: Home, Projects, Login
- [ ] Responsive Hero Section
    - [ ] Mandatory items: Usage of Card, Image, Your Name, Description.
- [ ] Responsive Project Preview Card
    - [ ] Mandatory Details:
        - [ ] Customizable Number of Projects Rendered
        - [ ] Same Sized Project Preview Cards
- [ ] Consistant Stylings and Clean UI Across All Components using only TailwindCSS



## Submission

- Please provide full page (beginning to the end) screenshot of your lab 1 rendered page, in both Mobile and Desktop view.
- Commit and Push all the changes made for this Lab in a new Github Repo; and provide your Repo's link in your submission.