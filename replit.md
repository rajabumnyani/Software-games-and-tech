# GameVault Marketplace

## Overview

A professional digital marketplace for games and software with two main user types: Buyers and Sellers.

## Stack

- **Monorepo tool**: pnpm workspaces
- **Node.js version**: 24
- **Package manager**: pnpm
- **TypeScript version**: 5.9
- **API framework**: Express 5
- **Database**: PostgreSQL + Drizzle ORM
- **Validation**: Zod (`zod/v4`), `drizzle-zod`
- **API codegen**: Orval (from OpenAPI spec)
- **Build**: esbuild (CJS bundle)
- **Frontend**: React + Vite + Tailwind CSS v4
- **Auth**: Replit Auth (OIDC + PKCE)
- **Payments**: Flutterwave (card + mobile money)
- **File uploads**: Multer (local disk storage)

## Structure

```text
artifacts-monorepo/
├── artifacts/              # Deployable applications
│   ├── api-server/         # Express API server
│   └── marketplace/        # React + Vite frontend
├── lib/                    # Shared libraries
│   ├── api-spec/           # OpenAPI spec + Orval codegen config
│   ├── api-client-react/   # Generated React Query hooks
│   ├── api-zod/            # Generated Zod schemas from OpenAPI
│   ├── db/                 # Drizzle ORM schema + DB connection
│   └── replit-auth-web/    # Browser auth hook (useAuth)
└── scripts/                # Utility scripts
```

## Features

### User Authentication
- Replit Auth (OIDC + PKCE) — no custom login forms
- Session-based auth with PostgreSQL session storage
- User roles: buyer, seller, admin

### Storefront (Public)
- Dark-themed homepage with hero banner
- Category filters: Action, RPG, Strategy, Sports, Puzzle, Horror, Simulation, Software, Other
- Search across product titles and descriptions
- Featured product grid

### Seller Dashboard
- Upload game files (ZIP/EXE/RAR) up to 500MB
- Upload screenshots (up to 10 images)
- Set pricing and descriptions
- View sales stats and earnings
- Products go through admin approval before listing

### Admin Panel
- Review pending products
- Approve or reject with reason
- Manage user roles
- Feature/unfeature products

### Payment Integration
- Flutterwave integration (card + Mobile Money: M-Pesa/Tigo Pesa/MTN)
- Orders created before payment
- Payment verified via Flutterwave webhook/verify API
- Demo mode works without FLUTTERWAVE_SECRET_KEY

### My Library (Secure Downloads)
- After successful payment, products appear in library
- Temporary download links (10 minutes, single-use)
- Token-based file access

## API Endpoints

All endpoints at `/api`:
- `GET /healthz` — health check
- `GET /auth/user` — current auth state
- `GET /login` — OIDC login redirect
- `GET /callback` — OIDC callback
- `GET /logout` — logout

- `GET /products` — list approved products (with ?category=, ?search=, ?featured=)
- `GET /products/:id` — product detail
- `POST /products` — create product (seller, multipart)

- `GET /seller/products` — seller's own products
- `PUT /seller/products/:id` — update product
- `DELETE /seller/products/:id` — delete product
- `GET /seller/stats` — earnings stats

- `POST /orders` — create order
- `POST /payments/initialize` — start Flutterwave payment
- `POST /payments/verify` — verify payment completion

- `GET /library` — user's purchased products
- `GET /library/:purchaseId/download` — temporary download URL

- `GET /admin/products` — all products for admin
- `POST /admin/products/:id/approve` — approve product
- `POST /admin/products/:id/reject` — reject with reason
- `GET /admin/users` — all users
- `PUT /admin/users/:id/role` — change user role

- `PUT /users/role` — set my role (buyer/seller)

- `GET /files/download?token=` — secure file download
- `GET /files/:key` — serve uploaded screenshots

## Environment Variables

- `DATABASE_URL` — PostgreSQL connection string (auto-provided)
- `FLUTTERWAVE_SECRET_KEY` — Flutterwave secret key (optional, demo mode if not set)
- `APP_URL` — App base URL for payment redirects
- `REPL_ID` — Replit app ID (auto-provided, for OIDC)
- `PORT` — server port (auto-provided)

## Database Schema

- `sessions` — Replit Auth sessions (required)
- `users` — user profiles with roles
- `products` — game/software listings
- `orders` — purchase orders
- `purchases` — completed purchase records
- `download_tokens` — temporary secure download tokens

## Admin Setup

To grant admin access, use SQL:
```sql
UPDATE users SET role = 'admin' WHERE username = 'your-username';
```
Or use the Admin Panel once you're logged in (if already admin).

## Flutterwave Setup

1. Create a Flutterwave account at flutterwave.com
2. Get your secret key from the dashboard
3. Set `FLUTTERWAVE_SECRET_KEY` environment variable
4. Without the key, payments run in demo mode (automatically "successful")
