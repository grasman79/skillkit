---
name: tanstack-start
description: TanStack Start framework patterns with type-safe routing and server functions. Trigger words - tanstack start, tanstack router, createFileRoute, createServerFn, server function, type-safe routing, vinxi
---

# TanStack Start

Full-stack React framework with type-safe file-based routing, server functions, and TanStack Query integration.

## When to Use This Skill

- User asks about TanStack Start or TanStack Router
- Project has `app.config.ts` with TanStack Start config
- User mentions type-safe routing or server functions
- Working with file-based routing in React
- User wants to use TanStack Query with SSR

## Security Requirements

**IMPORTANT:** When setting up a new TanStack Start project or working with an existing one, you MUST also apply the [tanstack-nitro-security](../../security/tanstack-nitro-security/SKILL.md) skill to implement security headers middleware.

This is required for:
- New TanStack Start project setup
- Preparing for production deployment
- Security audits

The security skill adds Nitro middleware for HTTP security headers (X-Frame-Options, HSTS, etc.).

## Instructions

### Nitro Installation (Critical)

TanStack Start uses Nitro as its server layer. You **must** install the nightly version - this is the official recommendation from the TanStack Start docs.

**In `package.json`, add:**
```json
"nitro": "npm:nitro-nightly@latest"
```

Then install dependencies. Without this, Nitro will be imported but not actually installed, causing the dev server to fail.

### Vite Configuration (Critical)

The order of plugins in `vite.config.ts` is **critical**. Use this exact order:

```typescript
import tailwindcss from "@tailwindcss/vite";
import { tanstackStart } from "@tanstack/react-start/plugin/vite";
import viteReact from "@vitejs/plugin-react";
import { nitro } from "nitro/vite";
import { defineConfig } from "vite";
import viteTsConfigPaths from "vite-tsconfig-paths";

const config = defineConfig({
  plugins: [
    tanstackStart({ srcDirectory: "app" }),  // Must be FIRST
    nitro(),                                  // Must be SECOND
    viteTsConfigPaths({ projects: ["./tsconfig.json"] }),
    tailwindcss(),
    viteReact(),
  ],
  ssr: {
    external: ["lucide-react"],
    noExternal: [],
  },
  build: {
    minify: "esbuild",
    chunkSizeWarningLimit: 1000,
  },
});

export default config;
```

**Why order matters:** `tanstackStart()` must come before `nitro()` for proper server-side environment variable handling. Wrong order causes "No database host or connection string was set" errors.

### Nitro Configuration File (Required)

Create `nitro.config.ts` at project root. This file is **required** for Nitro to properly load `.env` files in server-side code:

```typescript
import { defineNitroConfig } from "nitro/config";

export default defineNitroConfig({});
```

Even an empty config triggers Nitro's automatic `.env` loading. Without this file, environment variables won't be available in server-side code.

### Environment Variables

- Use `.env` at project root (not `.env.local`)
- Nitro automatically loads `.env` when `nitro.config.ts` exists
- No need for `dotenv/config` in application code

### Project Structure

```
app/
├── routes/
│   ├── __root.tsx         # Root layout
│   ├── index.tsx          # Home (/)
│   ├── about.tsx          # /about
│   ├── posts/
│   │   ├── index.tsx      # /posts
│   │   └── $postId.tsx    # /posts/:postId
│   ├── _layout.tsx        # Pathless layout
│   └── api/
│       └── posts.ts       # API route
└── client.tsx
```

### Basic Route

```tsx
// app/routes/index.tsx
import { createFileRoute } from '@tanstack/react-router';

export const Route = createFileRoute('/')({
  component: HomePage,
});

function HomePage() {
  return <h1>Welcome</h1>;
}
```

### Dynamic Route

```tsx
// app/routes/posts/$postId.tsx
import { createFileRoute } from '@tanstack/react-router';

export const Route = createFileRoute('/posts/$postId')({
  component: PostDetail,
  loader: async ({ params }) => {
    const post = await getPost(params.postId);
    return { post };
  },
});

function PostDetail() {
  const { post } = Route.useLoaderData();
  return <h1>{post.title}</h1>;
}
```

### API Route (for Prisma/Auth)

```ts
// app/routes/api/posts.ts
import { createFileRoute } from '@tanstack/react-router';
import { db } from '@/lib/db';

export const Route = createFileRoute('/api/posts')({
  server: {
    handlers: {
      GET: async ({ request }) => {
        const posts = await db.post.findMany();
        return Response.json({ posts });
      },
      POST: async ({ request }) => {
        const body = await request.json();
        const post = await db.post.create({ data: body });
        return Response.json({ post });
      },
    },
  },
});
```

## Examples

**Root Layout:**
```tsx
// app/routes/__root.tsx
import { createRootRoute, Outlet } from '@tanstack/react-router';

export const Route = createRootRoute({
  component: () => (
    <html>
      <body>
        <Outlet />
      </body>
    </html>
  ),
});
```

**With Loader:**
```tsx
export const Route = createFileRoute('/dashboard')({
  component: Dashboard,
  pendingComponent: () => <div>Loading...</div>,
  errorComponent: ({ error }) => <div>Error: {error.message}</div>,
  loader: async () => {
    const data = await fetchDashboardData();
    return { data };
  },
});
```

**Search Params:**
```tsx
import { z } from 'zod';

export const Route = createFileRoute('/products')({
  component: Products,
  validateSearch: z.object({
    page: z.number().default(1),
    sort: z.enum(['name', 'date']).default('name'),
  }),
});

function Products() {
  const { page, sort } = Route.useSearch();
  const navigate = Route.useNavigate();

  return (
    <select
      value={sort}
      onChange={(e) => navigate({ search: { sort: e.target.value } })}
    >
      <option value="name">Name</option>
      <option value="date">Date</option>
    </select>
  );
}
```

**Navigation:**
```tsx
import { Link, useNavigate } from '@tanstack/react-router';

<Link to="/posts/$postId" params={{ postId: '123' }}>View Post</Link>
<Link to="/products" search={{ page: 2 }}>Page 2</Link>

const navigate = useNavigate();
navigate({ to: '/dashboard' });
```

**Protected Route:**
```tsx
// app/routes/_auth.tsx
export const Route = createFileRoute('/_auth')({
  beforeLoad: async () => {
    const session = await getSession();
    if (!session) {
      throw redirect({ to: '/login' });
    }
    return { session };
  },
  component: () => <Outlet />,
});
```

**Client Fetching API:**
```tsx
function ProductsPage() {
  const [products, setProducts] = useState([]);

  useEffect(() => {
    fetch('/api/products', { credentials: 'include' })
      .then(r => r.json())
      .then(d => setProducts(d.products));
  }, []);

  return <ul>{products.map(p => <li key={p.id}>{p.name}</li>)}</ul>;
}
```

## Data Fetching Strategy

**Start with loaders. They're like a mini-Query.** If you need more control, move to TanStack Query.

| Approach | When to Use |
|----------|-------------|
| **Loaders** (default) | Route-level data, initial page load, simple fetch-and-display |
| **TanStack Query** | Mutations, optimistic updates, infinite scroll, manual refetching, shared cache across components, polling |
| **API Routes** | Database operations (Prisma), authentication, webhooks |

**Progression path:**
1. Start with loaders for all route data
2. Add TanStack Query when you need mutations or cross-component cache
3. Use API routes for server-only code (DB, auth)

```tsx
// Step 1: Simple loader (start here)
export const Route = createFileRoute('/posts')({
  loader: async () => {
    const posts = await fetch('/api/posts').then(r => r.json());
    return { posts };
  },
  component: Posts,
});

// Step 2: Graduate to Query when you need mutations
import { useMutation, useQueryClient } from '@tanstack/react-query';

function Posts() {
  const { posts } = Route.useLoaderData();
  const queryClient = useQueryClient();

  const deleteMutation = useMutation({
    mutationFn: (id: string) => fetch(`/api/posts/${id}`, { method: 'DELETE' }),
    onSuccess: () => queryClient.invalidateQueries({ queryKey: ['posts'] }),
  });

  return /* ... */;
}
```

### Preventing UI Flash / Double-Load (Critical)

**Never use `useEffect` + `useState` for initial page data.** This causes the classic flash: content renders on server, disappears into a skeleton, then reappears after client-side fetch.

```tsx
// BAD - causes flash/double-load
function Dashboard() {
  const [stats, setStats] = useState(null);
  const [projects, setProjects] = useState([]);

  useEffect(() => {
    fetch('/api/dashboard').then(r => r.json()).then(setStats);
    fetch('/api/projects').then(r => r.json()).then(d => setProjects(d.projects));
  }, []);

  if (!stats) return <DashboardSkeleton />;  // User sees flash here
  return <DashboardContent stats={stats} projects={projects} />;
}

// GOOD - skeleton shows during navigation, page renders once with data
export const Route = createFileRoute('/dashboard')({
  loader: async () => {
    const [stats, projects] = await Promise.all([
      fetch('/api/dashboard').then(r => r.json()),
      fetch('/api/projects').then(r => r.json()),
    ]);
    return { stats, projects: projects.projects };
  },
  pendingComponent: () => <DashboardSkeleton />,  // Shows DURING navigation (correct)
  component: Dashboard,
});

function Dashboard() {
  const { stats, projects } = Route.useLoaderData();
  // Data is ready - page renders once, no second loading state
  return <DashboardContent stats={stats} projects={projects} />;
}
```

**Rules:**

1. **All initial page data goes in `loader`** - The page should render complete on first paint
2. **Use `pendingComponent` for navigation loading** - Skeleton shows BEFORE the page renders (during loader execution), not after. This is the correct skeleton pattern
3. **Parallel fetch in loaders** - Use `Promise.all()` to avoid waterfalls
4. **Reserve client-side fetching for user actions** - Form submissions, infinite scroll, real-time updates
5. **Auth checks in `beforeLoad`** - Don't render a page then redirect after checking auth on the client

```tsx
// GOOD - auth checked before render, no flash of unauthorized content
export const Route = createFileRoute('/dashboard')({
  beforeLoad: async () => {
    const session = await getSession();
    if (!session) throw redirect({ to: '/login' });
    return { session };
  },
  loader: async ({ context }) => {
    const data = await fetchDashboardData(context.session.userId);
    return { data };
  },
  component: Dashboard,
});
```

### Loader Caching (Skeleton Only on First Visit)

**The skeleton should only show on the first navigation.** Subsequent visits to the same page should be instant from cache. TanStack Router supports this via `staleTime`.

```tsx
// Skeleton shows on first visit, cached for 5 minutes after
export const Route = createFileRoute('/dashboard')({
  loader: async () => {
    const [stats, projects] = await Promise.all([
      fetch('/api/dashboard').then(r => r.json()),
      fetch('/api/projects').then(r => r.json()),
    ]);
    return { stats, projects: projects.projects };
  },
  pendingComponent: () => <DashboardSkeleton />,
  staleTime: 5 * 60 * 1000,  // Data stays fresh for 5 minutes
  component: Dashboard,
});
```

**How it works:**
- First visit to `/dashboard`: loader runs, skeleton shows, page renders with data
- Navigate to `/settings` then back to `/dashboard` (within 5 min): instant, no skeleton, cached data served
- After 5 minutes: data is stale, next visit re-runs loader with skeleton

**Common staleTime values:**
- `0` (default) - Always refetch on navigation (skeleton every time)
- `30_000` (30s) - Good for frequently changing data
- `5 * 60 * 1000` (5 min) - Good for dashboards, profiles
- `Infinity` - Never refetch (only load once per session)

**For more control, use TanStack Query in the loader:**

```tsx
// TanStack Query gives you staleTime + background refetching + shared cache
export const Route = createFileRoute('/dashboard')({
  loader: async ({ context }) => {
    // ensureQueryData uses cache if fresh, fetches if stale
    const stats = await context.queryClient.ensureQueryData({
      queryKey: ['dashboard-stats'],
      queryFn: fetchDashboardStats,
      staleTime: 5 * 60 * 1000,
    });
    return { stats };
  },
  pendingComponent: () => <DashboardSkeleton />,
  component: Dashboard,
});

function Dashboard() {
  // useLoaderData for initial data, useSuspenseQuery for reactive updates
  const { stats } = Route.useLoaderData();
  return <DashboardContent stats={stats} />;
}
```

**When to use which:**

| Approach | Use when |
|----------|----------|
| `staleTime` on route | Simple caching, data only needed on this route |
| TanStack Query `ensureQueryData` | Same data used across multiple routes, need background refetching, need manual invalidation |

**Preloading on hover (instant navigation):**

```tsx
// TanStack Router can preload routes on hover/intent
<Link to="/dashboard" preload="intent">Dashboard</Link>
```

When the user hovers over the link, the loader starts running. By the time they click, data is already cached - zero skeleton, zero wait.

**See also:** `workflow/best-practices` skill for the full Server-First Rendering guide.

## Reference Files

- [routing.md](routing.md) - Advanced routing patterns
- [server-functions.md](server-functions.md) - Server function patterns

## Critical: API Routes vs Server Functions

**Use API routes** (`/api/*.ts`) for Prisma and Better Auth:
- Route files with Prisma imports can leak to client bundle
- API routes are purely server-side

```ts
// CORRECT: API route for DB operations
// app/routes/api/users.ts
export const Route = createFileRoute('/api/users')({
  server: {
    handlers: {
      GET: async () => {
        const users = await db.user.findMany();
        return Response.json({ users });
      },
    },
  },
});
```

## Performance: FastResponse (Node.js)

If deploying to Node.js with Nitro, you can get a ~5% throughput improvement by replacing the global Response constructor with srvx's optimized FastResponse.

**Install srvx:**
```bash
bun add srvx
```

**Add to `src/server.ts` (or `app/server.ts`):**
```typescript
import { FastResponse } from "srvx";
globalThis.Response = FastResponse;
```

This works because srvx's FastResponse includes an optimized `_toNodeResponse()` path that avoids the overhead of standard Web Response to Node.js conversion. Only applies to Node.js deployments using Nitro/h3/srvx.

## Tips

- Use `Route.useParams()`, `Route.useSearch()`, `Route.useLoaderData()`
- API routes at `/api/*` are safe for Prisma/auth
- Use `credentials: 'include'` when fetching from client
- Validate search params with Zod schemas

## How to Verify

### Quick Checks
- `npm run build` - Build completes, route tree generates correctly
- `npm run dev` - Dev server starts without errors
- Check `routeTree.gen.ts` shows new routes

### Manual Verification
- Navigate to new route - page loads with correct data
- Dynamic routes resolve params correctly
- API routes return expected data (`curl localhost:3000/api/...`)
- Loaders fetch data before page renders

### Common Issues
- Route not found: Check file naming matches route path exactly
- Loader data undefined: Ensure loader returns data, use `Route.useLoaderData()`
- Type errors on params: Regenerate route tree with `npm run dev`
- Prisma in client bundle: Move DB code to API routes, not loaders
- "No database host or connection string was set": Missing `nitro.config.ts` or wrong Vite plugin order (tanstackStart must come before nitro)
- Environment variables undefined in server code: Add `nitro.config.ts` at project root
