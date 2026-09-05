Housing & Roommate Platform backend using your preferred stack:

TypeScript + Node.js + Express + Prisma + PostgreSQL + JWT

The uploaded code is currently organized around Patient, Doctor, Appointment, etc.; authentication is the main implemented feature.

Recommended House Backend Architecture
house-backend/
├── prisma/
│ ├── schema/
│ │ ├── schema.prisma
│ │ ├── user.prisma
│ │ ├── property.prisma
│ │ ├── room.prisma
│ │ ├── booking.prisma
│ │ ├── roommate.prisma
│ │ ├── payment.prisma
│ │ ├── review.prisma
│ │ ├── favorite.prisma
│ │ └── enums.prisma
│ └── migrations/
│
├── src/
│ ├── server.ts
│ ├── app.ts
│ │
│ └── app/
│ ├── config/
│ │ └── index.ts
│ │
│ ├── lib/
│ │ └── prisma.ts
│ │
│ ├── middleware/
│ │ ├── checkAuth.ts
│ │ ├── globalErrorHandler.ts
│ │ └── notFound.ts
│ │
│ ├── utils/
│ │ ├── catchAsync.ts
│ │ ├── jwt.ts
│ │ ├── sendResponse.ts
│ │ └── pagination.ts
│ │
│ └── module/
│ ├── auth/
│ │ ├── auth.route.ts
│ │ ├── auth.controller.ts
│ │ ├── auth.service.ts
│ │ └── auth.interface.ts
│ │
│ ├── user/
│ │ ├── user.route.ts
│ │ ├── user.controller.ts
│ │ ├── user.service.ts
│ │ └── user.interface.ts
│ │
│ ├── property/
│ │ ├── property.route.ts
│ │ ├── property.controller.ts
│ │ ├── property.service.ts
│ │ └── property.interface.ts
│ │
│ ├── room/
│ ├── booking/
│ ├── roommate/
│ ├── payment/
│ ├── review/
│ ├── favorite/
│ └── admin/
│
├── .env
├── .env.example
├── package.json
├── tsconfig.json
└── prisma.config.ts

Your original project already uses the useful route → controller → service → interface separation, and I recommend keeping that architecture for the House backend.

House Platform Roles

I would change:

SUPER_ADMIN
ADMIN
DOCTOR
PATIENT

to:

SUPER_ADMIN
ADMIN
PROVIDER
CUSTOMER

Where:

CUSTOMER — searches properties, rooms and roommates, sends booking requests, reviews properties.
PROVIDER — creates and manages houses/rooms, manages booking requests.
ADMIN — manages users, properties, bookings, reports and disputes.
SUPER_ADMIN — complete system administration.
Main Prisma Models
User
├── CustomerProfile
└── ProviderProfile

Provider
└── Property
├── PropertyImage
├── Room
│ └── Booking
├── Amenity
├── Favorite
└── Review

Customer
├── Booking
├── Favorite
├── Review
└── RoommateProfile

Payment
└── Booking
Main API
AUTH
POST /api/v1/auth/register
POST /api/v1/auth/login
POST /api/v1/auth/refresh-token
GET /api/v1/auth/me
POST /api/v1/auth/logout

USERS
GET /api/v1/users
GET /api/v1/users/:id
PATCH /api/v1/users/:id
DELETE /api/v1/users/:id

PROPERTIES
POST /api/v1/properties
GET /api/v1/properties
GET /api/v1/properties/:id
PATCH /api/v1/properties/:id
DELETE /api/v1/properties/:id

ROOMS
POST /api/v1/properties/:propertyId/rooms
GET /api/v1/properties/:propertyId/rooms
GET /api/v1/rooms/:id
PATCH /api/v1/rooms/:id
DELETE /api/v1/rooms/:id

BOOKINGS
POST /api/v1/bookings
GET /api/v1/bookings
GET /api/v1/bookings/:id
PATCH /api/v1/bookings/:id/status
DELETE /api/v1/bookings/:id

ROOMMATES
POST /api/v1/roommates/profile
GET /api/v1/roommates
GET /api/v1/roommates/:id
PATCH /api/v1/roommates/:id

FAVORITES
POST /api/v1/favorites
GET /api/v1/favorites
DELETE /api/v1/favorites/:id

REVIEWS
POST /api/v1/reviews
GET /api/v1/properties/:propertyId/reviews
PATCH /api/v1/reviews/:id
DELETE /api/v1/reviews/:id

PAYMENTS
POST /api/v1/payments/create
POST /api/v1/payments/webhook
GET /api/v1/payments
GET /api/v1/payments/:id

ADMIN
GET /api/v1/admin/dashboard
GET /api/v1/admin/users
GET /api/v1/admin/properties
GET /api/v1/admin/bookings
PATCH /api/v1/admin/users/:id/status
PATCH /api/v1/admin/properties/:id/status
Professional Prisma User starting point
model User {
id String @id @default(uuid())
name String
email String @unique
password String
phone String?
profileImage String?
role Role @default(CUSTOMER)
status UserStatus @default(ACTIVE)

customerProfile CustomerProfile?
providerProfile ProviderProfile?

bookings Booking[]
reviews Review[]
favorites Favorite[]
payments Payment[]

createdAt DateTime @default(now())
updatedAt DateTime @updatedAt
isDeleted Boolean @default(false)
deletedAt DateTime?

@@index([email])
@@index([role])
@@index([status])
@@map("users")
}

model CustomerProfile {
id String @id @default(uuid())
userId String @unique
occupation String?
budget Decimal?
preferredArea String?
bio String?

user User @relation(fields: [userId], references: [id])

roommateProfile RoommateProfile?

createdAt DateTime @default(now())
updatedAt DateTime @updatedAt
}

model ProviderProfile {
id String @id @default(uuid())
userId String @unique
businessName String?
address String?
description String?

user User @relation(fields: [userId], references: [id])
properties Property[]

createdAt DateTime @default(now())
updatedAt DateTime @updatedAt
}

And the enums:

enum Role {
SUPER_ADMIN
ADMIN
PROVIDER
CUSTOMER
}

enum UserStatus {
ACTIVE
BLOCKED
SUSPENDED
}

This keeps the professional configuration pattern from the supplied backend—centralized environment configuration, shared Prisma client, JWT middleware, global error handling, and standardized responses.

I can take this one step further and convert the entire uploaded backend specification into the actual House Backend source code, including:

Complete Prisma schema
Auth module
Customer module
Provider module
Property module
Room module
Booking module
Roommate module
Favorite module
Review module
Payment module
Admin module
JWT role-based authorization
Validation with Zod
Pagination/search/filter
Professional error handling
Postman API structure
Seed/admin account
.env.example
package.json, tsconfig.json, Prisma config
Complete app.ts and server.ts
