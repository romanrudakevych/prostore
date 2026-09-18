# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- `npm run dev` — start dev server (Next.js, Turbopack not configured — plain `next dev`)
- `npm run build` — production build (`next.config.ts` sets `typescript.ignoreBuildErrors: true`, so type errors will NOT fail the build — run `tsc` manually if you need a real type check)
- `npm run lint` — ESLint via `next lint`
- `npm test` / `npm run test:watch` — Jest (no `jest.config.*` file exists yet and no `*.test.*` files currently exist in the repo; add config before relying on this)
- `npx prisma generate` — regenerate Prisma client (also runs automatically via `postinstall`)
- `npx prisma migrate dev --name <name>` — create/apply a migration after editing `prisma/schema.prisma`
- `npx tsx db/seed.ts` (or similar runner) — seed the DB from `db/sample-data.ts`
- `npm run email` — copies `.env` into `node_modules/react-email` and runs the React Email preview server on port 3001 (email templates presumably live under an `email/` dir when added)

## Architecture

This is a Next.js 15 App Router e-commerce app ("Prostore") using Prisma + PostgreSQL (Neon), Auth.js v5, Tailwind v4 + shadcn/ui, Uploadthing for image uploads, and Zod for all validation.

### Route groups (`app/`)
- `app/(root)` — public storefront: home, product detail (`product/[slug]`), cart, checkout flow (`shipping-address` → `payment-method` → `place-order` → `order/[id]`)
- `app/(auth)` — `sign-in`, `sign-up`
- `app/admin` — admin dashboard: `overview`, `products` (list/create/edit at `products/[id]`), `orders`. Gated by `requireAdmin()` (see below)
- `app/user` — signed-in user's own `profile` and `orders`
- `app/api/auth/[...nextauth]` — Auth.js handler
- `app/api/uploadthing` — `core.ts` defines the Uploadthing file router (`OurFileRouter`), `route.ts` exposes it as a Next.js route handler

### Server actions, not API routes
Business logic lives in `lib/actions/*.actions.ts` (`product`, `cart`, `order`, `user`) as `"use server"` files called directly from components/pages. There is no REST/tRPC layer for app data — new features should follow this same pattern rather than adding API routes. Conventions used throughout these files:
- Zod-parse input with the relevant schema from `lib/validators.ts` before touching the DB
- Wrap mutations in `try/catch`, return `{ success: boolean, message: string }` (via `formatError()` from `lib/utils.ts`) rather than throwing, so the caller (often a form) can render the message
- Call `revalidatePath(...)` after mutations that affect a listing page
- Reads return `convertToPlainObject(...)` to strip Prisma's non-plain (Decimal/Date) wrapper types before passing data to Client Components

### Validation & types
- `lib/validators.ts` is the single source of truth for Zod schemas (products, cart, orders, shipping, payment, auth forms). `updateProductSchema` extends `insertProductSchema` with an `id`.
- `types/index.ts` derives all domain types from those schemas via `z.infer<...>`, then intersects with DB-only fields (`id`, `createdAt`, relations, etc.) rather than hand-maintaining parallel types. Add new domain types the same way instead of writing a separate interface.
- `lib/constants/index.ts` holds env-driven config and form default-value objects (e.g. `productDefaultValues`, `signInDefaultValues`) that forms initialize from — keep new form defaults here too.

### Auth
- `auth.ts` configures Auth.js (Credentials provider, Prisma adapter, JWT session strategy, 30-day maxAge). The `jwt`/`session` callbacks propagate `id`/`role`/`name` onto the session, and on sign-in merge the guest's `sessionCartId` cart into the now-authenticated user's cart.
- `auth.config.ts` holds only the route-protection `authorized` callback (regex list of protected paths) plus session-cart-cookie bootstrapping, so it can be imported by `middleware.ts` (edge-safe) without pulling in the Prisma/bcrypt-dependent parts of `auth.ts`. Note `auth.ts`'s own `authorized` callback duplicates/supersedes this list for the actual runtime auth check — when adding a protected route, update the regex list in `auth.ts` (and mirror in `auth.config.ts` if middleware-level behavior matters).
- `lib/auth-guard.ts`'s `requireAdmin()` is called at the top of admin pages/actions to redirect non-admins to `/unauthorized`.

### Database
- `prisma/schema.prisma` is the schema of record; run migrations rather than editing the DB directly. Money fields are `Decimal(12,2)`; IDs are `Uuid` with `gen_random_uuid()` defaults.
- `Cart.items` and `Order`/`OrderItem` store line items as JSON rather than normalized join rows for the cart (cart is JSON blob; orders are normalized into `OrderItem`).
- `db/prisma.ts` currently exports a plain `PrismaClient` singleton (cached on `globalThis` in dev to avoid connection exhaustion on hot reload). The file also contains a large commented-out alternate implementation using the Neon serverless driver adapter + `$extends` to stringify Decimal fields — if you re-enable the Neon adapter, note that `convertToPlainObject()` in `lib/utils.ts` is currently relied on instead to make Decimal/Date fields plain-JSON-safe for Client Components.

### Components
- `components/ui` — shadcn/ui primitives (generated; prefer regenerating via shadcn CLI over hand-editing where possible)
- `components/shared` — cross-cutting composed components (`header`, `product`, etc.)
- `components/admin` — admin-only components
- Path aliases: `@/components`, `@/lib`, `@/hooks`, `@/ui` (see `components.json`); Tailwind config lives in `assets/styles/globals.css` (Tailwind v4 CSS-based config, no `tailwind.config.js`)

### Images
Product images are uploaded via Uploadthing (`lib/uploadthing.ts` exports `UploadButton`/`UploadDropzone` typed against `app/api/uploadthing/core.ts`'s router) and served from `utfs.io` (whitelisted in `next.config.ts` `images.remotePatterns`).
