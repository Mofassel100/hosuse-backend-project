# Project Requirements — House & Roommate Platform

## 1. Overview

House & Roommate Platform connects customers/seeker with property providers for house, apartment, room, and roommate rentals.

A customer can search for available properties or rooms, view property details, check availability, submit a rental request, make a payment, and manage their booking.

A provider can create and manage property listings, add rooms, publish availability, review rental requests, approve or reject customers, and manage bookings.

Admins manage the entire platform: users, providers, properties, rental requests, bookings, payments, reviews, and platform activity.

This document describes what the system must do and the business rules it must follow. The implementation uses **Node.js, TypeScript, Express.js, PostgreSQL, and Prisma ORM**.

---

# 2. User Roles

Three roles exist:

- **Admin**
- **Provider**
- **Customer**

| Role         | How they join                                    | How they log in |
| ------------ | ------------------------------------------------ | --------------- |
| **Customer** | Registers directly with name, email and password | Email/password  |
| **Provider** | Applies through provider registration            | Email/password  |
| **Admin**    | Created by an existing Admin                     | Email/password  |

A user cannot select `ADMIN` during normal registration.

---

## 2.1 Who Can Manage Whom

| Action                   | Customer | Provider |  Admin   |
| ------------------------ | :------: | :------: | :------: |
| Register                 |    ✅    |    ❌    |    ❌    |
| Apply as Provider        |    ❌    |    ✅    |    ❌    |
| View properties          |    ✅    |    ✅    |    ✅    |
| Create property          |    ❌    |    ✅    |    ❌    |
| Update own property      |    ❌    |    ✅    |    ❌    |
| Delete own property      |    ❌    |    ✅    |    ❌    |
| Create rental request    |    ✅    |    ❌    |    ❌    |
| Approve rental request   |    ❌    |    ✅    |    ❌    |
| Reject rental request    |    ❌    |    ✅    |    ❌    |
| Make payment             |    ✅    |    ❌    |    ❌    |
| Add review               |    ✅    |    ❌    |    ❌    |
| Manage customers         |    ❌    |    ❌    |    ✅    |
| Manage providers         |    ❌    |    ❌    |    ✅    |
| Approve provider         |    ❌    |    ❌    |    ✅    |
| Block/unblock user       |    ❌    |    ❌    |    ✅    |
| Manage all properties    |    ❌    |    ❌    |    ✅    |
| Manage payments          |    ❌    |    ❌    |    ✅    |
| View platform statistics |    ❌    | Own data | All data |

---

# 3. Accounts and Authentication

## 3.1 Customer Registration

A customer registers with:

```text
name
email
password
phone
```

The default role is:

```text
CUSTOMER
```

Customers cannot register directly as Admin.

---

## 3.2 Provider Registration

A provider submits an application containing:

```text
name
email
password
phone
address
NID/identity information
property information
```

The provider starts with:

```text
PENDING
```

The provider cannot publish properties until an Admin approves the application.

---

## 3.3 Login

Users log in with:

```text
email
password
```

After successful login, the backend generates:

```text
accessToken
refreshToken
```

Tokens can be stored in secure HTTP-only cookies or returned according to the application's authentication strategy.

---

# 4. Password Management

## 4.1 Forgot Password

Flow:

```text
Customer
   ↓
Enter Email
   ↓
Backend generates OTP
   ↓
OTP sent to email
   ↓
Customer submits OTP
   ↓
Set new password
```

---

## 4.2 Change Password

A logged-in user provides:

```text
currentPassword
newPassword
```

The backend verifies the current password before updating it.

---

# 5. Provider Management

An interested property owner can apply to become a Provider.

Flow:

```text
Provider Application
        ↓
Email Verification
        ↓
Application = PENDING
        ↓
Admin reviews application
        ↓
       ┌───────────────┐
       │               │
    APPROVED        REJECTED
       │               │
       ↓               ↓
 Provider Active    Application End
```

Only an approved Provider can:

- Create properties
- Create rooms
- Publish properties
- Manage rental requests
- Manage bookings

---

# 6. Property Management

A Provider can create a property.

A property contains:

```text
title
description
address
city
area
propertyType
rent
status
images
amenities
providerId
```

Example property types:

```text
APARTMENT
HOUSE
SUBLET
ROOM
HOSTEL
```

---

## 6.1 Property Status

A property can have:

```text
AVAILABLE
RENTED
INACTIVE
```

Only `AVAILABLE` properties are shown in normal customer search results.

---

# 7. Room Management

A property can contain multiple rooms.

A room contains:

```text
name
description
rent
capacity
available
propertyId
```

Example:

```text
Property: Green View Apartment

Room 1
Rent: 8000
Capacity: 1

Room 2
Rent: 12000
Capacity: 2

Room 3
Rent: 15000
Capacity: 3
```

---

# 8. Property Search

Customers can search properties using:

```text
city
area
propertyType
minimum rent
maximum rent
room capacity
availability
```

Example:

```text
GET /api/properties?city=Dhaka
```

or:

```text
GET /api/properties?city=Dhaka&minRent=5000&maxRent=15000
```

---

# 9. Property Pagination

Property list APIs must support pagination.

Example:

```text
GET /api/properties?page=1&limit=10
```

Response:

```json
{
  "success": true,
  "data": [],
  "meta": {
    "page": 1,
    "limit": 10,
    "total": 50,
    "totalPages": 5
  }
}
```

---

# 10. Rental Request

A customer can request to rent an available room.

Flow:

```text
Customer
   ↓
Search Property
   ↓
View Property
   ↓
Select Room
   ↓
Submit Rental Request
   ↓
Provider receives request
   ↓
Provider approves/rejects
```

Rental request statuses:

```text
PENDING
APPROVED
REJECTED
CANCELLED
```

---

# 11. Booking

A booking is created after the rental request has been approved and the customer completes the required payment.

Booking contains:

```text
customerId
providerId
propertyId
roomId
startDate
endDate
rent
status
```

Booking statuses:

```text
PENDING
CONFIRMED
CANCELLED
COMPLETED
```

---

# 12. Booking Workflow

The complete rental workflow is:

```text
Customer
    ↓
Search Property
    ↓
Select Room
    ↓
Rental Request
    ↓
Provider Approval
    ↓
Booking Created
    ↓
Payment
    ↓
Payment Successful
    ↓
Booking CONFIRMED
    ↓
Room becomes unavailable
```

---

# 13. Payment

The platform supports a payment gateway such as:

```text
Stripe
SSLCommerz
bKash
```

The payment flow is:

```text
Customer
    ↓
Create Booking
    ↓
POST /api/payments/create
    ↓
Payment Gateway
    ↓
Customer Pays
    ↓
Webhook
    ↓
Verify Payment
    ↓
Payment = PAID
    ↓
Booking = CONFIRMED
    ↓
Room = unavailable
```

The backend must not trust payment status sent directly from the frontend.

Payment must be verified using the payment gateway's server-side webhook/callback.

---

# 14. Payment Status

Payment can have:

```text
PENDING
PAID
FAILED
REFUNDED
```

Payment information:

```text
id
userId
bookingId
amount
status
transactionId
paymentGateway
createdAt
updatedAt
```

---

# 15. Cancellation and Refund

Customers can cancel a booking according to the platform's cancellation policy.

Example business rule:

| Cancellation time               |            Refund |
| ------------------------------- | ----------------: |
| More than 48 hours before start |    ✅ Full refund |
| 24–48 hours before start        | ⚠️ Partial refund |
| Less than 24 hours              |      ❌ No refund |
| After rental start              |      ❌ No refund |

The exact refund policy can be configured by the Admin.

---

# 16. Reviews

A customer can review a property after completing a rental.

Review contains:

```text
userId
propertyId
rating
comment
```

Rating:

```text
1 → 5
```

A customer cannot submit multiple reviews for the same property unless the business rules explicitly allow it.

Example:

```json
{
  "propertyId": "property-id",
  "rating": 5,
  "comment": "Very clean and comfortable room."
}
```

---

# 17. Favorites

Customers can save properties to their favorites.

Endpoints:

```text
POST   /api/favorites/:propertyId
GET    /api/favorites
DELETE /api/favorites/:propertyId
```

A customer cannot add the same property twice.

Database rule:

```text
unique(userId, propertyId)
```

---

# 18. Admin Dashboard

Admin can view:

```text
Total Customers
Total Providers
Total Properties
Total Rooms
Total Bookings
Total Payments
Total Revenue
Pending Provider Applications
Pending Rental Requests
```

Example:

```json
{
  "users": 250,
  "providers": 45,
  "properties": 120,
  "rooms": 380,
  "bookings": 540,
  "revenue": 1250000
}
```

---

# 19. Admin Operations

Admin can:

```text
Approve Provider
Reject Provider
Block Customer
Unblock Customer
Block Provider
Unblock Provider
Delete Property
Manage Users
Manage Reviews
Manage Bookings
View Payments
View Reports
```

Only Admin can access administrative endpoints.

---

# 20. API Endpoints

## Authentication

```text
POST   /api/auth/register
POST   /api/auth/login
POST   /api/auth/refresh-token
POST   /api/auth/logout
POST   /api/auth/forgot-password
POST   /api/auth/reset-password
POST   /api/auth/change-password
```

## User

```text
GET    /api/users/me
PATCH  /api/users/me
GET    /api/users/:id
```

## Provider

```text
POST   /api/providers/apply
GET    /api/providers/me
PATCH  /api/providers/me
GET    /api/providers/:id
```

## Property

```text
POST   /api/properties
GET    /api/properties
GET    /api/properties/:id
PATCH  /api/properties/:id
DELETE /api/properties/:id
```

## Room

```text
POST   /api/properties/:propertyId/rooms
GET    /api/properties/:propertyId/rooms
GET    /api/rooms/:id
PATCH  /api/rooms/:id
DELETE /api/rooms/:id
```

## Rental Request

```text
POST   /api/rental-requests
GET    /api/rental-requests/my
GET    /api/rental-requests/:id
PATCH  /api/rental-requests/:id/cancel
PATCH  /api/rental-requests/:id/approve
PATCH  /api/rental-requests/:id/reject
```

## Booking

```text
POST   /api/bookings
GET    /api/bookings/my
GET    /api/bookings/:id
PATCH  /api/bookings/:id/cancel
```

## Payment

```text
POST   /api/payments/create
POST   /api/payments/webhook
GET    /api/payments
GET    /api/payments/:id
```

## Review

```text
POST   /api/reviews
GET    /api/properties/:propertyId/reviews
PATCH  /api/reviews/:id
DELETE /api/reviews/:id
```

## Favorite

```text
POST   /api/favorites/:propertyId
GET    /api/favorites
DELETE /api/favorites/:propertyId
```

## Admin

```text
GET    /api/admin/dashboard
GET    /api/admin/users
PATCH  /api/admin/users/:id/block
PATCH  /api/admin/users/:id/unblock
GET    /api/admin/providers
PATCH  /api/admin/providers/:id/approve
PATCH  /api/admin/providers/:id/reject
GET    /api/admin/properties
DELETE /api/admin/properties/:id
GET    /api/admin/bookings
GET    /api/admin/payments
```

This provides significantly more than the required **20 API endpoints**.

---

# 21. Database Models

The main Prisma models are:

```text
User
Profile
Property
Room
RentalRequest
Booking
Payment
Review
Favorite
```

Relationships:

```text
User
 ├── Profile
 ├── Properties
 ├── RentalRequests
 ├── Bookings
 ├── Payments
 ├── Reviews
 └── Favorites

Property
 ├── Rooms
 ├── RentalRequests
 ├── Bookings
 ├── Reviews
 └── Favorites

Room
 ├── RentalRequests
 └── Bookings

Booking
 └── Payment
```

---

# 22. Prisma Role Enum

```prisma
enum Role {
  CUSTOMER
  PROVIDER
  ADMIN
}
```

---

# 23. Prisma Status Enums

```prisma
enum UserStatus {
  ACTIVE
  BLOCKED
}

enum ProviderStatus {
  PENDING
  APPROVED
  REJECTED
}

enum PropertyStatus {
  AVAILABLE
  RENTED
  INACTIVE
}

enum RentalRequestStatus {
  PENDING
  APPROVED
  REJECTED
  CANCELLED
}

enum BookingStatus {
  PENDING
  CONFIRMED
  CANCELLED
  COMPLETED
}

enum PaymentStatus {
  PENDING
  PAID
  FAILED
  REFUNDED
}
```

---

# 24. Authentication & Authorization

Authentication:

```text
JWT
+
Access Token
+
Refresh Token
+
bcrypt
```

Authorization:

```text
checkAuth()
        ↓
verify JWT
        ↓
get user
        ↓
check role
        ↓
allow / reject
```

Example:

```typescript
router.post(
  "/properties",
  checkAuth(Role.PROVIDER),
  validateRequest(createPropertySchema),
  PropertyController.createProperty,
);
```

Admin:

```typescript
router.get(
  "/admin/dashboard",
  checkAuth(Role.ADMIN),
  AdminController.dashboard,
);
```

---

# 25. Validation

Use **Zod** for all:

```text
POST
PUT
PATCH
```

Example:

```typescript
const createPropertySchema = z.object({
  body: z.object({
    title: z.string().min(3),
    description: z.string().min(10),
    address: z.string().min(5),
    city: z.string().min(2),
    rent: z.number().positive(),
  }),
});
```

---

# 26. Database Transactions

Complex operations must use Prisma transactions.

Example booking/payment workflow:

```text
Create Booking
      +
Create Payment
      +
Update Room Availability
```

These operations should be handled atomically.

```typescript
await prisma.$transaction(async (tx) => {
  const booking = await tx.booking.create({
    data: bookingData,
  });

  await tx.payment.create({
    data: paymentData,
  });

  await tx.room.update({
    where: { id: roomId },
    data: { available: false },
  });

  return booking;
});
```

If one operation fails, the transaction rolls back.

---

# 27. Database Indexes

Frequently searched fields should have indexes:

```prisma
@@index([email])
@@index([role])
@@index([city])
@@index([status])
@@index([providerId])
@@index([propertyId])
@@index([customerId])
```

This improves search and filtering performance.

---

# 28. Project Structure

```text
house-backend-project/
│
├── src/
│   ├── app/
│   │   ├── routes.ts
│   │   └── index.ts
│   │
│   ├── config/
│   │   ├── env.ts
│   │   └── database.ts
│   │
│   ├── modules/
│   │   ├── auth/
│   │   ├── user/
│   │   ├── provider/
│   │   ├── property/
│   │   ├── room/
│   │   ├── rentalRequest/
│   │   ├── booking/
│   │   ├── payment/
│   │   ├── review/
│   │   ├── favorite/
│   │   └── admin/
│   │
│   ├── middlewares/
│   │   ├── auth.ts
│   │   ├── validateRequest.ts
│   │   ├── notFound.ts
│   │   └── globalErrorHandler.ts
│   │
│   ├── utils/
│   │   ├── jwt.ts
│   │   ├── bcrypt.ts
│   │   ├── catchAsync.ts
│   │   └── sendResponse.ts
│   │
│   ├── types/
│   │   └── index.d.ts
│   │
│   └── server.ts
│
├── prisma/
│   ├── schema.prisma
│   └── seed.ts
│
├── .env
├── .env.example
├── .gitignore
├── package.json
├── tsconfig.json
└── README.md
```

---

# 29. Five-Day Development Plan

## Day 1 — Planning, Architecture & Database

```text
✓ Select Housing & Roommate Platform
✓ Define Customer / Provider / Admin
✓ Define permissions
✓ Design ERD
✓ Initialize Node.js
✓ Initialize TypeScript
✓ Initialize Express
✓ Configure PostgreSQL
✓ Configure Prisma
✓ Create schema
✓ Run migration
✓ Create seed data
✓ Initialize Git
✓ Deploy initial backend
```

## Day 2 — Authentication & Core APIs

```text
✓ Customer registration
✓ Provider application
✓ Login
✓ JWT
✓ Refresh token
✓ bcrypt
✓ Authentication middleware
✓ RBAC
✓ User profile
✓ Property CRUD
✓ Room CRUD
✓ Postman collection
```

## Day 3 — Business Logic

```text
✓ Rental request
✓ Booking
✓ Reviews
✓ Favorites
✓ Pagination
✓ Filtering
✓ Sorting
✓ Zod validation
✓ Global error handler
✓ Prisma transactions
✓ Database indexes
```

## Day 4 — Payment & Testing

```text
✓ Stripe / SSLCommerz / bKash
✓ Payment creation
✓ Payment webhook
✓ Payment verification
✓ Refund handling
✓ Payment history
✓ Test all 3 roles
✓ Test unauthorized requests
✓ Test validation errors
✓ Test duplicate requests
✓ Test not-found errors
✓ Finalize Postman/Swagger
```

## Day 5 — Deployment & Submission

```text
✓ Production environment variables
✓ Production PostgreSQL
✓ Deploy backend
✓ Test live APIs
✓ Test authentication
✓ Test RBAC
✓ Test payment
✓ 20+ meaningful Git commits
✓ Final README
✓ Admin demo credentials
✓ API walkthrough video
✓ Submit project links
```

---

# 30. Final Architecture

```text
                    HOUSE PLATFORM
                           │
                           ▼
                    Express.js API
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
          Customer      Provider        Admin
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                    Authentication
                       JWT + RBAC
                           │
                           ▼
                       Controllers
                           │
                           ▼
                        Services
                           │
                           ▼
                       Prisma ORM
                           │
                           ▼
                      PostgreSQL
                           │
           ┌───────────────┼───────────────┐
           ▼               ▼               ▼
        Payment         Cloudinary        Redis
        Gateway          Images           Cache
```

## Core Business Flow

```text
CUSTOMER
   │
   ├── Register/Login
   │
   ├── Search Properties
   │
   ├── View Room
   │
   ├── Rental Request
   │
   ├── Provider Approval
   │
   ├── Booking
   │
   ├── Payment
   │
   ├── Confirm Booking
   │
   └── Review
             ▲
             │
         PROVIDER
             │
             ├── Apply
             ├── Admin Approval
             ├── Create Property
             ├── Add Rooms
             ├── Manage Requests
             └── Manage Bookings

             ▲
             │
           ADMIN
             │
             ├── Manage Users
             ├── Approve Providers
             ├── Manage Properties
             ├── Manage Bookings
             ├── Manage Payments
             └── Dashboard
```
