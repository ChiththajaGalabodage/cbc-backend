<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:0F172A,100:047857&height=170&section=header&text=CBC%20E-Commerce%20API&fontColor=FFFFFF&fontSize=40&fontAlignY=37&animation=fadeIn" alt="CBC E-Commerce API banner" width="100%" />

# CBC E-Commerce — Backend API

**A Node.js, Express and MongoDB REST API supporting CBC's clothing e-commerce storefront.**

Product catalogue · Authentication · Google sign-in · Customer orders · Admin operations

[![Node.js](https://img.shields.io/badge/Node.js-JavaScript-339933?logo=nodedotjs&logoColor=white)](https://nodejs.org/)
[![Express](https://img.shields.io/badge/Express-4-000000?logo=express&logoColor=white)](https://expressjs.com/)
[![MongoDB](https://img.shields.io/badge/MongoDB-6-47A248?logo=mongodb&logoColor=white)](https://www.mongodb.com/)
[![Mongoose](https://img.shields.io/badge/Mongoose-8-880000?logo=mongoose&logoColor=white)](https://mongoosejs.com/)

**[Frontend Repository](https://github.com/ChiththajaGalabodage/cbc-frontend)** · **[Frontend Demo](https://cbc-frontend-tau.vercel.app/)** · **[Developer Profile](https://github.com/ChiththajaGalabodage)**

</div>

---

## Overview

This repository contains the **server-side REST API** for the CBC clothing e-commerce application. It connects an Express server to MongoDB with Mongoose and provides endpoints for user authentication, products and orders.

### Key functionality

- **User accounts:** Registration, email/password login and Google account login.
- **JWT-based authentication:** Bearer-token verification and role-aware operations.
- **Product catalogue:** List products, retrieve a product by ID and search by name/alternative names.
- **Product administration:** Add, update and delete products with admin checks.
- **Order processing:** Create authenticated orders, calculate totals from stored product prices, list relevant orders and update order status (admin).
- **Password recovery flow:** OTP email sending and password-reset endpoints (see security note before using in production).

> **Development status:** This documents the routes and capabilities present in the public code. It does not claim that payment processing, automated tests or production-grade security are implemented.

## Technology stack

| Purpose | Technology |
| --- | --- |
| Runtime | Node.js (JavaScript ES modules) |
| HTTP API | Express 4 |
| Database / ODM | MongoDB, Mongoose 8 |
| Authentication | JSON Web Tokens (`jsonwebtoken`), bcrypt |
| Google user info requests | Axios |
| Email / OTP delivery | Nodemailer |
| Configuration | dotenv |
| Cross-origin requests | cors |
| Development reload | nodemon |

## Architecture

```text
CBC Frontend (React)
       │ HTTP/JSON + optional Bearer token
       ▼
Express Server (index.js, port 3000)
       ├── /api/users     → userRoute → userController
       ├── /api/products  → productRoute → productController
       └── /api/orders    → orderRoute → orderController
                                           │
                                    Mongoose models
                                           │
                                        MongoDB
```

## Getting started

### Prerequisites

- Node.js and npm (a current LTS release is recommended).
- A local or hosted MongoDB database.
- A strong JWT signing secret.

### 1. Clone the repository

```bash
git clone https://github.com/ChiththajaGalabodage/cbc-backend.git
cd cbc-backend
npm install
```

### 2. Set environment variables

Create an `.env` file in the backend repository root:

```dotenv
MONGODB_URL=mongodb://127.0.0.1:27017/cbc-ecommerce
JWT_KEY=replace_with_a_long_random_secret
```

These are the **environment variable names read by the current server code**. Replace the values with your own configuration. Never commit this file.

**Email/OTP configuration needs a security fix:** The current user controller contains hardcoded SMTP credentials. Before enabling password reset, revoke exposed credentials, refactor the mail transport to load its credentials from server-side environment variables, and update your local `.env` accordingly. Simply adding `SMTP_EMAIL` or `SMTP_APP_PASSWORD` to `.env` will **not** change the existing code until you refactor the mail transport.

### 3. Start the API

```bash
npm start
```

The `start` script runs `nodemon index.js`. With the current code, the server listens on **`http://localhost:3000`** (the port is hardcoded in `index.js`). A successful database connection is reported in the console.

> There is no working automated test command configured yet: `npm test` currently exits with an error message.

## API reference

**Base URL (local):** `http://localhost:3000`

### User routes — `/api/users`

| Method | Endpoint | Purpose | Access |
| --- | --- | --- | --- |
| `POST` | `/api/users` | Register a user | Public (admin role creation is restricted) |
| `POST` | `/api/users/login` | Log in using email and password | Public |
| `POST` | `/api/users/login/google` | Log in using a Google access token | Public |
| `POST` | `/api/users/send-otp` | Request a password-reset OTP | Public (security hardening needed) |
| `POST` | `/api/users/reset-password` | Reset password with OTP | Public (security hardening needed) |
| `GET` | `/api/users` | Return authenticated user's token claims | Authenticated |
| `GET` | `/api/users/gau` | List users (admin) or return own details | Authenticated |

### Product routes — `/api/products`

| Method | Endpoint | Purpose | Access |
| --- | --- | --- | --- |
| `GET` | `/api/products` | List available products (admins can see all) | Public |
| `GET` | `/api/products/search/:query` | Search available products | Public |
| `GET` | `/api/products/:productId` | Get one product | Public (unavailable items restricted) |
| `POST` | `/api/products` | Create a product | Admin |
| `PUT` | `/api/products/:productId` | Update a product | Admin |
| `DELETE` | `/api/products/:productId` | Delete a product | Admin |

### Order routes — `/api/orders`

| Method | Endpoint | Purpose | Access |
| --- | --- | --- | --- |
| `POST` | `/api/orders` | Create an order | Authenticated |
| `GET` | `/api/orders` | View all orders (admin) or own orders (customer) | Authenticated |
| `PUT` | `/api/orders/:orderId/:status` | Change an order's status | Admin |

### Authentication header

For endpoints requiring authentication:

```http
Authorization: Bearer YOUR_JWT_TOKEN
```

### Request examples

**Register a customer**

```bash
curl -X POST http://localhost:3000/api/users \
  -H "Content-Type: application/json" \
  -d '{"firstName":"Alex","lastName":"Perera","email":"alex@example.com","password":"change-me"}'
```

**Get available products**

```bash
curl http://localhost:3000/api/products
```

**Create a product (admin JWT required)**

```bash
curl -X POST http://localhost:3000/api/products \
  -H "Authorization: Bearer YOUR_ADMIN_JWT" \
  -H "Content-Type: application/json" \
  -d '{"productId":"CBC-P001","name":"Custom Shirt","description":"Handcrafted cotton shirt","image":[],"labelledPrice":3500,"price":3000,"stock":10,"isAvailable":true}'
```

**Submit an order (signed-in customer JWT required)**

```bash
curl -X POST http://localhost:3000/api/orders \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"phone":"0770000000","address":"Colombo, Sri Lanka","products":[{"productId":"CBC-P001","qty":2}]}'
```

The backend computes order prices from the database rather than trusting client-submitted totals. The order ID uses a `CBC` prefix. New orders default to a `pending` status.

## Project structure

```text
cbc-backend/
├── controllers/
│   ├── productController.js  # Product retrieval, search and CRUD
│   ├── userController.js     # Authentication, users and password recovery
│   └── orderController.js    # Order creation, listing and status updates
├── models/
│   ├── product.js            # Product schema
│   ├── user.js               # User schema
│   ├── order.js              # Order schema
│   └── otp.js                # OTP schema
├── routes/
│   ├── productRoute.js
│   ├── userRoute.js
│   └── orderRoute.js
├── index.js                  # Express setup, JWT handling, MongoDB, routes
├── .gitignore
├── package.json
└── README.md
```

## Database entities

| Model | Representative fields |
| --- | --- |
| `Product` | `productId`, `name`, `description`, `image`, `price`, `labelledPrice`, `stock`, `isAvailable` |
| `User` | `email`, `firstName`, `lastName`, hashed `password`, `role`, `isBlocked` |
| `Order` | `orderId`, `email`, `address`, `phone`, `products`, `total`, `status`, `date` |
| `OTP` | `email`, `otp` |

## Security actions before production

**Security changes are required before deploying or relying on password recovery.** In the reviewed public code:

1. SMTP credentials are hardcoded in `controllers/userController.js`. **Revoke/rotate exposed credentials immediately**, move replacements to server-side configuration and remove secrets from Git history.
2. The password-reset OTP is returned in an API response. Do **not** expose OTP values in responses; apply expiration, retry limits and rate limiting.
3. Review the Google sign-in and password-hashing flows, including how Google-created accounts are stored.
4. Tighten CORS to approved origins and add input validation, authorization tests and appropriate error handling.
5. Keep `.env` out of source control and use a different JWT secret in every environment.

These are **recommended fixes**, not features already implemented.

## Roadmap

- [ ] Add API validation and integration tests.
- [ ] Implement environment-based SMTP credentials and secure OTP recovery.
- [ ] Add pagination and filtering to product and order listings.
- [ ] Make the listening port configurable for deployment.
- [ ] Document production deployment and add monitoring.
- [ ] Introduce secure payment processing only if the project requires it.

## Related repository

- **Client:** [CBC E-Commerce Frontend](https://github.com/ChiththajaGalabodage/cbc-frontend)

## Author

**Chiththaja Galabodage** — Software Engineering Undergraduate  
[GitHub](https://github.com/ChiththajaGalabodage) · [LinkedIn](https://www.linkedin.com/in/chiththaja-galabodage-40734b2b8/)

---

<div align="center"><sub>Backend component of the CBC full-stack e-commerce project.</sub></div>
