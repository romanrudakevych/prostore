# Prostore

> A full-stack e-commerce storefront and admin dashboard built with Next.js.

<img src="/public/images/screenshot.jpg" />

## Features

- [x] User authentication with credentials (Auth.js v5 + Prisma adapter)
- [x] User authorization & route protection (guest / user / admin roles)
- [x] Guest cart with session cookie, merged into the user's cart on sign-in
- [x] Product catalog with search, category filtering, and pagination
- [x] Product detail pages with image gallery
- [x] Cart management (add / update / remove items)
- [x] Multi-step checkout: shipping address → payment method → place order
- [x] Order history, order detail pages, and order summaries
- [x] Cash on Delivery payment flow with admin "mark as paid" / "mark as delivered"
- [x] User profile management (name, email)
- [x] Admin dashboard with sales overview & charts (Recharts)
- [x] Admin product CRUD (create, edit, delete) with multi-image upload
- [x] Admin order management (view, deliver, delete)
- [x] Image uploads via Uploadthing
- [x] Transactional email templates with React Email
- [x] Toast notifications (Sonner)
- [x] Responsive design (Tailwind CSS v4 + shadcn/ui)
- [x] Database seeding with sample products & users

Prostore uses the following technologies:

- [Next.js 15](https://nextjs.org/) (App Router, Server Actions)
- [React 19](https://reactjs.org/)
- [TypeScript](https://www.typescriptlang.org/)
- [Tailwind CSS](https://tailwindcss.com/) + [shadcn/ui](https://ui.shadcn.com/)
- [Prisma](https://www.prisma.io/) ORM
- [PostgreSQL](https://www.postgresql.org/) (developed against [Neon](https://neon.tech/))
- [Auth.js (NextAuth) v5](https://authjs.dev/)
- [Zod](https://zod.dev/) for schema validation
- [React Hook Form](https://react-hook-form.com/)
- [Uploadthing](https://uploadthing.com/) for file/image uploads
- [Recharts](https://recharts.org/) for the admin dashboard
- [React Email](https://react.email/) + [Resend](https://resend.com/)
- [Stripe](https://stripe.com/) & [PayPal](https://developer.paypal.com/) SDKs (installed, scaffolding for card payments in addition to Cash on Delivery)
- [Jest](https://jestjs.io/) for testing

## Getting Started

### Prerequisites

- Node.js 18 or higher
- A PostgreSQL database (this project was developed against [Neon](https://neon.tech/), a serverless Postgres provider)
- An [Uploadthing](https://uploadthing.com/) account for image uploads

### `.env` File

Create a `.env` file in the project root and fill in the following environment variables:

```bash
# App
NEXT_PUBLIC_APP_NAME=Prostore
NEXT_PUBLIC_APP_DESCRIPTION="A modern ecommerce store built with Next.js"
NEXT_PUBLIC_SERVER_URL=http://localhost:3000

# Database (Postgres / Neon)
DATABASE_URL=
DIRECT_URL=

# Auth.js
AUTH_SECRET=
AUTH_URL=http://localhost:3000/api/auth
AUTH_URL_INTERNAL=http://localhost:3000/api/auth

# Checkout
PAYMENT_METHODS=PayPal, Stripe, CashOnDelivery
DEFAULT_PAYMENT_METHOD=PayPal

# Uploadthing
UPLOADTHING_TOKEN=
UPLOADTHING_SECRET=
UPLOADTHING_APPID=
```

- `DATABASE_URL` / `DIRECT_URL` — connection strings for your Postgres database. When using Neon, `DATABASE_URL` is the pooled connection string and `DIRECT_URL` is the direct (non-pooled) connection string used by Prisma Migrate.
- `AUTH_SECRET` — generate one with:
  ```bash
  openssl rand -base64 32
  ```
- `UPLOADTHING_TOKEN` / `UPLOADTHING_SECRET` / `UPLOADTHING_APPID` — from your [Uploadthing](https://uploadthing.com/dashboard) app settings.

### Install Dependencies

```bash
npm install
```

`postinstall` automatically runs `prisma generate`.

### Set Up the Database

Apply the Prisma schema to your database:

```bash
npx prisma migrate deploy
```

Then seed it with sample products and users:

```bash
npx tsx db/seed.ts
```

This creates two accounts you can sign in with:

| Role  | Email               | Password |
| ----- | ------------------- | -------- |
| Admin | admin@example.com   | 123456   |
| User  | user@example.com    | 123456   |

### Run the Development Server

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) with your browser to see the result.

### Other Scripts

```bash
npm run build       # production build
npm run start       # start the production server
npm run lint        # run ESLint
npm test            # run Jest tests
npm run test:watch  # run Jest in watch mode
npm run email       # preview React Email templates at http://localhost:3001
```

## Project Structure

```
app/
  (root)/       storefront: home, product pages, cart, checkout flow
  (auth)/       sign-in, sign-up
  admin/        admin dashboard: overview, products, orders
  user/         signed-in user's profile & orders
  api/          Auth.js and Uploadthing route handlers
lib/
  actions/      server actions for products, cart, orders, and users
  validators.ts Zod schemas shared across forms, actions, and types
  constants/    env-driven config and form defaults
components/
  ui/           shadcn/ui primitives
  shared/       cross-cutting components (header, product cards, etc.)
  admin/        admin-only components
prisma/         database schema and migrations
db/             Prisma client, seed script, and sample data
```

## License

No license file is currently included in this repository. Add a `LICENSE` file (e.g. MIT) if you intend to open-source this project.
