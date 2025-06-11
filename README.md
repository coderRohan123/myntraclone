# 🛒 Myntra Clone - Full Stack E-commerce Project

This is a full-stack Myntra-like e-commerce web application built with:

- **Frontend:** Next.js (App Router)
- **Backend:** NestJS
- **Database:** MySQL using Prisma ORM
- **Auth:** JWT-based login/register (user & admin)

---

## 🔧 Features

### 👤 User
- Register / Login
- Browse Products with:
  - Infinite Scroll
  - Filters (by category, price)
  - Sort (by price)
- Add to Cart (with quantity)
- Simulate Buy button
- View Order History

### 🛠️ Admin
- Add, Edit, Delete Products (name, color, price, image, category)

---

## 🌐 API

Swagger documentation available at:  
👉 **http://localhost:3000/api**

---

## ⚙️ Getting Started

### 📦 Backend Setup
```bash
cd backend
npm install
npx prisma generate
npx prisma migrate dev
npm run start:dev
```
Make sure you have a `.env` file in the backend root:
```env
DATABASE_URL="mysql://username:password@localhost:3306/myntra"
JWT_SECRET=your_jwt_secret
```

### 🌍 Frontend Setup
```bash
cd frontend
npm install
npm run dev
```

---

## 📁 Folder Structure

### 📦 Backend (NestJS + Prisma)
```
backend/
├── prisma/
│   ├── schema.prisma
│   └── migrations/
├── src/
│   ├── auth/
│   ├── users/
│   ├── products/
│   ├── cart/
│   ├── orders/
│   ├── main.ts
│   └── app.module.ts
├── test/
├── .env
├── package.json
└── tsconfig.json
```

### 🌍 Frontend (Next.js)
```
frontend/
├── app/
│   ├── login/
│   ├── register/
│   ├── products/
│   ├── cart/
│   ├── orders/
│   └── layout.tsx
├── components/
├── lib/
│   └── api.ts
├── styles/
├── public/
├── package.json
└── tsconfig.json
```

---

## 🔐 Authentication
- JWT stored in localStorage
- Axios sends `Authorization: Bearer <token>` for authenticated requests
- Admin access is role-based with guards in NestJS

---

## 🧭 API Routes Summary

### 🧑 Auth
- `POST /auth/register` → Register user
- `POST /auth/login` → Login user (returns JWT)

### 👤 Users
- `GET /users/me` → Get current user (requires auth)

### 🛍️ Products
- `GET /products` → List products (with filters/sort)
- `GET /products/:id` → Get product by ID
- `POST /products` → Add product (admin only)
- `PUT /products/:id` → Update product (admin only)
- `DELETE /products/:id` → Delete product (admin only)

### 🛒 Cart
- `GET /cart` → Get cart items for current user
- `POST /cart` → Add product to cart `{ productId, quantity }`
- `PUT /cart` → Update quantity in cart
- `DELETE /cart/:productId` → Remove item from cart

### 📦 Orders
- `POST /orders` → Place an order (buy cart)
- `GET /orders` → View order history

---

## 🧩 Database Schema

### 📦 Tables
- **User:** id, email, password, role (USER/ADMIN)
- **Product:** id, name, price, category, image, color
- **Cart:** id, userId, productId, quantity
- **Order:** id, userId, createdAt
- **OrderItem:** id, orderId, productId, quantity

📌 You can generate the ER diagram using Prisma or tools like [dbdiagram.io](https://dbdiagram.io).

---

## 🚀 Developer Tips
- Check API at `http://localhost:3000/api`
- If getting 401 errors → Check token in localStorage
- If CORS errors → Add CORS config in `main.ts`

---

## 📌 Future Improvements
- Payment gateway integration
- Image uploads (Cloudinary or S3)
- Pagination for orders
- Wishlist feature

---

## ✨ Happy Coding!

This project is built to help you **learn full-stack development** using modern technologies like **Next.js, NestJS, Prisma, and MySQL.**
