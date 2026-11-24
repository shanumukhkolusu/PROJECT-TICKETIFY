# 🎬 TICKETIFY – Full Stack Online Movie & Event Ticket Booking Platform

[![React](https://img.shields.io/badge/React-18.3-61dafb.svg?style=for-the-badge&logo=react)](https://reactjs.org/)
[![Vite](https://img.shields.io/badge/Vite-6.1-646CFF.svg?style=for-the-badge&logo=vite)](https://vitejs.dev/)
[![Redux Toolkit](https://img.shields.io/badge/Redux%20Toolkit-RTK%20Query-764ABC.svg?style=for-the-badge&logo=redux)](https://redux-toolkit.js.org/)
[![Node.js](https://img.shields.io/badge/Node.js-Express%20MVC-339933.svg?style=for-the-badge&logo=node.js)](https://nodejs.org/)
[![MongoDB](https://img.shields.io/badge/MongoDB%20Atlas-Mongoose-47A248.svg?style=for-the-badge&logo=mongodb)](https://www.mongodb.com/atlas)
[![Razorpay](https://img.shields.io/badge/Razorpay-HMAC%20SHA--256-02042B.svg?style=for-the-badge&logo=razorpay)](https://razorpay.com/)
[![Vercel](https://img.shields.io/badge/Vercel-Serverless%20Deploy-000000.svg?style=for-the-badge&logo=vercel)](https://vercel.com/)

**TICKETIFY** is a complete, production-grade Online Entertainment Ticket Booking System inspired by BookMyShow. Built from scratch with a dark cinematic interface, real-time seat selection, concurrency-safe seat locking, cryptographic payment verification, QR code digital tickets, and an administrative control panel.

---

## 📑 Table of Contents

1. [Project Overview & Architecture](#1-project-overview--architecture)
2. [Technology Stack](#2-technology-stack)
3. [Five Mandatory Academic Evaluation Rubrics](#3-five-mandatory-academic-evaluation-rubrics)
   - [Rubric 1: JWT Authentication](#rubric-1-jwt-authentication)
   - [Rubric 2: Redux Toolkit & RTK Query](#rubric-2-redux-toolkit--rtk-query)
   - [Rubric 3: Vercel Deployment](#rubric-3-vercel-deployment)
   - [Rubric 4: Third-Party API Integration (TMDB & Razorpay)](#rubric-4-third-party-api-integration-tmdb--razorpay)
   - [Rubric 5: MongoDB Atlas & Concurrency Seat Locking](#rubric-5-mongodb-atlas--concurrency-seat-locking)
4. [Database Schemas & Models](#4-database-schemas--models)
5. [Backend API Documentation](#5-backend-api-documentation)
6. [Frontend Application Pages (21 Pages)](#6-frontend-application-pages-21-pages)
7. [Step-by-Step Local Setup & Execution Guide](#7-step-by-step-local-setup--execution-guide)
8. [Demonstration Guide for Course Evaluators](#8-demonstration-guide-for-course-evaluators)
9. [Vercel Deployment Walkthrough](#9-vercel-deployment-walkthrough)

---

## 1. Project Overview & Architecture

TICKETIFY follows the classic **Model-View-Controller (MVC)** design pattern on the backend coupled with a **Component-Driven Redux Single Page Application (SPA)** on the frontend.

```
                              ┌─────────────────────────────────────────┐
                              │            TICKETIFY CLIENT             │
                              │   React 18 + Vite + Redux Toolkit SPA   │
                              └────────────────────┬────────────────────┘
                                                   │
                            HTTP / RTK Query Hooks │ (Bearer JWT / Cookies)
                                                   ▼
                              ┌─────────────────────────────────────────┐
                              │            EXPRESS MVC API              │
                              │        Vercel Serverless / Node         │
                              └───────────┬───────────────┬─────────────┘
                                          │               │
                 ┌────────────────────────┴────┐     ┌────┴────────────────────────┐
                 ▼                             ▼     ▼                             ▼
       ┌──────────────────┐           ┌──────────────────┐               ┌──────────────────┐
       │  MONGODB ATLAS   │           │    TMDB API      │               │   RAZORPAY SDK   │
       │ - Users          │           │ - Trending       │               │ - Create Orders  │
       │ - Movies/Events  │           │ - Movie Details  │               │ - HMAC SHA-256   │
       │ - Bookings       │           │ - High-Res Art   │               │   Signatures     │
       │ - SeatLocks(TTL) │           └──────────────────┘               └──────────────────┘
       └──────────────────┘
```

---

## 2. Technology Stack

### Frontend:
- **Core:** React.js 18 with Vite 6
- **State Management:** Redux Toolkit (`@reduxjs/toolkit` v2.5)
- **Data Fetching:** RTK Query (`fetchBaseQuery` with multi-tag cache invalidation)
- **Routing:** React Router DOM v7
- **Styling:** Modern CSS custom properties, glassmorphism design system, dark palette
- **Icons:** Lucide React (`lucide-react`)
- **Visuals & E-Tickets:** `qrcode.react`, `canvas-confetti`

### Backend:
- **Runtime:** Node.js v24 (CommonJS)
- **Server Framework:** Express.js v4.21
- **Database & ODM:** MongoDB Atlas with Mongoose v8.9
- **Authentication:** JSON Web Tokens (`jsonwebtoken`) & `bcryptjs`
- **Security:** HTTP-only cookies (`cookie-parser`), CORS whitelist, Crypto HMAC SHA-256

### Third-Party APIs:
- **The Movie Database (TMDB) API:** Real movie metadata, posters, backdrops, ratings, trailers.
- **Razorpay Payments API:** Test-mode orders, checkout popup, cryptographic server validation.

---

## 3. Five Mandatory Academic Evaluation Rubrics

### Rubric 1: JWT Authentication
- **Password Hashing:** Passwords are never stored in plain text. Pre-save Mongoose hook hashes passwords using `bcryptjs` with 10 salt rounds.
- **JWT Generation:** Signed tokens containing user `_id`, `name`, `email`, and `role`, expiring in 7 days.
- **Verification Middleware:** Intercepts `Authorization: Bearer <token>` and `req.cookies.token`.
- **Role-Based Authorization:** `authorize('admin')` middleware enforces strict role gates for admin analytics, CRUD, and cancellations.
- **Key Files:**
  - `backend/models/User.js`
  - `backend/middleware/auth.js`
  - `backend/middleware/role.js`
  - `backend/controllers/authController.js`
  - `backend/routes/authRoutes.js`
  - `frontend/src/features/auth/authSlice.js`
  - `frontend/src/components/ProtectedRoute.jsx`
  - `frontend/src/components/AdminRoute.jsx`

---

### Rubric 2: Redux Toolkit & RTK Query
- **Single Store:** `frontend/src/app/store.js`
- **Auth Slice:** `frontend/src/features/auth/authSlice.js` (stores token, user, isAuthenticated, localStorage sync).
- **API Slice:** `frontend/src/features/api/apiSlice.js`
- **Automatic Header Injection:** `prepareHeaders` extracts token from Redux state / localStorage.
- **Cache Tags:** `tagTypes: ['Auth', 'Movie', 'Event', 'Booking', 'Admin']`
- **Automatic Invalidation:** Creating bookings or events invalidates appropriate tags and automatically triggers refetches across views.
- **Components consume RTK Query Hooks:**
  - `useRegisterMutation`, `useLoginMutation`, `useLogoutMutation`, `useGetProfileQuery`
  - `useGetMoviesQuery`, `useGetTrendingMoviesQuery`, `useGetMovieDetailsQuery`
  - `useGetAvailableSeatsQuery`, `useLockSeatsMutation`, `useCreateBookingMutation`, `useVerifyPaymentMutation`
  - `useGetDashboardStatisticsQuery`, `useGetAllBookingsQuery`, `useCreateEventMutation`

---

### Rubric 3: Vercel Deployment
- **Fullstack Serverless Configuration:** Configured through `vercel.json` and `api/index.js`.
- **Cached DB Connection:** `backend/config/db.js` reuses MongoDB Mongoose connections across warm serverless function invocations to eliminate connection spikes.
- **Routing & Rewrites:** Rewrites `/api/(.*)` to `api/index.js` and all other paths to `frontend/dist/index.html`.
- **Key Files:**
  - `vercel.json`
  - `api/index.js`
  - `backend/config/db.js`
  - `backend/app.js`

---

### Rubric 4: Third-Party API Integration (TMDB & Razorpay)

#### A. TMDB Movie Database Integration
- **Live Endpoint Integration:** Queries TMDB v3 API for trending, search, and movie details.
- **Curated High-Res Dataset:** Seamless fallback dataset ensures the application works instantly out-of-the-box before the user provides their API key.
- **Key Files:**
  - `backend/utils/tmdb.js`
  - `backend/controllers/movieController.js`

#### B. Razorpay Test-Mode Payment Integration
- **Order Creation:** Backend generates Razorpay order with unique receipt (`TKF-XXXX-XXXX`).
- **Client Checkout Modal:** Opens Razorpay modal or instant testing simulator.
- **HMAC SHA-256 Verification:** Backend verifies cryptographic signature:
  `expected = HMAC-SHA256(order_id + '|' + payment_id, secret)`
  Booking transitions to `CONFIRMED` only upon valid signature verification!
- **Key Files:**
  - `backend/utils/razorpay.js`
  - `backend/controllers/paymentController.js`
  - `frontend/src/components/PaymentModal.jsx`

---

### Rubric 5: MongoDB Atlas & Concurrency Seat Locking
- **Database Connection:** Connects to `online_ticket_booking` database on MongoDB Atlas.
- **Models:** User, Movie, Event, Booking, Payment, SeatLock.
- **Concurrency-Safe Seat Locking:**
  - When users proceed to checkout, an 8-minute temporary lock is inserted into `SeatLock`.
  - MongoDB compound unique index `{ targetId: 1, showDate: 1, showtime: 1, seatNumber: 1 }` immediately catches and blocks duplicate seat selections.
  - Active locks automatically expire via MongoDB TTL index (`expires: 0`).
- **Key Files:**
  - `backend/models/Booking.js`
  - `backend/models/Payment.js`
  - `backend/models/SeatLock.js`
  - `backend/controllers/bookingController.js`

---

## 4. Database Schemas & Models

| Model | Collection | Primary Fields |
|---|---|---|
| **User** | `users` | `name`, `email` (unique), `password` (hashed), `phone`, `role` (enum: user, admin), `createdAt` |
| **Movie** | `movies` | `tmdbId` (unique), `title`, `description`, `poster`, `backdrop`, `genres`, `rating`, `duration`, `showtimes`, `ticketPrices` |
| **Event** | `events` | `title`, `description`, `category`, `image`, `venue`, `location`, `eventDate`, `showtimes`, `ticketPrice`, `totalSeats`, `status` |
| **Booking** | `bookings` | `bookingId` (unique), `userId` (ref User), `itemType`, `movieId`, `eventId`, `showDate`, `showtime`, `selectedSeats`, `totalAmount`, `paymentStatus`, `bookingStatus`, `qrCode` |
| **Payment** | `payments` | `bookingId` (ref Booking), `userId` (ref User), `razorpayOrderId`, `razorpayPaymentId`, `amount`, `paymentStatus`, `paymentMethod`, `signatureVerified` |
| **SeatLock** | `seatlocks` | `targetId`, `itemType`, `showDate`, `showtime`, `seatNumber`, `userId`, `expiresAt` (TTL Index) |

---

## 5. Backend API Documentation

### Authentication (`/api/auth`)
- `POST /api/auth/register` - Create account & generate JWT
- `POST /api/auth/login` - Authenticate & return token
- `POST /api/auth/logout` - Clear auth cookies
- `GET /api/auth/profile` - Current user details (Protected)
- `PUT /api/auth/profile` - Update user details (Protected)

### Movies & Events (`/api/movies`, `/api/events`)
- `GET /api/movies` - List movies with filters (genre, language, search)
- `GET /api/movies/trending` - Top rated & trending movies
- `GET /api/movies/search?q=...` - Search movies
- `GET /api/movies/:id` - Single movie details
- `POST /api/movies/sync` - Synchronize with TMDB (Admin)
- `GET /api/events` - List live events (Concerts, Comedy, etc.)
- `GET /api/events/:id` - Single event details
- `POST /api/events` - Create event (Admin)
- `PUT /api/events/:id` - Edit event (Admin)
- `DELETE /api/events/:id` - Delete event (Admin)

### Seat Management & Bookings (`/api/bookings`)
- `GET /api/bookings/available-seats` - Current booked & locked seats
- `POST /api/bookings/lock-seats` - Concurrency-safe 8-minute seat reservation
- `POST /api/bookings/release-locks` - Release active locks
- `POST /api/bookings` - Create booking & generate Razorpay order
- `GET /api/bookings/my-bookings` - Logged-in user's booking history
- `GET /api/bookings/:id` - View single booking
- `PUT /api/bookings/:id/cancel` - Cancel booking and process refund

### Payments (`/api/payments`)
- `POST /api/payments/verify` - HMAC SHA-256 signature verification & booking confirmation
- `POST /api/payments/failure` - Record payment error
- `GET /api/payments/:bookingId` - View payment transaction record

### Admin Portal (`/api/admin`)
- `GET /api/admin/statistics` - Real-time KPI analytics & category breakdown
- `GET /api/admin/bookings` - All system bookings with search & status filters
- `PUT /api/admin/bookings/:id/status` - Update booking status
- `GET /api/admin/users` - Registered user directory
- `DELETE /api/admin/users/:id` - Delete user account

---

## 6. Frontend Application Pages (21 Pages)

### Public Pages:
1. **Home Page (`/`)** - Hero blockbuster banner, trending movies carousel, live events, feature highlights.
2. **Login Page (`/login`)** - JWT authentication, remember me, fast demo credentials.
3. **Register Page (`/register`)** - Full account creation, role selection for testing.
4. **Movies Page (`/movies`)** - All movie cards with dynamic genre & language filters.
5. **Movie Details Page (`/movies/:id`)** - Synopsis, ratings, trailers, 5-day date picker, theater showtimes.
6. **Search Results Page (`/search`)** - Cross-catalog search for movies and events.
7. **Events Page (`/events`)** - Concerts, comedy, and theater with city filters.
8. **About Page (`/about`)** - System architecture and 5 evaluation criteria documentation.

### Authenticated User Pages:
9. **Seat Selection Page (`/booking/:type/:id`)** - Screen curvature, 3 tiers (Recliner, Premium, Regular), color legend, 8-min lock.
10. **Checkout Page (`/checkout`)** - Seat breakdown, convenience fee, 18% GST tax calculation, timer.
11. **Payment Result Page (`/payment-result/:bookingId`)** - Confetti animation, transaction reference, direct link to pass.
12. **My Bookings Page (`/my-bookings`)** - Booking cards, status badges, cancel button, view pass.
13. **User Profile Page (`/profile`)** - User identity, update name/phone, total bookings counter.
14. **Ticket Confirmation Page (`/tickets/:bookingId`)** - Perforated cinema pass design, turnstile QR code, Print / Save PDF.

### Administrator Pages:
15. **Admin Dashboard (`/admin`)** - Gross revenue, total bookings, registered users, recent transaction feed.
16. **Manage Catalog (`/admin/events`)** - Movies & events table, edit/delete actions, TMDB sync.
17. **Add Event Page (`/admin/events/new`)** - Form to create concert or stage performance.
18. **Edit Event Page (`/admin/events/edit/:id`)** - Update venue, showtimes, and prices.
19. **Manage Bookings (`/admin/bookings`)** - All bookings across users with status modification.
20. **Manage Users (`/admin/users`)** - Registered customer accounts and administration list.
21. **Revenue Analytics (`/admin/analytics`)** - Category breakdown charts, average order values, conversion rates.

---

## 7. Step-by-Step Local Setup & Execution Guide

### Prerequisites:
- **Node.js:** v18+ (tested on Node v24)
- **NPM:** v9+

### 1. Clone & Enter Project Directory
```powershell
cd TICKETIFY
```

### 2. Configure Environment Variables
Inside `backend/.env`:
```env
PORT=5000
MONGODB_URI=mongodb+srv://shanumukhkolusu_db_user:YOUR_NEW_PASSWORD@cluster0.pt2xmbb.mongodb.net/online_ticket_booking?retryWrites=true&w=majority
JWT_SECRET=ticketify_super_secure_jwt_secret_key_2026_production
JWT_EXPIRE=7d
TMDB_API_KEY=your_tmdb_api_key_here
RAZORPAY_KEY_ID=rzp_test_ticketify2026
RAZORPAY_KEY_SECRET=ticketify_test_secret_key_2026
CLIENT_URL=http://localhost:5173
NODE_ENV=development
```
*(Replace `YOUR_NEW_PASSWORD` with your rotated MongoDB Atlas cluster password to persist live data directly).*

Inside `frontend/.env`:
```env
VITE_API_BASE_URL=/api
VITE_RAZORPAY_KEY_ID=rzp_test_ticketify2026
```

### 3. Install Dependencies
```powershell
# Install backend dependencies
cd backend
npm.cmd install

# Install frontend dependencies
cd ../frontend
npm.cmd install
```

### 4. Start the Application
Open two terminal windows:

**Terminal 1 (Backend API):**
```powershell
cd backend
npm.cmd start
# Server starts on http://localhost:5000
```

**Terminal 2 (Frontend Client):**
```powershell
cd frontend
npm.cmd run dev
# Frontend runs on http://localhost:5173
```

Navigate to `http://localhost:5173` in your browser.

---

## 8. Demonstration Guide for Course Evaluators

| Step | Action | What Evaluator Observes |
|---|---|---|
| **1. Public Exploration** | Open `http://localhost:5173` | Dark cinematic UI, trending TMDB movies, live events, genre filters. |
| **2. Demo Authentication** | Click **Sign In**, click **👤 Demo User** button | Form auto-fills `user@ticketify.com` / `user123`. Submits `POST /api/auth/login`. JWT returned and stored in Redux & localStorage. |
| **3. Seat Selection & Concurrency Hold** | Click any movie (e.g. *Dune: Part Two*), pick showtime, select seats (e.g., C4, C5) | Real-time seat counter, subtotal calculation. Clicking "Proceed to Pay" locks seats for 8 minutes via `POST /api/bookings/lock-seats`. |
| **4. Concurrency Protection Verification** | In an incognito window, open the same show | Seats C4 and C5 appear as **Temporarily Held (Amber)**. Attempting to select them blocks double booking. |
| **5. Razorpay Checkout & Signature Verification** | On Checkout page, click **Pay with Razorpay Gateway** | Gateway modal opens. Evaluator can use standard Razorpay test card or click **Simulate Instant Test Payment (HMAC Verified)**. Backend verifies cryptographic signature: status updates to `CONFIRMED`. |
| **6. Verified E-Ticket Pass** | Redirects to `/payment-result/:id` then `/tickets/:id` | High-resolution tear-strip ticket pass with Turnstile QR code and **Print / Save as PDF** button. |
| **7. Booking History & Refund** | Click **My Bookings** in navbar | Displays booking record. Clicking **Cancel Booking** refunds payment and releases seat reservation. |
| **8. Admin Portal & Analytics** | Logout and click **🛡️ Demo Admin** (`admin@ticketify.com` / `admin123`) | Unlocks Admin Console (`/admin`), KPI metrics, revenue charts, Catalog CRUD, and User directory. |

---

## 9. Vercel Deployment Walkthrough

To deploy TICKETIFY on Vercel:

1. **Install Vercel CLI (optional) or push to GitHub:**
   ```bash
   git init
   git add .
   git commit -m "Initial commit of TICKETIFY full-stack platform"
   git remote add origin https://github.com/your-username/ticketify.git
   git push -u origin main
   ```

2. **Connect to Vercel:**
   - Import your GitHub repository in the Vercel Dashboard.
   - Framework Preset: **Vite**.
   - Root Directory: `./` (or leave default).

3. **Configure Environment Variables in Vercel Project Settings:**
   - `MONGODB_URI` = your full MongoDB Atlas connection string with password
   - `JWT_SECRET` = your 64-character JWT secret
   - `TMDB_API_KEY` = your TMDB v3 API key
   - `RAZORPAY_KEY_ID` = your Razorpay Test Key ID
   - `RAZORPAY_KEY_SECRET` = your Razorpay Test Key Secret
   - `NODE_ENV` = `production`

4. **Deploy:**
   - Vercel automatically detects `vercel.json` and builds both the static React frontend from `frontend/` and the serverless Node backend from `api/index.js`.
   - Your application will be live with full database connectivity, JWT authentication, and payments!

---

### 🎓 Academic Evaluation Checklist

- [x] **Criterion 1 (JWT Auth):** Hashed passwords with bcrypt, signed JWT, Auth headers, Protected routes, Admin vs User roles.
- [x] **Criterion 2 (Redux API):** Redux Toolkit store, `apiSlice`, `authSlice`, RTK Query hooks throughout all components.
- [x] **Criterion 3 (Vercel Deploy):** `vercel.json`, serverless entrypoint `api/index.js`, cached Mongoose connection pool.
- [x] **Criterion 4 (Third-Party APIs):** Live TMDB API integration + Razorpay test gateway with HMAC SHA-256 validation.
- [x] **Criterion 5 (MongoDB Atlas):** Mongoose models for User, Movie, Event, Booking, Payment, SeatLock with concurrency-safe locking.

