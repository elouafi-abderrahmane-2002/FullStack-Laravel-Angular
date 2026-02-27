# 🛒 Full-Stack E-Commerce App — Laravel + Angular

I wanted to build something end-to-end. Not a tutorial app, not a CRUD demo — a real project where
I had to think about architecture, security, deployment, and how the frontend and backend actually
talk to each other under real conditions.

This e-commerce app is that project.

---

## How it's structured

```
  Browser (Angular SPA)
       │
       │  HTTP/REST (JSON)
       │  Authorization: Bearer <JWT>
       ▼
  ┌─────────────────────────────────┐
  │        Laravel API Backend      │
  │  ┌──────────┐  ┌─────────────┐ │
  │  │  Routes  │  │ Middleware  │ │
  │  │  /api/v1 │  │ JWT Auth    │ │
  │  └────┬─────┘  └──────┬──────┘ │
  │       │               │        │
  │  ┌────▼───────────────▼──────┐ │
  │  │      Controllers           │ │
  │  │  ProductController         │ │
  │  │  OrderController           │ │
  │  │  AuthController            │ │
  │  └────────────┬──────────────┘ │
  │               │                │
  │  ┌────────────▼──────────────┐ │
  │  │    Eloquent ORM / MySQL   │ │
  │  └───────────────────────────┘ │
  └─────────────────────────────────┘
       │
       ▼
  MySQL Database
```

---

## What's inside

**Backend (Laravel)**
- RESTful API with versioned routes (`/api/v1/...`)
- JWT authentication via `tymon/jwt-auth` — login, register, refresh, logout
- Product catalog with categories, filtering, pagination
- Order management with status transitions
- OWASP-aware middleware: rate limiting, input validation, CORS config
- PHPUnit tests for auth and order flows

**Frontend (Angular)**
- Standalone components with lazy-loaded feature modules
- `HttpInterceptor` for automatic JWT header injection and 401 handling
- Responsive product grid + cart built with Bootstrap 5
- Route guards protecting authenticated pages
- Environment-based API URL config (dev / prod)

**DevOps**
- Docker Compose: Laravel app + MySQL + Nginx
- `.env`-based secrets management — no credentials in code

---

## Running it locally

```bash
git clone https://github.com/elouafi-abderrahmane-2002/FullStack-Laravel-Angular.git
cd FullStack-Laravel-Angular

# Start containers
docker-compose up -d

# Install Laravel dependencies
docker exec app composer install
docker exec app php artisan migrate --seed
docker exec app php artisan jwt:secret

# Install Angular dependencies
cd frontend && npm install && ng serve
```

App: `http://localhost:4200` | API: `http://localhost:8000/api/v1`

---

## Request lifecycle (example: add to cart)

```
  Angular CartService
       │  POST /api/v1/cart/items
       │  { "product_id": 42, "qty": 2 }
       │
       ▼
  JWT Middleware ──► valid? ──► No ──► 401 Unauthorized
       │
       ▼ Yes
  CartController@store
       │
       ├── validate input (qty > 0, product exists)
       ├── check stock availability
       └── CartItem::create([...])
            │
            ▼
       MySQL  →  cart_items table
            │
            ▼
  JSON Response { cart_id, items, total }
       │
       ▼
  Angular updates CartComponent state
```

---

## What I actually struggled with

JWT refresh logic was messier than I expected. When an access token expires mid-session, you need
to silently refresh it before retrying the failed request — without the user noticing. Getting the
Angular interceptor to queue concurrent requests during refresh (instead of firing multiple refresh
calls at once) took me a while to get right. Race conditions in async code are humbling.

Also: setting up CORS correctly between the Laravel API and the Angular dev server.
It's one of those things that seems trivial until it's 2am and you're staring at a
`Access-Control-Allow-Origin` error that makes no sense.

---

*Final-year engineering project — ENSET Mohammedia, Big Data & Cloud Computing*
*By **Abderrahmane Elouafi** · [LinkedIn](https://www.linkedin.com/in/abderrahmane-elouafi-43226736b/) · [Portfolio](https://my-first-porfolio-six.vercel.app/)*
