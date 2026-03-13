# Digital Platform for Electric Truck Aftersales & Fleet Management
## Full System Architecture Document

**Document Version:** 1.1
**Prepared by:** CTO / Senior Software Architect
**Date:** March 2026
**Audience:** Solo Founder — Claude Code + Home Lab Development
**Changelog v1.1:** Added local dev environment (TrueNAS Scale + workstation), updated documentation architecture for multi-product line (T7-90/90L/134L/169L × body variants), updated AI tooling to Claude Code, added MinIO for local S3 storage.

---

## Table of Contents

1. [Product Vision](#1-product-vision)
2. [High-Level System Architecture](#2-high-level-system-architecture)
3. [Recommended Tech Stack](#3-recommended-tech-stack)
4. [System Components](#4-system-components)
5. [Database Schema Design](#5-database-schema-design)
6. [API Architecture](#6-api-architecture)
7. [Documentation Architecture](#7-documentation-architecture)
8. [Infrastructure and Deployment Strategy](#8-infrastructure-and-deployment-strategy)
9. [Security Considerations](#9-security-considerations)
10. [Scaling Strategy for FleetOS](#10-scaling-strategy-for-fleetos)
11. [Development Roadmap](#11-development-roadmap-for-a-solo-founder)
12. [AI-Assisted Development Workflow](#12-ai-assisted-development-workflow)
13. [Estimated Infrastructure Cost](#13-estimated-infrastructure-cost)

---

## 1. Product Vision

### The Problem

Electric truck operators in emerging markets (Southeast Asia, China) face a fragmented aftersales experience. Manuals are scattered, warranty claims are manual and opaque, parts sourcing is painful, and fleet monitoring is non-existent. Traditional diesel OEM platforms don't translate to the EV world — they lack battery diagnostics, charge cycle tracking, and predictive maintenance capabilities.

### The Solution

A unified, modular digital platform that starts as a knowledge and aftersales portal and evolves into a full FleetOS — giving operators, service partners, and internal engineers a single source of truth for everything related to their electric trucks.

### Two-Phase Vision

**Phase 1 — Aftersales Knowledge Platform (Months 1–9)**

A public-facing documentation hub and authenticated aftersales portal. Serves customers, service partners, and internal teams. Generates leads through the Marketing & Knowledge Hub. Manages warranties, parts lookups, and service documentation.

**Phase 2 — FleetOS SaaS Platform (Month 10+)**

Extends the platform with real-time vehicle telemetry, fleet dashboards, predictive maintenance, fault monitoring, and multi-tenant SaaS billing. Transforms from a support tool into a revenue-generating operations platform.

### Core Design Principles

- **Modular first:** Every feature is an independent module. Nothing is tightly coupled.
- **Content-driven growth:** Documentation and case studies drive organic traffic and inbound leads.
- **Data ownership:** All vehicle data is owned by the operator, stored securely, and never sold.
- **Developer velocity:** Architecture is optimized for a single developer using AI coding tools.
- **Progressive complexity:** Start simple. Add complexity only when traffic and revenue justify it.

---

## 2. High-Level System Architecture

### Architecture Style: Modular Monolith → Microservices

A **modular monolith** is the right starting architecture for a solo founder. It gives you the speed of a monolith with the logical separation of microservices. When Phase 2 begins, individual modules (e.g., telemetry ingestion) can be extracted into standalone services without rewriting everything.

### System Architecture Diagram (ASCII)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         CLOUDFLARE CDN / WAF                            │
│                    (DDoS protection, edge caching, SSL)                  │
└───────────────────────────────┬─────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         NGINX REVERSE PROXY                             │
│                    (routing, rate limiting, TLS termination)             │
└──────┬──────────────────┬────────────────────┬──────────────────────────┘
       │                  │                    │
       ▼                  ▼                    ▼
┌─────────────┐  ┌─────────────────┐  ┌───────────────────────────────┐
│  DOCUSAURUS │  │   NEXT.JS APP   │  │     NEXT.JS MARKETING SITE    │
│    PORTAL   │  │  (Aftersales +  │  │   (Case Studies, EV Guides,   │
│  (Docs Hub) │  │   Customer UI)  │  │    Knowledge Hub, Lead Gen)   │
│  :3001      │  │    :3000        │  │    :3002                      │
└─────────────┘  └───────┬─────────┘  └──────────────┬────────────────┘
                         │                            │
                         └──────────────┬─────────────┘
                                        │ REST API calls
                                        ▼
                         ┌──────────────────────────────┐
                         │        NESTJS API SERVER      │
                         │   (Core Business Logic)       │
                         │         :4000                 │
                         │                               │
                         │  ┌──────────────────────┐    │
                         │  │  Module: Auth         │    │
                         │  │  Module: Users        │    │
                         │  │  Module: Vehicles     │    │
                         │  │  Module: Warranty     │    │
                         │  │  Module: Parts        │    │
                         │  │  Module: Cases        │    │
                         │  │  Module: Tickets      │    │
                         │  │  Module: Fleet (stub) │    │
                         │  └──────────────────────┘    │
                         └───────────┬──────────────────┘
                                     │
              ┌──────────────────────┼───────────────────┐
              │                      │                   │
              ▼                      ▼                   ▼
  ┌────────────────────┐  ┌─────────────────┐  ┌────────────────────┐
  │   POSTGRESQL DB    │  │      REDIS      │  │   OBJECT STORAGE   │
  │  (Primary store)   │  │  (Cache, Queue) │  │  (S3/Spaces: docs, │
  │                    │  │                 │  │   parts images,    │
  │  Managed on DO     │  │                 │  │   manuals)         │
  └────────────────────┘  └─────────────────┘  └────────────────────┘

- - - - - - - - - - - FUTURE PHASE 2 (FleetOS) - - - - - - - - - - - - -

                         ┌──────────────────────────────┐
                         │     MQTT / KAFKA BROKER       │
                         │  (Vehicle telemetry ingestion)│
                         └───────────────┬──────────────┘
                                         │
                         ┌───────────────▼──────────────┐
                         │   TELEMETRY MICROSERVICE     │
                         │  (Time-series: TimescaleDB)   │
                         └──────────────────────────────┘
```

### Traffic Flow

1. All traffic enters through Cloudflare (DDoS, CDN, caching).
2. Nginx routes requests to one of three frontend applications.
3. Both Next.js apps call the NestJS API for authenticated data.
4. Docusaurus serves static documentation (no API needed).
5. NestJS handles all business logic, writing/reading from PostgreSQL.
6. Redis handles sessions, caching, and background job queues.
7. Object storage (DigitalOcean Spaces / S3-compatible) holds files.

---

## 3. Recommended Tech Stack

### Decision Rationale

Every technology choice below was made against three criteria: **developer velocity** (can a solo developer be productive with AI tools?), **ecosystem maturity** (is there abundant documentation, community, and AI training data?), and **scalability path** (can this grow to 10,000+ vehicles without rewriting?).

### Frontend

| Layer | Technology | Justification |
|---|---|---|
| Web Framework | **Next.js 14 (App Router)** | Server components reduce client bundle, excellent SEO, best AI code generation support |
| Language | **TypeScript** | Type safety prevents runtime bugs; Claude Code generates the most accurate TypeScript due to its deep training on typed codebases |
| Styling | **TailwindCSS + shadcn/ui** | shadcn/ui provides a production-quality component library on top of Tailwind — eliminates weeks of UI work |
| State Management | **Zustand** | Simpler than Redux, excellent TypeScript support, sufficient for this scale |
| Data Fetching | **TanStack Query (React Query)** | Handles caching, refetching, and loading states cleanly — pairs perfectly with REST APIs |
| Form Handling | **React Hook Form + Zod** | Schema-based validation with zero re-renders; Zod schemas can be shared with NestJS backend |
| Documentation Site | **Docusaurus 3** | Purpose-built for multi-language technical docs, Markdown-native, versioning built in |
| Marketing Site | **Next.js 14** | Can share the same codebase/components with the Aftersales app |

### Backend

| Layer | Technology | Justification |
|---|---|---|
| Framework | **NestJS** | Modular, opinionated, TypeScript-native — closest to enterprise patterns without enterprise complexity |
| Language | **TypeScript** | Full-stack TypeScript means shared types between frontend and backend |
| ORM | **Prisma** | Best-in-class TypeScript ORM; migrations, type safety, and schema-first design; Claude Code generates complete Prisma schemas and migrations from natural language requirements |
| Validation | **class-validator + class-transformer** | Native NestJS validation pipeline; integrates with Swagger auto-docs |
| API Docs | **Swagger (OpenAPI)** | Auto-generated from decorators; essential for solo developer and future team onboarding |
| Auth | **JWT + Passport.js** | Industry standard; NestJS has first-class Passport integration |
| Background Jobs | **BullMQ (Redis-backed)** | Handles email sending, report generation, and future telemetry processing |
| Email | **Resend + React Email** | Modern email API with React component-based templates |

### Database & Storage

| Layer | Technology | Justification |
|---|---|---|
| Primary Database | **PostgreSQL 16** | ACID-compliant, JSON support for flexible fields, excellent with Prisma |
| Cache / Queue | **Redis 7** | Session storage, API response caching, BullMQ job queues |
| File Storage (prod) | **DigitalOcean Spaces (S3-compatible)** | Parts images, PDF manuals, warranty documents; CDN-backed |
| File Storage (dev) | **MinIO (self-hosted, Docker)** | S3-compatible local storage on TrueNAS Scale or workstation; identical API means zero code changes between dev and prod |
| Future: Telemetry | **TimescaleDB** (extension to Postgres) | Time-series extension; allows adding vehicle telemetry without a separate DB technology |
| Search | **Meilisearch** (self-hosted) | Fast full-text search for parts catalog, documentation; simple to operate |

### Infrastructure

| Layer | Technology | Justification |
|---|---|---|
| Containerization | **Docker + Docker Compose** | Entire stack runs locally and in production with identical configuration |
| Local Dev Server | **TrueNAS Scale (home lab)** | Runs persistent services (PostgreSQL, MinIO, Meilisearch) over LAN; workstation connects via `.env.local` pointing at TrueNAS IP |
| Local App Runtime | **Workstation PC (Docker Compose)** | Runs hot-reload Next.js, NestJS, and Docusaurus dev servers during active development |
| Reverse Proxy | **Nginx** | Battle-tested, handles SSL termination, rate limiting, request routing |
| CDN / WAF | **Cloudflare (Free/Pro)** | Free tier covers most needs; DDoS protection, caching, global edge |
| Hosting (prod) | **DigitalOcean Droplets** | Predictable pricing, excellent Managed Postgres and managed Redis add-ons |
| CI/CD | **GitHub Actions** | Free for small projects; deploy on push to main |
| Monitoring | **Grafana + Prometheus** | Self-hosted monitoring; alerts on API errors, DB latency, memory |
| Logging | **Loki + Grafana** | Centralized log aggregation without expensive third-party SaaS |
| Secret Management | **Doppler** (free tier) | Injects environment variables into Docker containers securely |

---

## 4. System Components

### Component 1: Documentation Platform (Docusaurus)

**Purpose:** Serve structured, multilingual technical documentation to customers, service engineers, and partners.

**Key Features:**
- Sidebar navigation organized by document category (User Manual, Service Manual, Parts Documentation, Warranty Manual, Troubleshooting)
- i18n support for English, Chinese (Simplified), and Indonesian
- Full-text search powered by Algolia DocSearch (free for open-source/public docs) or local Meilisearch
- PDF export for offline manuals
- Version-controlled documentation (each truck model/software version has its own doc version)
- Feedback widget per page ("Was this helpful?") feeding into the NestJS API

**Architecture Notes:**
Docusaurus is a static site generator. All documentation lives as Markdown files in a `/docs` repository. On each Git push, GitHub Actions rebuilds and deploys the static site to Nginx. No database required. This makes it extremely fast and highly cacheable by Cloudflare.

**File Structure:**
```
/docs-site (Docusaurus)
  /docs
    /en
      /user-manual
      /service-manual
      /parts-catalog
      /warranty
      /troubleshooting
    /zh
      /... (translated versions)
    /id
      /... (translated versions)
  /blog              ← EV technology articles, release notes
  /src/components    ← Custom React components
  /static            ← Images, diagrams
  docusaurus.config.js
```

---

### Component 2: Marketing & Knowledge Hub (Next.js)

**Purpose:** Public-facing website for lead generation, brand authority, and EV adoption education.

**Key Features:**
- Case studies and success story pages (SEO-optimized)
- EV technology articles and fleet electrification guides
- Vehicle product pages
- Lead capture forms (integrated with CRM via API)
- Newsletter signup
- "Request a Demo" flow connecting to NestJS API
- Structured data markup (JSON-LD) for SEO

**Architecture Notes:**
This can be built as part of the main Next.js application using route groups, or as a completely separate Next.js project. For a solo founder, the recommended approach is to use a **single Next.js monorepo** with route groups: `/(marketing)` and `/(app)`. This avoids maintaining two separate deployment pipelines while keeping the code logically separated.

**Content Management:**
Two options depending on founder preference:

1. **File-based (simpler):** Case studies and articles as MDX files in the repository. Edit in Git. No CMS dashboard needed initially.
2. **Headless CMS (recommended after Month 4):** Integrate Payload CMS (self-hosted, TypeScript-native, runs alongside NestJS) to allow non-technical team members to publish content without GitHub access.

---

### Component 3: Aftersales Portal (Next.js + NestJS)

**Purpose:** Authenticated platform for customers, service partners, and internal engineers to manage vehicles, warranties, parts, and service tickets.

**User Roles:**

| Role | Permissions |
|---|---|
| `CUSTOMER` | View own vehicles, warranties, service history, parts catalog, open tickets |
| `SERVICE_PARTNER` | View assigned customer vehicles, create/update service tickets, access service manuals |
| `INTERNAL_ENGINEER` | Full access to all vehicles, diagnostic data, warranty approval/rejection |
| `ADMIN` | User management, system configuration, reporting |

**Sub-modules:**

**a) Vehicle Registry**
- Customers register their trucks using VIN (Vehicle Identification Number)
- Stores: VIN, model, purchase date, current mileage, assigned service partner
- Validation: VIN format check, optional OEM verification hook for Phase 2

**b) Warranty Management**
- Warranty records linked to vehicles and components
- Warranty claim submission with photo/document attachment
- Status workflow: `SUBMITTED → UNDER_REVIEW → APPROVED/REJECTED → RESOLVED`
- Email notifications at each status change via BullMQ + Resend
- Warranty expiry alerts (cron job, 30/60/90 days before expiry)

**c) Parts Catalog**
- Searchable, filterable parts database
- Part relationships: compatible vehicle models, supersession chains
- Stock availability status (manual update initially; API-ready for supplier integration)
- Parts inquiry form generating a service ticket

**d) Service Ticket System**
- Customers and service partners can open tickets
- Categorized: Warranty Claim, Technical Issue, Parts Request, General Inquiry
- Internal engineers can be assigned to tickets
- Comments thread on each ticket
- File attachment support (photos of damage, fault code screenshots)

**e) Service Documentation Access**
- Role-gated documentation: service partners see service manuals, customers see user manuals
- Documents served from Object Storage via signed URLs (time-limited access)

---

### Component 4: FleetOS (Stub → Full Platform)

**Purpose:** Real-time fleet monitoring and operations platform. Designed as a stub in Phase 1, fully built in Phase 2.

**Phase 1 Stub (built now, dormant):**
- Fleet organization data model in schema (Organization → Fleet → Vehicle)
- API endpoints defined but return static/empty data
- UI page shells in Next.js (locked behind a "coming soon" gate)

**Phase 2 Full Build:**
- OBD-II / telematics gateway integration (MQTT protocol)
- Real-time dashboard: GPS location, battery SOC (State of Charge), speed, fault codes
- TimescaleDB for time-series telemetry storage
- Predictive maintenance alerts (rule-based initially, ML in Phase 3)
- Geofencing and route analysis
- Multi-tenant fleet organization (SaaS model)
- Subscription billing via Stripe

---

## 5. Database Schema Design

### Schema Design Philosophy

Use PostgreSQL with Prisma ORM. The schema is designed to be normalized but practical. Avoid over-normalization for small lookup tables. Use Prisma's enum types for status fields to get type safety at the ORM layer.

### Core Schema (Prisma format)

```prisma
// ============================================================
// PRISMA SCHEMA — EV Truck Aftersales Platform
// ============================================================

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// ─────────────────────────────────────────
// ENUMS
// ─────────────────────────────────────────

enum UserRole {
  CUSTOMER
  SERVICE_PARTNER
  INTERNAL_ENGINEER
  ADMIN
}

enum WarrantyStatus {
  ACTIVE
  EXPIRED
  VOIDED
}

enum ClaimStatus {
  SUBMITTED
  UNDER_REVIEW
  APPROVED
  REJECTED
  RESOLVED
}

enum TicketCategory {
  WARRANTY_CLAIM
  TECHNICAL_ISSUE
  PARTS_REQUEST
  GENERAL_INQUIRY
}

enum TicketStatus {
  OPEN
  IN_PROGRESS
  AWAITING_CUSTOMER
  RESOLVED
  CLOSED
}

enum PartAvailability {
  IN_STOCK
  LOW_STOCK
  OUT_OF_STOCK
  DISCONTINUED
}

// ─────────────────────────────────────────
// USER & ORGANIZATION
// ─────────────────────────────────────────

model Organization {
  id          String    @id @default(cuid())
  name        String
  country     String
  phone       String?
  email       String?
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt

  users       User[]
  vehicles    Vehicle[]
  fleets      Fleet[]
}

model User {
  id             String       @id @default(cuid())
  email          String       @unique
  passwordHash   String
  firstName      String
  lastName       String
  role           UserRole     @default(CUSTOMER)
  isActive       Boolean      @default(true)
  emailVerified  Boolean      @default(false)
  language       String       @default("en") // en, zh, id

  organizationId String?
  organization   Organization? @relation(fields: [organizationId], references: [id])

  refreshTokens  RefreshToken[]
  vehicles       VehicleOwner[]
  tickets        Ticket[]       @relation("TicketCreator")
  assignedTickets Ticket[]      @relation("TicketAssignee")
  ticketComments TicketComment[]
  warrantyClaims WarrantyClaim[]

  createdAt      DateTime     @default(now())
  updatedAt      DateTime     @updatedAt
}

model RefreshToken {
  id        String   @id @default(cuid())
  token     String   @unique
  userId    String
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  expiresAt DateTime
  createdAt DateTime @default(now())
}

// ─────────────────────────────────────────
// VEHICLE
// ─────────────────────────────────────────

model VehicleModel {
  id              String    @id @default(cuid())
  name            String    // e.g., "ET-500 Pro"
  manufacturer    String
  category        String    // e.g., "Heavy Duty", "Light Commercial"
  batteryCapacityKwh Float?
  rangeKm         Float?
  payloadKg       Float?
  grossWeightKg   Float?
  chargePortType  String?
  imageUrl        String?
  isActive        Boolean   @default(true)

  vehicles        Vehicle[]
  compatibleParts PartCompatibility[]

  createdAt       DateTime  @default(now())
  updatedAt       DateTime  @updatedAt
}

model Vehicle {
  id              String       @id @default(cuid())
  vin             String       @unique
  registrationNo  String?
  purchaseDate    DateTime?
  deliveryDate    DateTime?
  mileageKm       Float        @default(0)

  modelId         String
  model           VehicleModel @relation(fields: [modelId], references: [id])

  organizationId  String?
  organization    Organization? @relation(fields: [organizationId], references: [id])

  fleetId         String?
  fleet           Fleet?       @relation(fields: [fleetId], references: [id])

  owners          VehicleOwner[]
  warranties      Warranty[]
  tickets         Ticket[]

  // FleetOS fields (Phase 2)
  telemetryEnabled Boolean   @default(false)
  lastSeenAt      DateTime?

  createdAt       DateTime   @default(now())
  updatedAt       DateTime   @updatedAt
}

model VehicleOwner {
  id        String   @id @default(cuid())
  userId    String
  vehicleId String
  isPrimary Boolean  @default(true)
  since     DateTime @default(now())

  user      User     @relation(fields: [userId], references: [id])
  vehicle   Vehicle  @relation(fields: [vehicleId], references: [id])

  @@unique([userId, vehicleId])
}

// ─────────────────────────────────────────
// WARRANTY
// ─────────────────────────────────────────

model WarrantyType {
  id            String     @id @default(cuid())
  name          String     // e.g., "Battery Warranty", "Drivetrain Warranty"
  description   String?
  durationMonths Int
  maxMileageKm  Float?
  coverageDetails String?  // JSON or Markdown string

  warranties    Warranty[]
}

model Warranty {
  id              String         @id @default(cuid())
  vehicleId       String
  vehicle         Vehicle        @relation(fields: [vehicleId], references: [id])
  warrantyTypeId  String
  warrantyType    WarrantyType   @relation(fields: [warrantyTypeId], references: [id])
  startDate       DateTime
  endDate         DateTime
  status          WarrantyStatus @default(ACTIVE)
  notes           String?

  claims          WarrantyClaim[]

  createdAt       DateTime       @default(now())
  updatedAt       DateTime       @updatedAt
}

model WarrantyClaim {
  id            String      @id @default(cuid())
  warrantyId    String
  warranty      Warranty    @relation(fields: [warrantyId], references: [id])
  submittedById String
  submittedBy   User        @relation(fields: [submittedById], references: [id])

  title         String
  description   String
  mileageAtClaim Float?
  status        ClaimStatus @default(SUBMITTED)
  resolution    String?
  attachments   Attachment[]

  createdAt     DateTime    @default(now())
  updatedAt     DateTime    @updatedAt
}

// ─────────────────────────────────────────
// PARTS CATALOG
// ─────────────────────────────────────────

model PartCategory {
  id        String  @id @default(cuid())
  name      String
  slug      String  @unique
  parentId  String?
  parent    PartCategory?  @relation("CategoryTree", fields: [parentId], references: [id])
  children  PartCategory[] @relation("CategoryTree")
  parts     Part[]
}

model Part {
  id              String           @id @default(cuid())
  partNumber      String           @unique
  name            String
  description     String?
  categoryId      String
  category        PartCategory     @relation(fields: [categoryId], references: [id])

  unitPrice       Float?
  currency        String           @default("USD")
  availability    PartAvailability @default(IN_STOCK)
  weight          Float?
  imageUrl        String?

  // Supersession chain
  supersededById  String?
  supersededBy    Part?            @relation("Supersession", fields: [supersededById], references: [id])
  supersedes      Part[]           @relation("Supersession")

  compatibleModels PartCompatibility[]

  createdAt       DateTime         @default(now())
  updatedAt       DateTime         @updatedAt
}

model PartCompatibility {
  id           String       @id @default(cuid())
  partId       String
  part         Part         @relation(fields: [partId], references: [id])
  modelId      String
  vehicleModel VehicleModel @relation(fields: [modelId], references: [id])
  notes        String?

  @@unique([partId, modelId])
}

// ─────────────────────────────────────────
// SERVICE TICKETS
// ─────────────────────────────────────────

model Ticket {
  id           String         @id @default(cuid())
  ticketNo     String         @unique // e.g., TKT-2026-00142
  title        String
  description  String
  category     TicketCategory
  status       TicketStatus   @default(OPEN)
  priority     Int            @default(2) // 1=Critical, 2=High, 3=Medium, 4=Low

  vehicleId    String?
  vehicle      Vehicle?       @relation(fields: [vehicleId], references: [id])

  createdById  String
  createdBy    User           @relation("TicketCreator", fields: [createdById], references: [id])

  assigneeId   String?
  assignee     User?          @relation("TicketAssignee", fields: [assigneeId], references: [id])

  comments     TicketComment[]
  attachments  Attachment[]

  resolvedAt   DateTime?
  createdAt    DateTime       @default(now())
  updatedAt    DateTime       @updatedAt
}

model TicketComment {
  id          String   @id @default(cuid())
  ticketId    String
  ticket      Ticket   @relation(fields: [ticketId], references: [id], onDelete: Cascade)
  authorId    String
  author      User     @relation(fields: [authorId], references: [id])
  body        String
  isInternal  Boolean  @default(false) // Internal notes not visible to customer
  attachments Attachment[]
  createdAt   DateTime @default(now())
}

// ─────────────────────────────────────────
// SHARED / UTILITY
// ─────────────────────────────────────────

model Attachment {
  id          String  @id @default(cuid())
  filename    String
  url         String  // Object storage URL
  mimeType    String
  sizeBytes   Int

  claimId     String?
  claim       WarrantyClaim? @relation(fields: [claimId], references: [id])
  ticketId    String?
  ticket      Ticket?        @relation(fields: [ticketId], references: [id])
  commentId   String?
  comment     TicketComment? @relation(fields: [commentId], references: [id])

  createdAt   DateTime @default(now())
}

// ─────────────────────────────────────────
// FLEET (Phase 2 Stub)
// ─────────────────────────────────────────

model Fleet {
  id             String       @id @default(cuid())
  name           String
  organizationId String
  organization   Organization @relation(fields: [organizationId], references: [id])
  vehicles       Vehicle[]
  createdAt      DateTime     @default(now())
  updatedAt      DateTime     @updatedAt
}

// Phase 2: VehicleTelemetry — will migrate to TimescaleDB hypertable
// model VehicleTelemetry {
//   id           String   @id @default(cuid())
//   vehicleId    String
//   timestamp    DateTime
//   soc          Float    // State of charge (%)
//   speedKmh     Float?
//   odometer     Float?
//   latitude     Float?
//   longitude    Float?
//   faultCodes   String[] // Array of active DTCs
//   ...
// }
```

---

## 6. API Architecture

### Design Philosophy: REST with Pragmatic Conventions

REST is preferred over GraphQL for this project. The reasons are: simpler to implement in NestJS, better tooling support for Swagger auto-generation, easier for AI coding tools to generate correct code, and more than sufficient for the complexity of this domain. GraphQL becomes valuable when frontend teams have diverse data shape requirements — not relevant for a solo founder's initial build.

### Base URL Structure

```
https://api.yourdomain.com/v1/
```

All API endpoints are versioned from day one. This costs nothing now and prevents breaking changes later.

### Authentication Endpoints

```
POST   /v1/auth/register          Register new user
POST   /v1/auth/login             Login, returns JWT + refresh token
POST   /v1/auth/refresh           Refresh access token
POST   /v1/auth/logout            Invalidate refresh token
POST   /v1/auth/forgot-password   Send password reset email
POST   /v1/auth/reset-password    Reset password with token
GET    /v1/auth/me                Get current user profile
```

### Vehicle Endpoints

```
GET    /v1/vehicles               List user's registered vehicles
POST   /v1/vehicles               Register a new vehicle (by VIN)
GET    /v1/vehicles/:id           Get vehicle details
PATCH  /v1/vehicles/:id           Update vehicle mileage, etc.
GET    /v1/vehicles/:id/warranty  Get warranties for a vehicle
GET    /v1/vehicles/:id/tickets   Get service tickets for a vehicle
GET    /v1/vehicles/:id/history   Get full service history
```

### Warranty Endpoints

```
GET    /v1/warranties             List warranties (scoped to user's vehicles)
GET    /v1/warranties/:id         Get warranty details
POST   /v1/warranties/:id/claims  Submit a warranty claim
GET    /v1/warranties/:id/claims  List claims for a warranty
PATCH  /v1/warranties/claims/:id  Update claim status (ENGINEER role)
```

### Parts Catalog Endpoints

```
GET    /v1/parts                  List/search parts (with filters: category, model, availability)
GET    /v1/parts/:id              Get part details
GET    /v1/parts/categories       Get category tree
GET    /v1/parts/:id/compatible   Get compatible vehicle models
POST   /v1/parts/:id/inquiry      Submit parts inquiry (creates ticket)
```

### Ticket Endpoints

```
GET    /v1/tickets                List tickets (scoped by role)
POST   /v1/tickets                Create new ticket
GET    /v1/tickets/:id            Get ticket details + comments
PATCH  /v1/tickets/:id            Update ticket (status, assignee, priority)
POST   /v1/tickets/:id/comments   Add comment to ticket
POST   /v1/tickets/:id/attachments Upload attachment to ticket
```

### Admin Endpoints

```
GET    /v1/admin/users            List all users
PATCH  /v1/admin/users/:id        Update user role/status
GET    /v1/admin/vehicles         List all vehicles
GET    /v1/admin/reports/warranties  Warranty claim statistics
GET    /v1/admin/reports/tickets     Ticket resolution metrics
```

### API Response Format

All API responses follow a consistent envelope format:

```typescript
// Success response
{
  "success": true,
  "data": { ... },          // or []
  "meta": {                 // for paginated responses
    "page": 1,
    "limit": 20,
    "total": 154,
    "totalPages": 8
  }
}

// Error response
{
  "success": false,
  "error": {
    "code": "VEHICLE_NOT_FOUND",
    "message": "No vehicle found with the provided VIN",
    "statusCode": 404
  }
}
```

### NestJS Module Structure

```
/src
  /modules
    /auth
      auth.module.ts
      auth.controller.ts
      auth.service.ts
      strategies/
        jwt.strategy.ts
        local.strategy.ts
      dto/
        login.dto.ts
        register.dto.ts
    /users
      users.module.ts
      users.controller.ts
      users.service.ts
    /vehicles
      vehicles.module.ts
      vehicles.controller.ts
      vehicles.service.ts
    /warranty
      warranty.module.ts
      warranty.controller.ts
      warranty.service.ts
    /parts
      parts.module.ts
      parts.controller.ts
      parts.service.ts
    /tickets
      tickets.module.ts
      tickets.controller.ts
      tickets.service.ts
    /fleet           ← Phase 2 stub
      fleet.module.ts
  /common
    /guards
      jwt-auth.guard.ts
      roles.guard.ts
    /decorators
      roles.decorator.ts
      current-user.decorator.ts
    /interceptors
      logging.interceptor.ts
      transform.interceptor.ts    ← Wraps all responses in envelope
    /filters
      http-exception.filter.ts
    /pipes
      validation.pipe.ts
  /prisma
    prisma.module.ts
    prisma.service.ts
  /config
    app.config.ts
    database.config.ts
    jwt.config.ts
  app.module.ts
  main.ts
```

---

## 7. Documentation Architecture

### Product Line Overview

The T7 series spans a **matrix of platform variants × body configurations**. A naive approach of writing one manual per product combination produces 20+ documents with massive duplication. Instead, the architecture uses a **three-layer content composition model** that separates shared content from platform-specific and body-specific content.

**Platform variants (differ by battery capacity and vehicle dimensions):**

| Model | Battery | Notes |
|---|---|---|
| T7-90 | 90 kWh | Standard wheelbase |
| T7-90L | 90 kWh | Long wheelbase |
| T7-134L | 134 kWh | Long wheelbase, larger pack |
| T7-169L | 169 kWh | Long wheelbase, largest pack |

**Body configurations (superstructure mounted on any platform):**

Arm Roll Truck · Dumper Truck · Compactor Truck · Sweeper Truck · Standard (flatbed / cargo)

---

### The Three-Layer Content Model

Every piece of documentation content belongs to exactly one layer. This prevents duplication and means a single edit propagates everywhere it applies.

```
Layer 1 — _shared/          → applies to ALL T7 trucks regardless of battery or body
Layer 2 — platforms/        → applies to ONE battery/platform variant across all bodies
Layer 3 — applications/     → applies to ONE body type across all platforms
```

**Decision rule for content placement:**

| Question | If YES → place in |
|---|---|
| Applies to all T7 trucks regardless of battery or body type? | `_shared/_partials/` |
| Applies specifically to the 134kWh pack (all body types)? | `platforms/t7-134l/` |
| Applies to all dumper trucks regardless of battery size? | `applications/dumper/` |
| Unique to one specific combination (e.g., T7-134L Dumper)? | `manuals/t7-134l-dumper/` assembly file |

In practice: ~70% of content lands in `_shared`, ~20% in `platforms`, ~10% in `applications` or assembly files.

---

### File Structure

```
/docs-site (Docusaurus root)
│
├── docs/
│   │
│   ├── _shared/                        ← LAYER 1: Shared across ALL T7 trucks
│   │   ├── _partials/
│   │   │   ├── hv-safety.mdx           ← High voltage safety (identical all models)
│   │   │   ├── charging-guide.mdx      ← Charging procedure (same connector/process)
│   │   │   ├── daily-inspection.mdx    ← Pre-trip checklist
│   │   │   ├── cab-controls.mdx        ← Dashboard controls (same cab all models)
│   │   │   ├── warranty-overview.mdx   ← Warranty terms (same policy all models)
│   │   │   ├── range-table.mdx         ← MDX component with batteryKwh/range props
│   │   │   └── spec-table.mdx          ← MDX component with vehicle spec props
│   │   └── components/
│   │       ├── drivetrain.mdx          ← Motor + inverter (same unit all T7)
│   │       ├── braking-system.mdx      ← Regenerative + air brakes
│   │       └── cooling-system.mdx      ← Thermal management
│   │
│   ├── platforms/                      ← LAYER 2: Per battery/platform variant
│   │   ├── t7-90/
│   │   │   ├── specifications.mdx      ← 90kWh specs, dimensions, GVW
│   │   │   ├── battery-system.mdx      ← 90kWh pack chemistry, BMS detail
│   │   │   └── range-charging.mdx      ← Range curves specific to 90kWh
│   │   ├── t7-90l/
│   │   │   ├── specifications.mdx      ← Same battery, longer chassis dimensions
│   │   │   ├── battery-system.mdx      ← Same as t7-90 (MDX import reuse)
│   │   │   └── range-charging.mdx
│   │   ├── t7-134l/
│   │   │   ├── specifications.mdx
│   │   │   ├── battery-system.mdx      ← 134kWh specific: different cell chemistry
│   │   │   └── range-charging.mdx
│   │   └── t7-169l/
│   │       ├── specifications.mdx
│   │       ├── battery-system.mdx      ← 169kWh largest pack, thermal notes
│   │       └── range-charging.mdx
│   │
│   ├── applications/                   ← LAYER 3: Per body/superstructure type
│   │   ├── arm-roll/
│   │   │   ├── overview.mdx
│   │   │   ├── operation.mdx           ← Arm roll mechanism operation
│   │   │   ├── maintenance.mdx         ← Body-specific service intervals
│   │   │   └── safety.mdx              ← Container handling safety
│   │   ├── dumper/
│   │   │   ├── overview.mdx
│   │   │   ├── operation.mdx
│   │   │   ├── maintenance.mdx
│   │   │   └── safety.mdx
│   │   ├── compactor/
│   │   │   ├── overview.mdx
│   │   │   ├── operation.mdx
│   │   │   └── maintenance.mdx
│   │   └── sweeper/
│   │       ├── overview.mdx
│   │       ├── operation.mdx
│   │       └── maintenance.mdx
│   │
│   ├── manuals/                        ← ASSEMBLED product manuals (thin import files)
│   │   ├── t7-90-standard/
│   │   │   └── user-manual.mdx
│   │   ├── t7-90-arm-roll/
│   │   │   └── user-manual.mdx
│   │   ├── t7-134l-dumper/
│   │   │   ├── user-manual.mdx
│   │   │   └── service-manual.mdx
│   │   └── ...                         ← One folder per product combination
│   │
│   └── shared-reference/               ← Cross-cutting reference docs
│       ├── fault-codes.mdx             ← DTC library (links into troubleshooting)
│       ├── parts-numbering.mdx
│       └── warranty-claim-process.mdx
│
├── i18n/
│   ├── zh/
│   │   └── docusaurus-plugin-content-docs/current/
│   │       ├── _shared/_partials/      ← Translated shared partials
│   │       ├── platforms/              ← Translated platform specs
│   │       └── applications/          ← Translated body operation guides
│   └── id/
│       └── ...                         ← Same structure for Indonesian
│
└── docusaurus.config.js
```

---

### How Assembly Works (MDX Composition)

Each final product manual is a thin file that imports from the three layers. The reader gets a complete, coherent document; you wrote the content once.

```mdx
---
# docs/manuals/t7-134l-dumper/user-manual.mdx
title: "T7-134L Dumper Truck — User Manual"
sidebar_label: "User Manual"
---

import HVSafety      from '@site/docs/_shared/_partials/hv-safety.mdx';
import ChargingGuide from '@site/docs/_shared/_partials/charging-guide.mdx';
import CabControls   from '@site/docs/_shared/_partials/cab-controls.mdx';
import { SpecTable } from '@site/docs/_shared/_partials/spec-table.mdx';
import BatterySystem from '@site/docs/platforms/t7-134l/battery-system.mdx';
import DumperOps     from '@site/docs/applications/dumper/operation.mdx';
import DumperSafety  from '@site/docs/applications/dumper/safety.mdx';

## Safety — High Voltage Systems
<HVSafety />

## Vehicle Specifications
<SpecTable
  batteryKwh={134}
  rangeKm={280}
  gvwKg={18000}
  lengthMm={8500}
  widthMm={2500}
/>

## Your Battery System
<BatterySystem />

## Cab Controls & Dashboard
<CabControls />

## Charging Your Truck
<ChargingGuide />

## Operating the Dumper Body
<DumperSafety />
<DumperOps />
```

### Handling Spec Variations with Typed Props

For content that shares structure but differs in numbers, use MDX components with props:

```mdx
<!-- docs/_shared/_partials/spec-table.mdx -->
export const SpecTable = ({ batteryKwh, rangeKm, gvwKg, lengthMm, widthMm }) => (
  <table className="spec-table">
    <tr><td><strong>Battery Capacity</strong></td><td>{batteryKwh} kWh</td></tr>
    <tr><td><strong>Estimated Range</strong></td><td>{rangeKm} km</td></tr>
    <tr><td><strong>Gross Vehicle Weight</strong></td><td>{(gvwKg/1000).toFixed(1)} t</td></tr>
    <tr><td><strong>Overall Length</strong></td><td>{lengthMm} mm</td></tr>
    <tr><td><strong>Overall Width</strong></td><td>{widthMm} mm</td></tr>
  </table>
);
```

---

### Docusaurus Sidebar Structure (Customer View)

The sidebar is organised by platform first, then body type. Customers navigate to their specific truck and get a complete manual assembled from shared content — they never see the shared layer.

```javascript
// sidebars.js
module.exports = {
  docs: [
    {
      type: 'category', label: 'T7-90 (90 kWh)',
      items: [
        'manuals/t7-90-standard/user-manual',
        {
          type: 'category', label: 'Arm Roll',
          items: ['manuals/t7-90-arm-roll/user-manual']
        },
        {
          type: 'category', label: 'Dumper',
          items: ['manuals/t7-90-dumper/user-manual']
        },
      ],
    },
    {
      type: 'category', label: 'T7-134L (134 kWh)',
      items: [
        'manuals/t7-134l-standard/user-manual',
        {
          type: 'category', label: 'Arm Roll',
          items: ['manuals/t7-134l-arm-roll/user-manual', 'manuals/t7-134l-arm-roll/service-manual']
        },
        {
          type: 'category', label: 'Dumper',
          items: ['manuals/t7-134l-dumper/user-manual', 'manuals/t7-134l-dumper/service-manual']
        },
        // ... compactor, sweeper
      ],
    },
    // ... t7-90l, t7-169l
    {
      type: 'category', label: 'Reference',
      items: ['shared-reference/fault-codes', 'shared-reference/warranty-claim-process'],
    },
  ],
};
```

---

### Multi-Language Strategy

Docusaurus ships with a **native `localeDropdown` navbar item** that renders a language switcher automatically — no plugins needed. When a user switches language, they are taken to the same page in the new locale, not back to the homepage.

```javascript
// docusaurus.config.js
module.exports = {
  i18n: {
    defaultLocale: 'en',
    locales: ['en', 'zh', 'id'],
    localeConfigs: {
      en: { label: 'English',           direction: 'ltr' },
      zh: { label: '中文',              direction: 'ltr' },
      id: { label: 'Bahasa Indonesia',  direction: 'ltr' },
    },
  },
  themeConfig: {
    navbar: {
      items: [
        // ... other nav items
        {
          type: 'localeDropdown',   // ← renders 🌐 English ▾  /  中文  /  Indonesia
          position: 'right',
        },
      ],
    },
  },
};
```

**URL routing by locale:**

| Language | Example URL |
|---|---|
| English (default) | `docs.yourdomain.com/docs/manuals/t7-134l-dumper/user-manual` |
| Chinese | `docs.yourdomain.com/zh/docs/manuals/t7-134l-dumper/user-manual` |
| Indonesian | `docs.yourdomain.com/id/docs/manuals/t7-134l-dumper/user-manual` |

**Translation workflow with Claude Code:**

Only the `_shared/_partials` and `applications/` files need translating — not every assembled manual file. When a partial is updated in English, run `npm run docusaurus write-translations` to flag all stale translations across all languages automatically.

```bash
# Generate translation key files for each locale
npm run docusaurus write-translations -- --locale zh
npm run docusaurus write-translations -- --locale id
```

Use this Claude Code prompt to translate a partial while preserving MDX:

```
Read docs/_shared/_partials/charging-guide.mdx.
Translate all prose to Chinese Simplified.
Preserve every MDX component tag (e.g., <SpecTable />, import statements,
frontmatter YAML) exactly as-is — only translate the surrounding text.
Write the result to i18n/zh/docusaurus-plugin-content-docs/current/_shared/_partials/charging-guide.mdx
```

---

### Gated Documentation (Service Manuals)

Service Manuals contain proprietary repair procedures and must not be publicly accessible.

- Service Manual pages are built into Docusaurus but wrapped in a custom React component that checks for a valid JWT cookie before rendering content.
- If not authenticated as `SERVICE_PARTNER` or `INTERNAL_ENGINEER`, the wrapper redirects to the portal login with a `returnUrl` parameter.
- Gating is implemented at the component level — Docusaurus remains a fully static build; no SSR or server-side auth logic required.

---

## 8. Infrastructure and Deployment Strategy

### Local Development Environment (Home Lab + Workstation)

Development runs on two machines on your local network before anything touches the cloud.

**The split:**

```
Workstation PC (Docker Compose — active development)
  ├── nextjs-app    :3000   ← hot reload, fast iteration
  ├── nestjs-api    :4000   ← watch mode, instant restart
  ├── docusaurus    :3001   ← dev server with live preview
  └── redis         :6379   ← ephemeral; fine to lose on restart

TrueNAS Scale (persistent services — always on, LAN IP e.g. 192.168.1.100)
  ├── PostgreSQL    :5432   ← data survives workstation reboots
  ├── MinIO         :9000   ← S3-compatible local object storage
  └── Meilisearch   :7700   ← search index survives reboots
```

**Why this split works:**
The workstation runs only the services that benefit from hot reload. TrueNAS hosts only the services that need persistence. Your `.env.local` points at the TrueNAS LAN IP — no VPN, no tunnelling. The production `docker-compose.prod.yml` points at DigitalOcean. The application code is identical.

**TrueNAS Scale service setup:**

TrueNAS Scale ships with a built-in Apps system (k3s-backed). The simplest approach is to run PostgreSQL, MinIO, and Meilisearch as Docker containers inside a TrueNAS jail using the Custom App option, with TrueNAS datasets mounted as volumes for persistence.

```yaml
# docker-compose.dev.yml  (runs on WORKSTATION)
version: '3.9'
services:
  app:
    build: ./frontend
    command: npm run dev
    ports: ["3000:3000"]
    volumes: ["./frontend:/app", "/app/node_modules"]
    environment:
      - NEXT_PUBLIC_API_URL=http://localhost:4000

  api:
    build: ./backend
    command: npm run start:dev
    ports: ["4000:4000"]
    volumes: ["./backend:/app", "/app/node_modules"]
    environment:
      - DATABASE_URL=postgresql://postgres:password@192.168.1.100:5432/evplatform
      - REDIS_URL=redis://localhost:6379
      - S3_ENDPOINT=http://192.168.1.100:9000
      - S3_BUCKET=evplatform-dev
      - MEILISEARCH_HOST=http://192.168.1.100:7700

  redis:
    image: redis:7-alpine
    ports: ["6379:6379"]

  docs:
    build: ./docs-site
    command: npm run start -- --host 0.0.0.0
    ports: ["3001:3001"]
    volumes: ["./docs-site:/app", "/app/node_modules"]
```

```yaml
# Services running on TrueNAS Scale (via Docker or TrueNAS Apps)
# PostgreSQL
image: postgres:16
ports: ["5432:5432"]
environment:
  POSTGRES_DB: evplatform
  POSTGRES_USER: postgres
  POSTGRES_PASSWORD: <strong-local-password>
volumes:
  - /mnt/tank/postgres-data:/var/lib/postgresql/data

# MinIO  (S3-compatible object storage)
image: minio/minio
ports: ["9000:9000", "9001:9001"]   # 9001 = web console
command: server /data --console-address ":9001"
volumes:
  - /mnt/tank/minio-data:/data
environment:
  MINIO_ROOT_USER: minioadmin
  MINIO_ROOT_PASSWORD: <strong-local-password>

# Meilisearch
image: getmeili/meilisearch:latest
ports: ["7700:7700"]
volumes:
  - /mnt/tank/meili-data:/meili_data
```

**Environment parity table:**

| Variable | Dev value | Prod value |
|---|---|---|
| `DATABASE_URL` | `postgresql://...@192.168.1.100:5432/...` | `postgresql://...@do-managed-db.../...` |
| `S3_ENDPOINT` | `http://192.168.1.100:9000` | `https://sgp1.digitaloceanspaces.com` |
| `S3_BUCKET` | `evplatform-dev` | `evplatform-prod` |
| `MEILISEARCH_HOST` | `http://192.168.1.100:7700` | `http://localhost:7700` (on server) |

---

### Server Architecture (Phase 1 — Production)

For Phase 1 production, a **two-server setup** balances cost and reliability:

```
Server 1: App Server (DigitalOcean Droplet — 4GB RAM / 2 vCPU)
  ├── Docker Compose
  │   ├── nginx (reverse proxy)
  │   ├── nextjs-app (Aftersales Portal + Marketing)
  │   ├── docusaurus (Documentation — static files)
  │   ├── nestjs-api (Backend API)
  │   └── redis (Cache + Queue)
  └── Certbot (Let's Encrypt SSL)

Server 2: Database Server (DigitalOcean Managed PostgreSQL — $15/mo)
  └── PostgreSQL 16 (Managed — automated backups, failover)

Object Storage: DigitalOcean Spaces ($5/mo)
  └── Parts images, manuals, warranty documents, attachments
```

Using **Managed PostgreSQL** is strongly recommended even for solo founders. The cost is marginal, but it provides: automated daily backups, point-in-time recovery, connection pooling (PgBouncer), and monitoring — all without ops burden.

### Docker Compose Configuration

```yaml
# docker-compose.prod.yml
version: '3.9'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/ssl:/etc/nginx/ssl
      - certbot-certs:/etc/letsencrypt
    depends_on:
      - app
      - api
    restart: always

  app:
    build:
      context: ./frontend
      dockerfile: Dockerfile.prod
    environment:
      - NEXT_PUBLIC_API_URL=https://api.yourdomain.com
      - NODE_ENV=production
    restart: always

  api:
    build:
      context: ./backend
      dockerfile: Dockerfile.prod
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
      - JWT_SECRET=${JWT_SECRET}
      - JWT_REFRESH_SECRET=${JWT_REFRESH_SECRET}
      - RESEND_API_KEY=${RESEND_API_KEY}
      - SPACES_KEY=${SPACES_KEY}
      - SPACES_SECRET=${SPACES_SECRET}
    depends_on:
      - redis
    restart: always

  redis:
    image: redis:7-alpine
    volumes:
      - redis-data:/data
    restart: always

  meilisearch:
    image: getmeili/meilisearch:latest
    volumes:
      - meili-data:/meili_data
    environment:
      - MEILI_MASTER_KEY=${MEILI_MASTER_KEY}
    restart: always

volumes:
  redis-data:
  meili-data:
  certbot-certs:
```

### Nginx Routing Configuration

```nginx
# nginx.conf (simplified)

server {
    listen 443 ssl;
    server_name yourdomain.com www.yourdomain.com;

    # Marketing + Aftersales Portal
    location / {
        proxy_pass http://app:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # Documentation Portal
    location /docs/ {
        proxy_pass http://app:3001;
    }

    # API
    location /api/ {
        proxy_pass http://api:4000;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        # Rate limiting
        limit_req zone=api_limit burst=20 nodelay;
    }
}
```

### CI/CD Pipeline (GitHub Actions)

```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Tests
        run: |
          npm ci
          npm test

      - name: Build Docker Images
        run: |
          docker build -t app ./frontend
          docker build -t api ./backend

      - name: Deploy to Server
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: deploy
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /opt/app
            git pull origin main
            docker compose -f docker-compose.prod.yml up -d --build
            docker system prune -f
```

### Domain & DNS Strategy

```
yourdomain.com           → Marketing site (Next.js)
app.yourdomain.com       → Aftersales Portal (Next.js)
docs.yourdomain.com      → Documentation Portal (Docusaurus)
api.yourdomain.com       → NestJS API
```

All managed through Cloudflare DNS with proxied records for CDN and DDoS protection.

### Backup Strategy

| Asset | Frequency | Method | Retention |
|---|---|---|---|
| PostgreSQL | Daily | DO Managed DB automated backup | 7 days |
| Object Storage (files) | Weekly | DO Spaces versioning | 30 days |
| Application code | Every push | GitHub | Unlimited |
| Redis | On restart | RDB snapshot | 1 day |

---

## 9. Security Considerations

### Authentication & Authorization

**JWT Strategy:**
- Access token: short-lived (15 minutes), stored in memory (not localStorage)
- Refresh token: long-lived (7 days), stored in httpOnly cookie — immune to XSS
- Refresh token rotation: each refresh generates a new refresh token and invalidates the old one
- Token family tracking: detect refresh token reuse (session theft indicator)

**Role-Based Access Control (RBAC):**
Implemented as a NestJS guard using the `@Roles()` decorator. Every protected endpoint declares its required role. The guard validates the JWT claim against the required role.

```typescript
@Get('service-manual')
@Roles(UserRole.SERVICE_PARTNER, UserRole.INTERNAL_ENGINEER, UserRole.ADMIN)
@UseGuards(JwtAuthGuard, RolesGuard)
async getServiceManualAccess() { ... }
```

### Input Validation

Every API endpoint uses NestJS `ValidationPipe` with `class-validator` DTOs. No raw request body reaches service code. This prevents SQL injection at the application layer (Prisma's parameterized queries prevent it at the DB layer).

### API Rate Limiting

```typescript
// main.ts
app.use(rateLimit({
  windowMs: 15 * 60 * 1000,   // 15 minutes
  max: 100,                    // 100 requests per window
  standardHeaders: true,
  legacyHeaders: false,
}));

// Stricter limit for auth endpoints
app.use('/v1/auth', rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 10,
}));
```

### File Upload Security

- File type whitelist: only `image/jpeg`, `image/png`, `image/webp`, `application/pdf`
- File size limit: 10MB per file
- Files scanned with ClamAV (via background job) before being made accessible
- Files stored in Object Storage with random UUID filenames — never original user filenames
- Access via signed URLs with 1-hour expiry — no permanent public URLs for sensitive documents

### Database Security

- PostgreSQL connection through SSL only
- Application uses a dedicated DB user with least-privilege access (no `CREATE`, `DROP` rights)
- All queries through Prisma ORM — no raw SQL in application code (except complex reporting queries with proper parameterization)
- Row-level security (RLS) considered for Phase 2 multi-tenant data isolation

### Environment & Secrets

- **Never commit `.env` files** — use Doppler for secret injection
- All secrets rotated every 90 days
- Database credentials never in application code
- Cloudflare WAF rules block common attack patterns (SQLi, XSS, RFI)

### OWASP Top 10 Checklist

| Risk | Mitigation |
|---|---|
| Broken Access Control | RBAC guards on every route, ownership checks in services |
| Cryptographic Failures | bcrypt for passwords (cost factor 12), TLS everywhere |
| Injection | Prisma parameterized queries, class-validator DTOs |
| Insecure Design | Security requirements documented before development |
| Security Misconfiguration | Docker containers run as non-root users |
| Vulnerable Components | `npm audit` in CI/CD pipeline |
| Auth Failures | Refresh token rotation, brute force rate limiting |
| Data Integrity | Signed file URLs, integrity checks on uploads |
| Logging Failures | Centralized logging, security event audit trail |
| SSRF | Whitelist for any outbound HTTP calls |

---

## 10. Scaling Strategy for FleetOS

### The Scaling Challenge

Fleet telemetry is an entirely different workload from the aftersales portal. A single vehicle can emit 1–10 data points per second. At 1,000 vehicles, that's up to 10,000 writes/second — orders of magnitude beyond what a standard PostgreSQL OLTP setup handles gracefully.

### Phased Scaling Approach

**Phase 1 (0–500 Vehicles, Months 1–12): No Telemetry**
Standard PostgreSQL handles all data. No scaling challenges.

**Phase 2 (500–5,000 Vehicles, Months 12–24): Telemetry Introduction**

Introduce **TimescaleDB** — a PostgreSQL extension that adds time-series optimization. It runs on the same server, uses the same Prisma connection, but partitions telemetry data by time chunks automatically.

Key advantage: Your DBA skills transfer completely. No new query language. No new infrastructure paradigm.

```sql
-- Convert vehicle_telemetry to a TimescaleDB hypertable
SELECT create_hypertable('vehicle_telemetry', 'timestamp');

-- Automatic compression of data older than 7 days
SELECT add_compression_policy('vehicle_telemetry', INTERVAL '7 days');

-- Automatic data retention (delete raw data older than 1 year)
SELECT add_retention_policy('vehicle_telemetry', INTERVAL '1 year');
```

**MQTT Broker for Telemetry Ingestion:**

```
Vehicle OBD Gateway → MQTT (EMQX broker) → Telemetry Microservice → TimescaleDB
```

EMQX is an open-source MQTT broker that handles millions of concurrent connections. It can be deployed as a Docker container alongside the existing stack.

**Phase 3 (5,000+ Vehicles, Month 24+): Full Microservices**

Extract the Telemetry Service into a standalone microservice. Introduce Apache Kafka as the message bus between telemetry ingestion and downstream consumers (predictive maintenance engine, alerts, dashboard aggregations).

```
Vehicles → EMQX MQTT → Kafka → [Telemetry Consumer → TimescaleDB]
                              → [Alerts Consumer → PostgreSQL + Push Notifications]
                              → [Analytics Consumer → ClickHouse (OLAP)]
```

At this scale, introduce **read replicas** for PostgreSQL to offload the fleet dashboard queries from the transactional primary.

### Multi-Tenancy Strategy for SaaS

Phase 2 converts the platform to a multi-tenant SaaS. The recommended multi-tenancy pattern for this scale is **Schema-per-tenant** using PostgreSQL schemas.

- Each fleet organization gets its own PostgreSQL schema: `tenant_acme_logistics`, `tenant_green_fleet`
- Shared tables (parts catalog, vehicle models) stay in the `public` schema
- Prisma handles schema switching via a middleware layer
- This provides strong data isolation without the operational complexity of separate databases per tenant

**SaaS Tiers (suggested):**

| Tier | Vehicles | Features | Price |
|---|---|---|---|
| Starter | Up to 10 | Aftersales portal, warranty, basic tickets | $99/mo |
| Growth | Up to 100 | + Real-time telemetry, fleet dashboard | $499/mo |
| Scale | Up to 1,000 | + Predictive maintenance, API access | $1,999/mo |
| Enterprise | Unlimited | Custom deployment, SLA, dedicated support | Custom |

---

## 11. Development Roadmap (for a Solo Founder)

### Guiding Principle

**Don't build what you can't sell yet.** Each sprint should produce something a customer can see and react to. Ship working software every 2 weeks. The roadmap is intentionally opinionated — the most common mistake for technical founders is building internal scaffolding instead of customer-facing value.

---

### Sprint 0 — Foundation (Week 1–2)
**Goal: Working skeleton deployed to production**

- [ ] Initialize monorepo with Turborepo (frontend + backend + docs in one repo)
- [ ] Set up Next.js app with TailwindCSS + shadcn/ui
- [ ] Set up NestJS with Prisma, PostgreSQL, Redis
- [ ] Set up Docusaurus with i18n configured
- [ ] Deploy all three to DigitalOcean via Docker Compose
- [ ] Configure Cloudflare DNS, SSL certificates
- [ ] Set up GitHub Actions CI/CD
- [ ] Set up Doppler for secrets

**Deliverable:** "Hello World" versions of all three apps running at their respective domains.

---

### Sprint 1 — Auth & User Foundation (Week 3–4)
**Goal: Users can create accounts and log in**

- [ ] Implement full auth flow: register, login, refresh, logout
- [ ] Email verification with Resend
- [ ] User profile page
- [ ] Basic RBAC setup
- [ ] Admin user management page
- [ ] Password reset flow

**Deliverable:** Working authentication system. Can demo user registration and login.

---

### Sprint 2 — Vehicle Registry (Week 5–6)
**Goal: Customers can register and view their vehicles**

- [ ] Vehicle registration form (VIN input)
- [ ] Vehicle model seed data (your truck models)
- [ ] Vehicle detail page (specs, purchase info)
- [ ] Vehicle list page in dashboard
- [ ] Mileage update feature

**Deliverable:** Customer can log in and register their electric truck.

---

### Sprint 3 — Documentation Portal (Week 7–8)
**Goal: Publish first real documentation content**

- [ ] Write User Manual (English) for your primary truck model
- [ ] Write Troubleshooting Guide (top 20 common issues)
- [ ] Write Warranty Overview
- [ ] Set up Docusaurus versioning for first model
- [ ] Implement search
- [ ] Translate to Chinese and Indonesian (machine translation + review)

**Deliverable:** Public documentation portal with real content. This can be shared with first customers immediately.

---

### Sprint 4 — Warranty System (Week 9–10)
**Goal: End-to-end warranty tracking**

- [ ] Warranty records linked to vehicles (seed data on vehicle registration)
- [ ] Warranty status dashboard on vehicle page
- [ ] Warranty claim submission form + file upload
- [ ] Email notifications to customer + engineer on claim submission
- [ ] Engineer review UI for claims
- [ ] Claim status update + customer email notification

**Deliverable:** Customer can view warranties and submit claims. Engineers can review claims.

---

### Sprint 5 — Parts Catalog (Week 11–12)
**Goal: Searchable parts database**

- [ ] Parts catalog database schema + seed data
- [ ] Parts list page with filters (category, vehicle model, availability)
- [ ] Parts detail page
- [ ] Parts search with Meilisearch
- [ ] "Request this part" form (creates service ticket)
- [ ] Admin: parts CRUD interface

**Deliverable:** Customers can browse and search the parts catalog.

---

### Sprint 6 — Service Tickets (Week 13–14)
**Goal: Full service ticket workflow**

- [ ] Ticket creation form (linked to vehicle)
- [ ] Ticket list for customers (own tickets) and engineers (all assigned)
- [ ] Ticket detail page with comment thread
- [ ] File attachment on tickets and comments
- [ ] Ticket assignment to engineers
- [ ] Email notifications on status changes

**Deliverable:** Complete service communication loop between customer and support team.

---

### Sprint 7 — Marketing Hub (Week 15–16)
**Goal: Lead-generating public content hub**

- [ ] Marketing homepage (hero, features, social proof)
- [ ] Vehicle product pages
- [ ] Case studies section (first 2–3 case studies as MDX files)
- [ ] EV education articles (first 5 articles)
- [ ] Lead capture form + Resend email sequence
- [ ] Basic SEO: meta tags, sitemap, robots.txt, JSON-LD

**Deliverable:** Public-facing marketing site. Platform can now be shared publicly.

---

### Sprint 8 — Polish & Launch (Week 17–18)
**Goal: Platform ready for first real customers**

- [ ] Full mobile responsiveness pass
- [ ] Accessibility audit (WCAG 2.1 AA basics)
- [ ] Performance audit (Lighthouse score > 85)
- [ ] Error monitoring setup (Sentry)
- [ ] Analytics setup (Plausible — privacy-friendly, GDPR compliant)
- [ ] Monitoring dashboard (Grafana + Prometheus)
- [ ] Security audit (check all endpoints, test rate limiting)
- [ ] First customer onboarding

**Deliverable:** Production-ready platform. First customers onboarded.

---

### Phase 2 Sprints (Month 5+): FleetOS

Sprint 9–10: Telemetry infrastructure (MQTT, TimescaleDB)
Sprint 11–12: Fleet dashboard (real-time vehicle status)
Sprint 13–14: Fault code monitoring + alerts
Sprint 15–16: Predictive maintenance (rule-based)
Sprint 17–18: Multi-tenant SaaS + Stripe billing
Sprint 19–20: Mobile app (React Native)

---

## 12. AI-Assisted Development Workflow

### The AI-Augmented Solo Developer

With the right AI workflow, a single developer can produce output equivalent to a 3–4 person team. The key is structuring your work so that AI tools maximize leverage — writing boilerplate, scaffolding, tests, and documentation — while you focus on architecture decisions and business logic.

### Recommended AI Tools

The primary AI coding tool for this project is **Claude Code** — Anthropic's agentic CLI tool that operates directly in your terminal, reads your entire codebase, writes files, runs commands, and executes multi-step coding tasks autonomously.

| Task | Tool | Why |
|---|---|---|
| Code generation, refactoring, debugging | **Claude Code** (primary) | Runs in terminal; reads whole repo; writes files and runs tests autonomously; no copy-paste required |
| Architecture & design decisions | **Claude (chat/Projects)** | Long context; maintains architectural context across planning sessions |
| Frontend component generation | **v0.dev → Claude Code** | v0 generates visual component; Claude Code wires it to API and adds types |
| Database schemas + migrations | **Claude Code** | Generates complete Prisma schema, migration files, and seed scripts from natural language |
| Test generation | **Claude Code** | Pass it a service file; it writes full Jest test suite autonomously |
| Documentation translation | **Claude Code** | Translates MDX partials to ZH/ID while preserving component tags |
| API scaffolding | **Claude Code** | Give it the Prisma model; it generates controller/service/DTO/guard/test in one command |
| Debugging | **Claude Code** | Point it at failing tests or error logs; it diagnoses and patches autonomously |

### AI-Assisted Development Patterns

**Pattern 1: Schema-First Code Generation**

Write your Prisma schema first (AI can help). Then prompt AI to generate the complete NestJS module for each model:

```
Prompt: "Given this Prisma schema for the Warranty model [paste schema],
generate a complete NestJS module including:
- warranty.module.ts
- warranty.controller.ts with REST endpoints
- warranty.service.ts with business logic
- DTOs for create, update, and response
- Swagger decorators on all endpoints
- Role guards (CUSTOMER can read own, ENGINEER can manage all)
Follow the existing pattern in my users module [paste users module]"
```

**Pattern 2: Component-First UI Development**

Use v0.dev to generate the UI component visually, then hand it to Claude Code to wire to your API:

```
v0.dev prompt: "Create a warranty claim submission form with:
title input, description textarea, mileage number input,
file upload for photos (multiple), submit button.
Use shadcn/ui components. Show validation errors inline."
```

Then in Claude Code: `"Connect this form component to POST /v1/warranties/:id/claims using React Hook Form and TanStack Query. Handle loading, success, and error states. Add Zod validation matching the CreateClaimDto on the backend."`

**Pattern 3: Test-Driven with AI**

Write a failing test (or ask AI to write it), then implement until it passes:

```
Prompt: "Write Jest unit tests for this warranty service
[paste service]. Cover: happy path claim submission,
claim on expired warranty (should throw),
claim on another user's warranty (should throw ForbiddenException)."
```

**Pattern 4: Architecture Decision Records (ADRs)**

Before starting each sprint, create a short ADR document (AI-assisted) that records why you made each key decision. This serves as context for future AI prompts and future team members:

```
ADR-005: Parts Catalog Search Strategy
Status: Accepted
Context: Need full-text search for parts catalog
Decision: Self-hosted Meilisearch over PostgreSQL full-text search
Reasoning: Better relevance ranking, typo tolerance, faceted filtering,
           easy to self-host, excellent Node.js client
Consequences: Additional Docker container, ~200MB RAM overhead
```

**Pattern 5: Claude Code Rules (CLAUDE.md)**

Create a `CLAUDE.md` file in your repo root. Claude Code reads this file automatically at the start of every session — it is the equivalent of `.cursorrules` but native to Claude Code. Include your architecture conventions, file patterns, and project context:

```markdown
# CLAUDE.md — EV Truck Platform

## Project Overview
NestJS + Next.js + Docusaurus monorepo for an EV truck aftersales and fleet platform.
Stack: NestJS API (port 4000), Next.js App Router (port 3000), Docusaurus docs (port 3001).
Database: PostgreSQL via Prisma ORM. Dev DB runs on TrueNAS at 192.168.1.100:5432.
Object storage: MinIO in dev (192.168.1.100:9000), DigitalOcean Spaces in prod.

## Code Conventions
- TypeScript strict mode on all files
- async/await everywhere — no raw Promises or .then() chains
- Prisma always injected via PrismaService — never imported directly
- Validate all DTOs with class-validator before any service logic runs
- Every NestJS controller method needs @ApiOperation() and @ApiResponse() for Swagger

## Architecture Rules
- Business logic in service files only — never in controllers
- Controllers only: parse request → call service → return response
- No raw SQL except TimescaleDB time-series queries (use $executeRaw with parameterisation)
- Errors: throw NestJS HttpException types; never throw generic Error

## Response Format
- All API responses wrapped in TransformInterceptor envelope: { success, data, meta }
- Paginated responses include meta: { page, limit, total, totalPages }
- Never expose passwordHash in any response DTO

## Documentation Rules
- New shared content → docs/_shared/_partials/ as MDX component
- Platform-specific content → docs/platforms/t7-{variant}/
- Body-specific content → docs/applications/{body-type}/
- Assembly files → docs/manuals/{platform}-{body}/
- Always preserve MDX component tags when translating
```

### Weekly Development Rhythm

```
Monday:    Plan sprint tasks. Write ADR for major decisions.
           Claude Code: "Review the sprint plan and flag any technical risks."
Tuesday:   Backend-focused (NestJS, Prisma, API).
           Claude Code: "Implement the Warranty module using the schema in prisma/schema.prisma."
Wednesday: Frontend-focused (Next.js pages, components).
           Claude Code: "Build the warranty claim form and wire it to POST /v1/warranties/:id/claims."
Thursday:  Integration, test-writing, bug fixes.
           Claude Code: "Write Jest integration tests for the warranty service covering all edge cases."
Friday:    Docs update + deploy to dev server.
           Claude Code: "Translate docs/_shared/_partials/charging-guide.mdx to Chinese and Indonesian."
```

---

## 13. Estimated Infrastructure Cost

### Phase 1 Monthly Costs (Months 1–6)

| Service | Provider | Spec | Monthly Cost |
|---|---|---|---|
| App Server (Droplet) | DigitalOcean | 4GB RAM / 2 vCPU / 80GB SSD | $24 |
| Managed PostgreSQL | DigitalOcean | 1GB RAM (Basic) | $15 |
| Object Storage | DigitalOcean Spaces | 250GB + 1TB transfer | $5 |
| Cloudflare | Cloudflare | Free tier (CDN, DDoS, DNS) | $0 |
| Domain Name | Namecheap | .com domain | ~$1/mo (paid annually) |
| Email (Resend) | Resend | 3,000 emails/month free | $0 |
| Monitoring (Grafana) | Self-hosted on App Server | Included in Droplet | $0 |
| Secret Management | Doppler | Free tier (1 project) | $0 |
| CI/CD | GitHub Actions | Free for public/small private | $0 |
| Search (Meilisearch) | Self-hosted on App Server | Included in Droplet | $0 |
| **TOTAL** | | | **~$45/month** |

### Phase 1 Scaled Costs (Months 6–12, Growing Traffic)

| Service | Provider | Spec | Monthly Cost |
|---|---|---|---|
| App Server | DigitalOcean | 8GB RAM / 4 vCPU | $48 |
| Managed PostgreSQL | DigitalOcean | 4GB RAM (Standard) | $50 |
| Object Storage | DigitalOcean Spaces | 250GB + 2TB transfer | $10 |
| Cloudflare | Cloudflare | Pro tier (better WAF) | $20 |
| Email (Resend) | Resend | Up to 50,000/month | $20 |
| Backups | DigitalOcean | Automated weekly snapshots | $5 |
| **TOTAL** | | | **~$153/month** |

### Phase 2 FleetOS Costs (Month 12+, 500+ Vehicles)

| Service | Provider | Spec | Monthly Cost |
|---|---|---|---|
| App Server | DigitalOcean | 16GB RAM / 8 vCPU | $96 |
| Managed PostgreSQL | DigitalOcean | 8GB RAM + 1 Read Replica | $150 |
| MQTT Broker (EMQX) | Self-hosted on separate Droplet | 2GB RAM | $18 |
| Object Storage | DigitalOcean Spaces | 1TB + 5TB transfer | $25 |
| Cloudflare | Cloudflare | Business tier | $200 |
| Email | Resend | 100,000+/month | $90 |
| Monitoring (Grafana Cloud) | Grafana | Pro tier | $29 |
| **TOTAL** | | | **~$608/month** |

### Revenue Offset (Phase 2 SaaS Projections)

At 10 fleet customers on the Growth tier ($499/mo each), monthly revenue = $4,990. Infrastructure cost at this scale ≈ $608. Infrastructure as % of revenue = **12%** — well within healthy SaaS infrastructure cost norms (typically 10–20%).

### Cost Optimization Notes

- **DigitalOcean Reserved Instances:** Committing to 1-year reserved pricing saves 20% on Droplet costs.
- **Cloudflare R2 for Storage:** If storage becomes a major cost, Cloudflare R2 offers S3-compatible storage with **zero egress fees** — significant savings at scale.
- **Vertical scaling first:** Always scale up the existing server before adding new servers. It's simpler and cheaper at this scale.
- **Avoid premature optimization:** Many SaaS platforms successfully reach $1M ARR on $500/month infrastructure. Don't architect for 1 million vehicles when you have 10.

---

## Appendix A: Monorepo Structure

```
/ (root)
├── CLAUDE.md                  ← Claude Code reads this automatically every session
├── apps/
│   ├── web/                   ← Next.js (Marketing + Aftersales Portal)
│   ├── api/                   ← NestJS Backend
│   └── docs/                  ← Docusaurus Documentation
│       ├── docs/
│       │   ├── _shared/       ← Layer 1: Content shared across all T7 trucks
│       │   │   ├── _partials/ ← Reusable MDX partials (charging, safety, cab, etc.)
│       │   │   └── components/← Shared system components (drivetrain, brakes, etc.)
│       │   ├── platforms/     ← Layer 2: Per battery/platform variant
│       │   │   ├── t7-90/
│       │   │   ├── t7-90l/
│       │   │   ├── t7-134l/
│       │   │   └── t7-169l/
│       │   ├── applications/  ← Layer 3: Per body configuration
│       │   │   ├── arm-roll/
│       │   │   ├── dumper/
│       │   │   ├── compactor/
│       │   │   └── sweeper/
│       │   ├── manuals/       ← Assembled product manuals (thin import files)
│       │   └── shared-reference/ ← Fault codes, parts numbering, warranty process
│       └── i18n/
│           ├── zh/            ← Chinese Simplified translations
│           └── id/            ← Indonesian translations
├── packages/
│   ├── ui/                    ← Shared shadcn/ui component library
│   ├── types/                 ← Shared TypeScript types (DTOs, enums)
│   ├── utils/                 ← Shared utility functions
│   └── config/                ← Shared ESLint, TypeScript, Tailwind configs
├── prisma/
│   ├── schema.prisma
│   ├── migrations/
│   └── seed.ts
├── docker/
│   ├── docker-compose.dev.yml     ← Workstation services (hot reload)
│   ├── docker-compose.prod.yml    ← Production services (DigitalOcean)
│   └── nginx/
├── .github/
│   └── workflows/
│       └── deploy.yml
├── turbo.json
└── package.json
```

---

## Appendix B: Key Third-Party Integrations (Future)

| Integration | Purpose | Priority |
|---|---|---|
| **Stripe** | SaaS subscription billing | Phase 2 |
| **Twilio / WhatsApp Business API** | WhatsApp notifications for service updates (critical for SE Asia) | Phase 2 |
| **Google Maps Platform** | Fleet vehicle location on map | Phase 2 |
| **Payload CMS** | Headless CMS for marketing content | Month 4 |
| **Hubspot / Pipedrive** | CRM for lead management from Marketing Hub | Month 3 |
| **Sentry** | Error tracking and alerting | Sprint 8 |
| **Plausible Analytics** | Privacy-compliant web analytics | Sprint 8 |

---

*Document prepared as a living architecture document. Update after each major sprint or architectural decision.*
