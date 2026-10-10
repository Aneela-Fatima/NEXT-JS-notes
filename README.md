# Next.js 15 Revision Notes

**App Router \| React Server Components \| Rendering \| Data Fetching \|
APIs \| Server Actions \| Authentication**

> **Scope:** These notes focus on Next.js 15 and the App Router. Some
> behavior differs in later Next.js releases, so check the
> version-specific documentation when working in a newer project.

## Table of Contents

1.  [Introduction and Overview](#1-introduction-and-overview)
2.  [Prerequisites and Setup](#2-prerequisites-and-setup)
3.  [Project Structure](#3-project-structure)
4.  [Routing Fundamentals](#4-routing-fundamentals)
5.  [Dynamic and Advanced Routing](#5-dynamic-and-advanced-routing)
6.  [Navigation and URL Data](#6-navigation-and-url-data)
7.  [Server and Client Components](#7-server-and-client-components)
8.  [Rendering Strategies](#8-rendering-strategies)
9.  [Data Fetching](#9-data-fetching)
10. [Caching and Revalidation](#10-caching-and-revalidation)
11. [Loading, Errors, and Streaming](#11-loading-errors-and-streaming)
12. [Route Handlers and API
    Endpoints](#12-route-handlers-and-api-endpoints)
13. [Server Actions and Mutations](#13-server-actions-and-mutations)
14. [Authentication, Authorization, and
    Middleware](#14-authentication-authorization-and-middleware)
15. [Metadata, SEO, Images, Fonts, and
    Scripts](#15-metadata-seo-images-fonts-and-scripts)
16. [Environment Variables and
    Security](#16-environment-variables-and-security)
17. [Forms, Validation, and File
    Uploads](#17-forms-validation-and-file-uploads)
18. [State Management and Common
    Hooks](#18-state-management-and-common-hooks)
19. [Database and Full-Stack
    Architecture](#19-database-and-full-stack-architecture)
20. [Performance and Deployment](#20-performance-and-deployment)
21. [Testing and Debugging](#21-testing-and-debugging)
22. [Common Mistakes](#22-common-mistakes)
23. [Quick Revision Cheat Sheet](#23-quick-revision-cheat-sheet)
24. [Official References](#24-official-references)

------------------------------------------------------------------------

## 1. Introduction and Overview

### What is Next.js?

Next.js is a React framework for building web applications. React
provides the component model for user interfaces; Next.js adds
conventions and tools for routing, server rendering, data access,
caching, metadata, optimization, and backend endpoints.

### Why use it?

-   **Routing:** routes are created from files and folders.
-   **Rendering choices:** static, request-time, and client-side
    rendering can be combined.
-   **Server Components:** render UI and access trusted server-side
    resources without sending all component code to the browser.
-   **Data fetching:** use `async`/`await` in Server Components.
-   **Mutations:** Server Actions can handle form submissions and data
    changes.
-   **API endpoints:** Route Handlers expose HTTP endpoints.
-   **Optimization:** built-in support for images, fonts, scripts, code
    splitting, and navigation prefetching.
-   **SEO:** metadata APIs help provide page titles, descriptions, Open
    Graph data, and related tags.
-   **TypeScript:** first-class support for typed components and route
    data.

### Important vocabulary

-   **React:** UI library.
-   **Next.js:** framework built around React.
-   **App Router:** routing system based on the `app/` directory.
-   **Server Component:** component rendered on the server; default in
    the App Router.
-   **Client Component:** component that can use state, effects, event
    handlers, and browser APIs.
-   **Route Handler:** server-side HTTP handler defined in `route.ts` or
    `route.js`.
-   **Server Action / Server Function:** async server function callable
    through supported React/Next.js mechanisms.
-   **Hydration:** React attaches client-side behavior to
    server-rendered HTML.
-   **RSC payload:** serialized information React uses to reconcile
    Server and Client Components during navigation.

### Prerequisites

-   HTML and CSS
-   Modern JavaScript: modules, destructuring, promises,
    `async`/`await`, array methods
-   React components, props, state, effects, and hooks
-   Basic HTTP concepts: requests, responses, methods, status codes,
    headers, cookies
-   Basic database and authentication concepts are useful for full-stack
    projects

------------------------------------------------------------------------

## 2. Prerequisites and Setup

### Create a project

``` bash
npx create-next-app@15 my-next-app
cd my-next-app
npm run dev
```

The setup wizard can configure TypeScript, ESLint, Tailwind CSS, a
`src/` directory, and the App Router. For an existing project, use its
installed Next.js version rather than assuming it is version 15.

Common commands:

``` bash
npm run dev       # Development server
npm run build     # Production build
npm run start     # Run the production build
npm run lint      # Only if this script is configured
```

Typical local URL: `http://localhost:3000`.

### TypeScript or JavaScript?

Next.js supports both. TypeScript adds static type checking and is
especially helpful for route parameters, form data, API responses, and
database models.

------------------------------------------------------------------------

## 3. Project Structure

A typical App Router project:

``` text
my-next-app/
├── public/                 # Static assets served from /
├── src/
│   └── app/
│       ├── layout.tsx      # Root layout
│       ├── page.tsx        # Home route: /
│       ├── globals.css
│       ├── about/
│       │   └── page.tsx    # /about
│       ├── products/
│       │   ├── page.tsx    # /products
│       │   └── [id]/
│       │       └── page.tsx
│       └── api/
│           └── users/
│               └── route.ts
├── components/             # Reusable UI (project convention)
├── lib/                    # Database clients, helpers, utilities
├── types/                  # Shared TypeScript types (optional)
├── next.config.ts
├── package.json
└── tsconfig.json
```

`src/` is optional. The `app/` directory may be at the project root or
inside `src/`.

### Public assets

Files in `public/` can be referenced from the site root, for example
`/logo.png`. Do not put secrets in `public/`.

### Root layout

The root `app/layout.tsx` is required in the App Router and must define
`<html>` and `<body>`.

``` tsx
export default function RootLayout({
  children,
}: Readonly<{ children: React.ReactNode }>) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

------------------------------------------------------------------------

## 4. Routing Fundamentals

### File-system routing

Folders represent URL segments. A `page.tsx` file makes that segment
publicly accessible as a page.

  -----------------------------------------------------------------------
  File                                Purpose
  ----------------------------------- -----------------------------------
  `page.tsx`                          Page UI for a route

  `layout.tsx`                        Shared UI that persists across
                                      child navigation

  `template.tsx`                      Similar to layout, but remounts
                                      when its segment changes

  `loading.tsx`                       Loading UI integrated with Suspense

  `error.tsx`                         Error boundary UI for a route
                                      segment; must be a Client Component

  `global-error.tsx`                  Root-level error UI; must provide
                                      its own `<html>` and `<body>`

  `not-found.tsx`                     Not-found UI

  `route.ts`                          HTTP Route Handler, not a page

  `default.tsx`                       Fallback UI for an unmatched
                                      parallel route slot
  -----------------------------------------------------------------------

Other useful conventions include `metadata` / `generateMetadata`,
`sitemap.ts`, `robots.ts`, and `opengraph-image.tsx`.

### Example

``` text
app/
├── page.tsx              -> /
├── about/page.tsx        -> /about
└── contact/page.tsx      -> /contact
```

### Layout vs. template

-   **Layout:** shared shell such as navigation and sidebar; preserved
    across navigation within the segment.
-   **Template:** creates a new instance when the segment changes;
    useful when remounting or rerunning entry animations is intended.
-   Neither should be treated as a replacement for ordinary component
    state management.

### Route groups

A folder wrapped in parentheses organizes routes without adding that
folder name to the URL.

``` text
app/
├── (marketing)/about/page.tsx  -> /about
└── (auth)/login/page.tsx       -> /login
```

Route groups can organize layouts and teams, but avoid defining
conflicting routes that resolve to the same URL.

------------------------------------------------------------------------

## 5. Dynamic and Advanced Routing

### Dynamic segment: `[id]`

``` text
app/products/[id]/page.tsx
```

Matches `/products/42`. In Next.js 15, `params` is asynchronous in App
Router pages and layouts.

``` tsx
type Props = {
  params: Promise<{ id: string }>;
};

export default async function ProductPage({ params }: Props) {
  const { id } = await params;
  return <h1>Product {id}</h1>;
}
```

Dynamic segments are strings. Validate and convert values before using
them as numeric IDs.

### Nested dynamic segments

``` text
app/products/[productId]/reviews/[reviewId]/page.tsx
```

For `/products/42/reviews/7`, params contains both `productId` and
`reviewId`.

### Catch-all segment: `[...slug]`

``` text
app/docs/[...slug]/page.tsx
```

Matches one or more segments, such as `/docs/setup/linux`. The `slug`
value is an array of strings. It does **not** match `/docs` without a
segment.

### Optional catch-all: `[[...slug]]`

Matches both `/docs` and nested paths such as `/docs/setup/linux`. Here,
`slug` can be `undefined` at the base route.

### Parallel routes: `@slot`

Parallel routes let a layout render multiple named slots at once, for
example analytics and activity panels in a dashboard.

``` text
app/dashboard/
├── layout.tsx
├── page.tsx
├── @analytics/page.tsx
└── @activity/page.tsx
```

The layout receives slots as props. Parallel routes are useful for
independently managed dashboard areas and complex loading states.

### Intercepting routes

Special folder names such as `(.)`, `(..)`, `(..)(..)`, and `(...)` let
a route render in the current layout context. A common pattern is
opening a photo or product in a modal while the URL changes, while
direct navigation to that URL still displays a full page. The notation
describes route-segment relationships, not ordinary relative filesystem
imports.

### Private folders: `_folder`

An underscore-prefixed folder is excluded from routing and can hold
internal components or helpers. It is a convention, not a security
boundary.

------------------------------------------------------------------------

## 6. Navigation and URL Data

### Use `Link` for navigation

``` tsx
import Link from "next/link";

<Link href="/products">Browse products</Link>
```

`Link` supports client-side navigation and can prefetch routes when
appropriate. Use a normal anchor for external websites or downloads
where suitable.

### Programmatic navigation

In Client Components:

``` tsx
"use client";

import { useRouter } from "next/navigation";

const router = useRouter();
// router.push("/dashboard");
// router.replace("/login");
// router.back();
// router.refresh();
```

Do not call `useRouter` from `next/router` in the App Router. Use
`next/navigation`.

### Read URL values

-   `params`: dynamic path segments such as `/products/42`.
-   `searchParams`: query string values such as `/products?sort=price`.
-   `usePathname()`: current pathname in a Client Component.
-   `useSearchParams()`: read query string in a Client Component.
-   `useParams()`: read dynamic params in a Client Component.

In Next.js 15 page props such as `params` and `searchParams` are
promises:

``` tsx
type Props = {
  searchParams: Promise<{ q?: string; page?: string }>;
};

export default async function SearchPage({ searchParams }: Props) {
  const { q = "", page = "1" } = await searchParams;
  return <p>Searching for {q}, page {page}</p>;
}
```

Validate query-string values because they are user-controlled input.

------------------------------------------------------------------------

## 7. Server and Client Components

### Server Components are the default

In the App Router, pages and layouts are Server Components unless a
client boundary is introduced.

**Good uses:** - Fetching data on the server - Reading from a database
through a server-only data layer - Keeping API keys and private logic
out of browser bundles - Rendering mostly static content - Reducing
JavaScript sent to the browser

### Client Components

Add `"use client"` at the top of a module to mark a client entry point.
Use them when you need: - `useState`, `useEffect`, and other client
hooks - Event handlers such as `onClick` or `onChange` - Browser APIs
such as `window` or `localStorage` - Interactive widgets, client-side
state, or effects

``` tsx
"use client";

import { useState } from "react";

export default function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>Count: {count}</button>;
}
```

### Important clarifications

-   `"use client"` does **not** mean the component is never rendered on
    the server. Client Components can be prerendered to HTML and then
    hydrated in the browser.
-   The directive creates a client module boundary. Modules imported
    below that boundary can become part of the client bundle.
-   Keep the boundary as low and narrow as practical to avoid shipping
    unnecessary JavaScript.
-   Server Components cannot use client hooks or event handlers.
-   Props passed from Server Components to Client Components must be
    serializable under React's rules. Do not pass database connections,
    arbitrary class instances, or secret objects.
-   A Server Component can render a Client Component. A Client Component
    should not directly import a Server Component as if it were an
    ordinary client module; instead, compose server-rendered content
    through props such as `children`.
-   Server Components are not a security guarantee by themselves. Only
    send the client the data it is allowed to see.

### Quick decision

Ask: "Does this UI need browser interaction?" - **No:** prefer a Server
Component. - **Yes:** use a Client Component for the interactive part,
and keep data access and secrets on the server.

------------------------------------------------------------------------

## 8. Rendering Strategies

Rendering choices can be mixed in one application.

### Static rendering / SSG

HTML and the React Server Component output can be produced ahead of a
request and reused. Useful for documentation, marketing pages, and
content that does not need personalized output on every request.

### Dynamic rendering / request-time SSR

The server renders the route for an incoming request. Useful when output
depends on request-specific data, such as authenticated user information
or request cookies.

### Client-side rendering (CSR)

The browser executes JavaScript to fetch or update UI after the page
loads. Useful for highly interactive portions, but do not assume the
whole Next.js app should be client-rendered.

### Incremental Static Regeneration (ISR)

A static route or cached data can be refreshed after a configured
interval or explicit invalidation, without requiring a full redeploy for
every content change.

### Streaming and Suspense

The server can send ready parts of the page while slower parts are still
loading. `loading.tsx` and `<Suspense>` can provide fallback UI for
these boundaries.

### Important distinction

**Rendering** determines when/how UI output is produced. **Caching**
determines whether data or rendered output is reused. They are related,
but not the same thing.

------------------------------------------------------------------------

## 9. Data Fetching

### Fetch in a Server Component

``` tsx
type Post = { id: number; title: string };

export default async function PostsPage() {
  const response = await fetch("https://api.example.com/posts");

  if (!response.ok) {
    throw new Error("Could not load posts");
  }

  const posts: Post[] = await response.json();

  return (
    <ul>
      {posts.map((post) => <li key={post.id}>{post.title}</li>)}
    </ul>
  );
}
```

Always consider network failures and non-2xx HTTP responses. `fetch()`
does not automatically throw just because the server returned a 404 or
500.

### Fetch from a database

Server Components can call a server-side data-access function or ORM
directly. This avoids an unnecessary internal HTTP request to your own
Route Handler in many cases.

``` tsx
import { getProducts } from "@/lib/products";

export default async function ProductsPage() {
  const products = await getProducts();
  return <div>{/* render products */}</div>;
}
```

Keep database credentials and database modules server-only. Do not
import a server-only database module into a Client Component.

### Parallel fetching

When independent requests do not depend on each other, run them in
parallel:

``` tsx
const [products, categories] = await Promise.all([
  getProducts(),
  getCategories(),
]);
```

This can reduce total waiting time compared with awaiting each request
sequentially.

### Sequential fetching

Use sequential awaits only when a later operation genuinely depends on
an earlier result, such as fetching a user before fetching that user's
orders.

### Client-side fetching

Use client-side fetching when data needs to update in response to
browser interactions or client state. Libraries such as SWR or TanStack
Query can help with revalidation, retries, deduplication, and client
cache management. Avoid fetching data in `useEffect` by default when a
Server Component can fetch it more simply.

### `generateStaticParams`

For known dynamic paths, `generateStaticParams()` can tell Next.js which
routes to prerender during the build. It can be combined with runtime
handling for other paths depending on configuration.

------------------------------------------------------------------------

## 10. Caching and Revalidation

### Next.js 15 default to remember

In Next.js 15, `fetch()` requests are **not cached by default** in the
Data Cache. `GET` Route Handlers are also not cached by default.
Explicitly opt into caching when it fits the data and route.

### Uncached fetch

``` tsx
const response = await fetch("https://api.example.com/profile", {
  cache: "no-store",
});
```

Use for data that must be requested fresh on each server render, such as
rapidly changing or request-specific data.

### Explicitly cached fetch

``` tsx
const response = await fetch("https://api.example.com/categories", {
  cache: "force-cache",
});
```

Use for data that can safely be reused until revalidation or
invalidation.

### Time-based revalidation

``` tsx
const response = await fetch("https://api.example.com/products", {
  next: { revalidate: 60 },
});
```

The cached data becomes eligible for revalidation after the specified
number of seconds. This is not a guarantee that a refresh happens at the
exact second; revalidation is generally triggered by subsequent
requests.

### Tag-based invalidation

``` tsx
const response = await fetch("https://api.example.com/products", {
  next: { tags: ["products"] },
});
```

Then, in trusted server-side mutation logic, use
`revalidateTag("products")` according to the API semantics of your
installed Next.js version. Tags let related cached data be invalidated
together.

### Path-based invalidation

``` tsx
import { revalidatePath } from "next/cache";

revalidatePath("/products");
```

Use after a successful mutation when a path needs revalidation.
`revalidatePath` and `revalidateTag` are server-only APIs and must be
called in supported server contexts.

### Route segment options

A page or layout can use options such as:

``` tsx
export const revalidate = 60;
```

Other options include `dynamic`, `fetchCache`, and `dynamicParams`. Use
these intentionally and check the version-specific rules; prefer
per-request `fetch` options when they express the requirement clearly.

### Four caching layers to distinguish

1.  **Request memoization:** deduplicates matching fetches during a
    single React render/request.
2.  **Data Cache:** stores eligible server data across requests.
3.  **Full Route Cache:** stores prerendered route output when eligible.
4.  **Router Cache:** stores RSC route segments on the client for
    navigation.

Do not confuse browser HTTP caching with the Next.js server Data Cache.
Also remember development mode may behave differently from production.

------------------------------------------------------------------------

## 11. Loading, Errors, and Streaming

### `loading.tsx`

Provides a loading fallback for a route segment. It is automatically
integrated with Suspense.

``` tsx
export default function Loading() {
  return <p>Loading products...</p>;
}
```

### `<Suspense>`

Use a Suspense boundary to show a fallback around a slower child
component:

``` tsx
import { Suspense } from "react";

export default function Page() {
  return (
    <>
      <h1>Dashboard</h1>
      <Suspense fallback={<p>Loading chart...</p>}>
        <SlowChart />
      </Suspense>
    </>
  );
}
```

### `error.tsx`

Must be a Client Component. It receives an error and a `reset()`
function that can retry rendering the segment.

``` tsx
"use client";

export default function ErrorPage({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <section>
      <h2>Something went wrong.</h2>
      <button onClick={() => reset()}>Try again</button>
    </section>
  );
}
```

An `error.tsx` boundary catches errors in its segment's children, not
errors thrown by the same segment's layout. A root-level
`global-error.tsx` is available for root layout failures.

### `notFound()`

``` tsx
import { notFound } from "next/navigation";

const product = await getProduct(id);
if (!product) notFound();
```

This terminates rendering for the route and shows the closest
`not-found.tsx`.

### `redirect()`

Use `redirect("/login")` in supported server contexts when the user must
be sent to another route. It throws a special control-flow signal, so
avoid swallowing it in a broad `try/catch`.

### Streaming

Streaming improves perceived speed by sending available page content
progressively. Use meaningful skeletons and place Suspense boundaries
around the slow areas rather than making the entire page wait
unnecessarily.

------------------------------------------------------------------------

## 12. Route Handlers and API Endpoints

### What is a Route Handler?

A Route Handler defines HTTP behavior using Web `Request` and `Response`
APIs. It is commonly used for webhooks, public APIs, integrations, or
endpoints that need to be called independently.

File: `app/api/users/route.ts` gives endpoint `/api/users`.

``` tsx
import { NextResponse } from "next/server";

export async function GET() {
  return NextResponse.json({ message: "Hello from Next.js" });
}

export async function POST(request: Request) {
  const body = await request.json();
  // Validate body before saving anything.
  return Response.json({ received: true }, { status: 201 });
}
```

### Supported methods

`GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `HEAD`, and `OPTIONS`.
Unsupported methods return `405 Method Not Allowed`.

### Important rules

-   A `page.tsx` and `route.ts` cannot define the same route segment at
    the same location.
-   In Next.js 15, `GET` Route Handlers are not cached by default. Opt
    into static behavior deliberately when appropriate.
-   Route Handlers are not automatically secure. Authenticate,
    authorize, validate input, and rate-limit sensitive endpoints.
-   Use meaningful status codes: `200` success, `201` created, `400` bad
    request, `401` unauthenticated, `403` forbidden, `404` not found,
    `409` conflict, and `500` server error.
-   For webhook endpoints, verify provider signatures and use the raw
    request body when the provider requires it.

### Route Handler or Server Action?

-   **Route Handler:** a public or independently addressable HTTP
    endpoint, external integrations, webhooks, or a REST-style API.
-   **Server Action:** a server mutation invoked from a React/Next.js
    UI, often through a form.
-   Neither removes the need for authentication, authorization, and
    validation.

------------------------------------------------------------------------

## 13. Server Actions and Mutations

### What are Server Actions?

Server Actions (Server Functions used as actions) are async functions
that execute on the server and can be invoked by supported React/Next.js
mechanisms. They are useful for form submissions and data mutations.

### Define a Server Action

A file with `"use server"` at the top exports async server functions:

``` tsx
// app/actions.ts
"use server";

export async function createPost(formData: FormData) {
  const title = formData.get("title");

  if (typeof title !== "string" || !title.trim()) {
    return { error: "A title is required." };
  }

  // Authenticate and authorize here.
  // Save the post through the server-side data layer.
  // Revalidate affected paths/tags after successful persistence.
  return { success: true };
}
```

### Use it with a form

``` tsx
import { createPost } from "@/app/actions";

export default function NewPostForm() {
  return (
    <form action={createPost}>
      <label>
        Title
        <input name="title" required />
      </label>
      <button type="submit">Create post</button>
    </form>
  );
}
```

### React 19 form hooks

-   **`useActionState`:** manages the state returned by an action,
    including validation errors and success messages.
-   **`useFormStatus`:** lets a child component read whether its parent
    form is pending.
-   **`useOptimistic`:** shows an optimistic UI update while a mutation
    is in progress.

### Rules and security

-   A Server Action is not magically private just because it is called
    from a UI. Treat it as a server endpoint that may receive untrusted
    input.
-   Authenticate the current user and verify permissions on every
    sensitive mutation.
-   Validate all submitted fields on the server, even if the browser
    also validates them.
-   Never trust a user ID, price, role, or ownership claim sent by the
    client without checking it against the authenticated session and
    database.
-   Do not expose secrets in action return values.
-   Persist data successfully before telling the user the operation
    succeeded.
-   Revalidate relevant data after successful writes.
-   Keep `"use server"` functions async.

------------------------------------------------------------------------

## 14. Authentication, Authorization, and Middleware

### Authentication vs. authorization

-   **Authentication:** who is the user?
-   **Authorization:** what is this user allowed to do? A logged-in user
    may still lack permission to access an admin page or edit another
    user's resource.

### Common authentication approaches

-   Auth provider such as Clerk or Auth.js
-   Server-managed sessions in secure cookies
-   Identity tokens validated on the server

Prefer established, maintained authentication libraries over inventing
password/session handling from scratch.

### Clerk example for Next.js 15

Clerk APIs can change by package version. Follow the installed Clerk
version's official guide. A common Next.js 15-era pattern uses
`middleware.ts`:

``` tsx
import { clerkMiddleware, createRouteMatcher } from "@clerk/nextjs/server";

const isProtectedRoute = createRouteMatcher([
  "/dashboard(.*)",
  "/settings(.*)",
]);

export default clerkMiddleware(async (auth, req) => {
  if (isProtectedRoute(req)) {
    await auth.protect();
  }
});

export const config = {
  matcher: [
    // Skip Next.js internals and static files when appropriate.
    "/((?!_next|[^?]*\\.(?:html?|css|js(?!on)|jpe?g|webp|png|gif|svg|ttf|woff2?|ico|csv|docx?|xlsx?|zip|webmanifest)).*)",
    "/(api|trpc)(.*)",
  ],
};
```

Use the exact setup recommended for your Clerk version. Newer Next.js
versions may use updated naming or conventions, so do not copy
middleware configuration blindly across major versions.

### Middleware's role

In Next.js 15, `middleware.ts` runs before a matched request is
completed and is useful for quick redirects, rewrites, request checks,
and lightweight session-based gating. It should not replace
authorization checks inside data access functions, Server Actions, or
Route Handlers.

### Security checklist

-   Verify permissions close to the data being accessed.
-   Protect APIs and actions, not only page URLs.
-   Use secure, HttpOnly, and appropriate SameSite cookie settings for
    sessions.
-   Never store sensitive session tokens in `localStorage` when a safer
    session-cookie pattern is available.
-   Avoid trusting role values sent from the browser.
-   Rate-limit login and sensitive mutation endpoints.
-   Avoid leaking stack traces, tokens, or private data in error
    messages.

------------------------------------------------------------------------

## 15. Metadata, SEO, Images, Fonts, and Scripts

### Metadata

Static metadata:

``` tsx
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: "Products | Gadget Store",
  description: "Explore our latest products.",
};
```

Dynamic metadata:

``` tsx
import type { Metadata } from "next";

export async function generateMetadata(
  { params }: { params: Promise<{ id: string }> }
): Promise<Metadata> {
  const { id } = await params;
  const product = await getProduct(id);

  return {
    title: product ? product.name : "Product not found",
    description: product?.description ?? "Product details",
  };
}
```

Metadata exports belong in Server Component route files, not Client
Components. Also review canonical URLs, Open Graph metadata,
`sitemap.ts`, and `robots.ts` where relevant.

### Images with `next/image`

``` tsx
import Image from "next/image";

<Image
  src="/products/keyboard.png"
  alt="Mechanical keyboard"
  width={800}
  height={600}
/>
```

For remote images, configure allowed remote hosts/patterns in
`next.config`. Use accurate `alt` text and size images appropriately.
For images that fill a positioned container, use `fill` and provide a
correctly sized parent container.

### Fonts

`next/font` can optimize and self-host local or Google fonts. This can
reduce layout shifts and avoid extra external font requests.

### Scripts

`next/script` supports controlled loading strategies for third-party
scripts. Avoid loading nonessential scripts before they are needed.

### SEO reminder

Rendering HTML on the server can help search engines and social
previews, but SEO still depends on meaningful content, metadata,
crawlability, performance, and correct canonical/robots configuration.

------------------------------------------------------------------------

## 16. Environment Variables and Security

### `.env.local`

``` env
DATABASE_URL="your-private-database-url"
CLERK_SECRET_KEY="your-secret-key"
NEXT_PUBLIC_SITE_URL="https://example.com"
```

### Key rule

-   Variables prefixed with `NEXT_PUBLIC_` are exposed to the browser
    bundle. Use that prefix only for values safe to disclose.
-   Server-only secrets must not be imported into client code.
-   Add `.env*` files containing secrets to `.gitignore`, while keeping
    safe example files such as `.env.example`.
-   Configure production secrets in the hosting provider's environment
    settings.
-   If a secret is committed to Git, removing the line later is not
    enough. Rotate the exposed credential.

### Server-only modules

Use `import "server-only";` in sensitive server-side modules where
useful to help prevent accidental imports into client code.

### Input safety

Validate and normalize user input. Use parameterized database queries or
a well-maintained ORM. Escape output where appropriate, and never
construct SQL by concatenating untrusted strings.

------------------------------------------------------------------------

## 17. Forms, Validation, and File Uploads

### FormData

HTML forms submitted to Server Actions commonly provide a `FormData`
object. Values can be strings or `File` objects, and missing values can
be `null`.

``` tsx
const email = formData.get("email");
if (typeof email !== "string") {
  return { error: "Email is required." };
}
```

### Validation

Use browser constraints for user experience and server-side validation
for security. Libraries such as Zod can define reusable schemas.
Validate: - Required fields and lengths - Email formats and allowed
values - Numeric ranges and currency - File type, size, and content -
Ownership and authorization

### File uploads

For production uploads, consider object storage or a dedicated media
service. Set size limits, validate file content, avoid trusting file
extensions alone, and use signed upload URLs where appropriate. Do not
assume a Server Action or Route Handler can accept arbitrarily large
files.

------------------------------------------------------------------------

## 18. State Management and Common Hooks

### Where should state live?

-   **Local component state:** `useState` for a menu, toggle, or form
    widget.
-   **URL state:** search, filters, pagination, and shareable navigation
    state.
-   **Server state:** data stored in a database or external API.
-   **Shared client state:** context or a store such as Redux Toolkit or
    Zustand when multiple client components need coordinated state.
-   **Form state:** `useActionState`, `useFormStatus`, or a form library
    when appropriate.

### Useful App Router hooks

-   `useRouter()` from `next/navigation`: navigate or refresh.
-   `usePathname()`: current pathname.
-   `useSearchParams()`: query string.
-   `useParams()`: dynamic route parameters.
-   `useSelectedLayoutSegment()` / `useSelectedLayoutSegments()`:
    inspect active segments.
-   `useActionState()`: manage action state (React 19).
-   `useFormStatus()`: pending form status from a child of the form.
-   `useOptimistic()`: temporary optimistic UI.

### `useEffect`

Use effects for synchronization with external systems, such as
subscriptions or browser APIs. Do not use `useEffect` simply to fetch
every piece of page data if the data can be loaded directly in a Server
Component.

------------------------------------------------------------------------

## 19. Database and Full-Stack Architecture

### Typical flow

1.  Browser requests a route.
2.  Server Component or Route Handler validates the request/session.
3.  A server-side data-access layer queries the database.
4.  The server returns rendered UI or a JSON response.
5.  A Server Action or API mutation validates and authorizes changes.
6.  After a successful write, invalidate the appropriate cache or route.

### Suggested separation

``` text
app/                 # Pages, layouts, route handlers
components/          # Reusable UI components
lib/db.ts            # Database connection
lib/auth.ts          # Auth/session helpers
lib/validators.ts    # Shared validation schemas
lib/data/            # Server-side data-access functions
app/actions.ts       # Server Actions (or colocated action files)
```

### Practical guidance

-   Reuse a database connection correctly in development and serverless
    environments.
-   Keep data-access functions server-only.
-   Return only fields the UI needs.
-   Paginate large lists instead of loading everything.
-   Add indexes for frequently queried fields.
-   Handle missing records and database failures explicitly.
-   Use transactions for multi-step writes that must succeed or fail
    together.
-   Never rely on hiding a button as the only access-control mechanism.

------------------------------------------------------------------------

## 20. Performance and Deployment

### Performance practices

-   Prefer Server Components for noninteractive UI.
-   Keep client boundaries small.
-   Use `next/image` and `next/font`.
-   Fetch independent data in parallel.
-   Use caching intentionally for stable data.
-   Stream slow sections with Suspense.
-   Paginate or virtualize large lists.
-   Avoid unnecessarily large client-side dependencies.
-   Measure with browser developer tools and production builds rather
    than guessing.

### Deployment checklist

-   Run `npm run build`.
-   Confirm environment variables are set in the hosting environment.
-   Confirm database network access and production migrations.
-   Verify image host configuration and callback URLs.
-   Check authentication redirect URLs and allowed origins.
-   Review logs without exposing secrets.
-   Test production behavior, since development caching and rendering
    can differ.
-   Use HTTPS and a supported Node.js version for the chosen Next.js
    release.

------------------------------------------------------------------------

## 21. Testing and Debugging

### What to test

-   Unit tests for pure functions and validation
-   Component tests for UI behavior
-   Integration tests for database/data-access behavior
-   End-to-end tests for login, checkout, forms, navigation, and
    protected routes
-   API tests for status codes, invalid input, and authorization
    failures

### Debugging checklist

-   Read the terminal error and browser console.
-   Check whether the component is server or client.
-   Confirm imports and file names use correct casing.
-   Check the actual installed Next.js and React versions.
-   Verify environment variables are available on the server.
-   Check response status codes and network requests.
-   Test a production build when behavior differs from development.

------------------------------------------------------------------------

## 22. Common Mistakes

1.  Adding `"use client"` to an entire page when only a small widget
    needs interactivity.
2.  Calling `useState` or `useEffect` inside a Server Component.
3.  Importing `useRouter` from `next/router` instead of
    `next/navigation` in the App Router.
4.  Forgetting to `await params` or `await searchParams` in Next.js 15.
5.  Assuming `fetch()` is cached by default in Next.js 15.
6.  Assuming `GET` Route Handlers are cached by default in Next.js 15.
7.  Assuming `fetch()` throws for HTTP 404 or 500 responses.
8.  Fetching from your own Route Handler inside a Server Component when
    a direct server-side data function is simpler.
9.  Treating Middleware as the only protection for private data or
    actions.
10. Trusting client-supplied IDs, roles, prices, or ownership.
11. Exposing secrets with `NEXT_PUBLIC_` or committing `.env.local`.
12. Forgetting to revalidate cached data after a mutation.
13. Putting `metadata` or `generateMetadata` exports in a Client
    Component.
14. Forgetting `alt`, dimensions, or remote host configuration for
    images.
15. Using `error.tsx` without `"use client"`.
16. Swallowing `redirect()` or `notFound()` control-flow signals inside
    broad error handling.
17. Assuming Server Components mean no JavaScript is ever sent to the
    browser; Client Components still ship client code.
18. Copying auth or middleware examples from a different major package
    version without checking the installed version.

------------------------------------------------------------------------

## 23. Quick Revision Cheat Sheet

  Need                                      Use
  ----------------------------------------- ---------------------------------------------
  Public page                               `app/about/page.tsx`
  Shared shell                              `layout.tsx`
  Remount on segment change                 `template.tsx`
  Loading fallback                          `loading.tsx` or `<Suspense>`
  Segment error UI                          `error.tsx`
  Not-found response                        `not-found.tsx` and `notFound()`
  Dynamic URL segment                       `[id]`
  Any one-or-more nested segments           `[...slug]`
  Base path plus optional nested segments   `[[...slug]]`
  Group routes without changing URL         `(group)`
  Interactive state/events                  `"use client"`
  Server-side data fetch                    async Server Component
  Independent HTTP API                      `app/api/.../route.ts`
  UI form mutation                          Server Action
  Server-side redirect                      `redirect()`
  Time-based data refresh                   `next: { revalidate: seconds }`
  Explicit uncached fetch                   `cache: "no-store"`
  Explicit cached fetch                     `cache: "force-cache"`
  Invalidate a path                         `revalidatePath()`
  Invalidate tagged data                    `revalidateTag()`
  Static page metadata                      `metadata` export
  Data-dependent metadata                   `generateMetadata()`
  Optimized image                           `next/image`
  Public asset                              `public/`
  Server secret                             environment variable without `NEXT_PUBLIC_`

### Five principles to remember

1.  **Server by default:** move to the client only when interactivity or
    browser APIs require it.
2.  **Validate on the server:** client validation improves UX but does
    not secure data.
3.  **Choose caching intentionally:** fresh data and reusable data have
    different requirements.
4.  **Protect the data layer:** authorization belongs near the data and
    mutation, not only in navigation or middleware.
5.  **Check the version:** Next.js APIs and defaults evolve. Use
    documentation for the exact major version installed.

------------------------------------------------------------------------

## 24. Official References

Use versioned documentation when revising Next.js 15: - Next.js 15
documentation: https://nextjs.org/docs/15 - Next.js 15 upgrade guide:
https://nextjs.org/docs/15/app/guides/upgrading/version-15 - App Router
overview: https://nextjs.org/docs/15/app - Routing and file conventions:
https://nextjs.org/docs/15/app/api-reference/file-conventions - Server
and Client Components:
https://nextjs.org/docs/15/app/getting-started/server-and-client-components -
Data fetching:
https://nextjs.org/docs/15/app/getting-started/fetching-data - Caching:
https://nextjs.org/docs/15/app/guides/caching - Route Handlers:
https://nextjs.org/docs/15/app/getting-started/route-handlers-and-middleware -
Server Actions and mutations:
https://nextjs.org/docs/15/app/getting-started/updating-data - Metadata:
https://nextjs.org/docs/15/app/getting-started/metadata-and-og-images

**Revision tip:** Learn the route conventions, the server/client
boundary, and Next.js 15 caching defaults first. Then practise one small
CRUD app that includes a dynamic route, a Route Handler, a Server
Action, validation, authentication, and cache revalidation.
