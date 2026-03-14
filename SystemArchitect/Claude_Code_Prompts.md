# Claude Code Prompt Playbook
## EV Truck Aftersales & Fleet Management Platform

**How to use this guide:**
Each prompt below is a complete instruction you paste directly into Claude Code in your terminal (`claude` command). Run them in order — each prompt assumes the previous one completed successfully. Before pasting, read the **Before running** note and check the **Verify** checklist afterward.

> **Golden rule:** One prompt = one logical unit of work. Never combine two sprints into one prompt. If Claude Code gets confused mid-task, type `stop`, fix the issue, then re-run from the last verified checkpoint.

---

## Table of Contents

- [Part 0 — Project Foundation](#part-0--project-foundation)
- [Phase 1 — Documentation Platform](#phase-1--documentation-platform-weeks-18)
  - [Sprint 0 — Local Dev Environment](#sprint-0--local-dev-environment)
  - [Sprint 1 — Documentation Architecture](#sprint-1--documentation-architecture)
  - [Sprint 2 — Next.js + NestJS Shell](#sprint-2--nextjs--nestjs-shell)
  - [Sprint 3 — SEO, Monitoring & Polish](#sprint-3--seo-monitoring--polish)
- [Phase 2 — Aftersales Portal](#phase-2--aftersales-portal-weeks-924)
  - [Sprint 4 — Authentication & Users](#sprint-4--authentication--users)
  - [Sprint 5 — Vehicle Registry](#sprint-5--vehicle-registry)
  - [Sprint 6 — Warranty System](#sprint-6--warranty-system)
  - [Sprint 7 — Parts Catalog](#sprint-7--parts-catalog)
  - [Sprint 8 — Service Tickets](#sprint-8--service-tickets)
  - [Sprint 9 — Admin Dashboard](#sprint-9--admin-dashboard)
  - [Sprint 10 — Polish & Hardening](#sprint-10--polish--hardening)
- [Phase 3 — FleetOS Foundation](#phase-3--fleetos-foundation-weeks-2536)
  - [Sprint 11 — Telemetry Infrastructure](#sprint-11--telemetry-infrastructure)
  - [Sprint 12 — Fleet Dashboard](#sprint-12--fleet-dashboard)
  - [Sprint 13 — Fault Monitoring & Alerts](#sprint-13--fault-monitoring--alerts)
  - [Sprint 14 — Predictive Maintenance](#sprint-14--predictive-maintenance)
  - [Sprint 15 — Multi-Tenancy & Billing](#sprint-15--multi-tenancy--billing)
  - [Sprint 16 — FleetOS Launch](#sprint-16--fleetos-launch)

---

## Part 0 — Project Foundation

These two prompts run before any sprint. They create the repo skeleton and the `CLAUDE.md` file that every future prompt depends on.

---

### P0-1 · Create the Monorepo Skeleton

> **Before running:** Create a new empty GitHub repo. Clone it locally. `cd` into the repo root. Make sure Node 20+, Docker, and `git` are installed on your workstation.

```
Create a Turborepo monorepo for an EV truck aftersales and fleet management platform.

Repo structure:
  apps/web      → Next.js 14 App Router, TypeScript, TailwindCSS, shadcn/ui
  apps/api      → NestJS 10, TypeScript, Prisma ORM, PostgreSQL
  apps/docs     → Docusaurus 3, TypeScript, i18n for en/zh/id
  packages/ui   → Shared shadcn/ui components
  packages/types → Shared TypeScript interfaces and enums (DTOs, roles, enums)
  packages/config → Shared ESLint, TypeScript, and Tailwind config files

Root files to create:
  turbo.json          → pipeline: build, dev, test, lint tasks
  package.json        → workspaces config for all apps and packages
  .gitignore          → Node, Docker, .env files, .DS_Store
  .env.example        → Template with all required env var keys (values empty)

For apps/web:
  - Next.js 14 with App Router
  - TypeScript strict mode
  - TailwindCSS + shadcn/ui (initialise with default theme)
  - Route groups: app/(marketing) and app/(app) inside src/app/
  - A basic layout.tsx with Inter font for each route group
  - A placeholder page.tsx in each route group that renders "coming soon"

For apps/api:
  - NestJS with @nestjs/cli scaffold
  - Prisma with PostgreSQL provider
  - A basic AppModule with a HealthController at GET /health
  - @nestjs/swagger configured at /api/docs
  - class-validator and class-transformer installed

For apps/docs:
  - Docusaurus 3 with TypeScript
  - i18n config: defaultLocale 'en', locales ['en', 'zh', 'id']
  - localeDropdown navbar item at position right
  - Custom navbar with brand name "T7 Electric Trucks"
  - Placeholder homepage at src/pages/index.tsx

Install all dependencies and confirm each app builds without errors by running
the build command for each. Show me the final directory tree.
```

**Verify:**
- `cd apps/web && npm run build` completes without errors
- `cd apps/api && npm run build` completes without errors
- `cd apps/docs && npm run build` completes without errors
- `.env.example` exists at root with placeholder keys

---

### P0-2 · Write CLAUDE.md

> **Before running:** Run this immediately after P0-1 in the same repo root.

```
Create a CLAUDE.md file at the repository root. Claude Code reads this file
automatically at the start of every session. It must contain the following
sections exactly:

---

# CLAUDE.md — EV Truck Platform

## Project Overview
This is a Turborepo monorepo for a Digital Platform for Electric Truck
Aftersales and Fleet Management. It will grow from a documentation portal
into a full FleetOS SaaS.

Apps:
- apps/web   → Next.js 14 App Router (port 3000) — Marketing site + Aftersales Portal
- apps/api   → NestJS API (port 4000) — All business logic and REST endpoints
- apps/docs  → Docusaurus 3 (port 3001) — Multilingual product documentation

Shared packages:
- packages/types  → Shared TypeScript interfaces, DTOs, and enums
- packages/ui     → Shared shadcn/ui components
- packages/config → Shared ESLint, TypeScript, Tailwind configs

## Development Environment
Local dev: Workstation running docker-compose.dev.yml (hot reload services)
Persistent services on TrueNAS Scale home lab:
  - PostgreSQL 16    → 192.168.1.100:5432  db: evplatform  user: postgres
  - MinIO (S3)       → 192.168.1.100:9000  bucket: evplatform-dev
  - Meilisearch      → 192.168.1.100:7700

Production: DigitalOcean Droplet + Managed PostgreSQL + DO Spaces

## Stack & Key Libraries
Frontend: Next.js 14, TypeScript, TailwindCSS, shadcn/ui, Zustand,
          TanStack Query v5, React Hook Form, Zod
Backend:  NestJS 10, Prisma ORM, class-validator, Passport.js, BullMQ,
          Resend (email), @aws-sdk/client-s3 (MinIO/Spaces)
Docs:     Docusaurus 3, MDX, i18n (en/zh/id)

## NestJS Conventions
- Every entity gets its own module (e.g. VehiclesModule, WarrantyModule)
- Business logic lives ONLY in service files — never in controllers
- Controllers only: validate input DTO → call service → return response
- All request bodies validated with class-validator DTOs in a global ValidationPipe
- All responses wrapped in a TransformInterceptor: { success, data, meta }
- Every endpoint decorated with @ApiOperation() and @ApiResponse() for Swagger
- Auth: JwtAuthGuard + RolesGuard on every protected route
- Never expose passwordHash in any response DTO

## Frontend Conventions
- TanStack Query for all server state (no useEffect for data fetching)
- Zustand for client state (auth store, UI preferences)
- React Hook Form + Zod for all forms
- Zod schemas in packages/types — shared between frontend and backend
- Route group /(marketing) for public pages, /(app) for authenticated portal

## API Conventions
- Base URL: /v1/
- Paginated responses include meta: { page, limit, total, totalPages }
- Error responses: { success: false, error: { code, message, statusCode } }
- Rate limiting: 100 req/15min general, 10 req/15min on auth endpoints

## Documentation Architecture (Three-Layer Model)
docs/_shared/_partials/   → Content shared across ALL T7 truck models
docs/platforms/t7-{90,90l,134l,169l}/  → Battery-variant specific content
docs/applications/{arm-roll,dumper,compactor,sweeper}/  → Body-type content
docs/manuals/{platform}-{body}/  → Thin MDX assembly files (imports from layers)

Product matrix:
  Platforms: T7-90 (90kWh), T7-90L (90kWh long), T7-134L (134kWh), T7-169L (169kWh)
  Body types: Standard, Arm Roll, Dumper, Compactor, Sweeper

Rule: When creating documentation, ask "does this apply to all T7 trucks?"
  YES → _shared/_partials/
  Only to one battery variant → platforms/t7-{variant}/
  Only to one body type → applications/{body-type}/
  Unique combination → manuals/{platform}-{body}/

MDX component tags (e.g. <SpecTable />, import statements, frontmatter YAML)
must ALWAYS be preserved exactly when translating to ZH or ID.

## Database
Prisma schema at: prisma/schema.prisma (in repo root, shared by apps/api)
Run migrations: cd apps/api && npx prisma migrate dev
Seed database:  cd apps/api && npx prisma db seed

## Environment Files
.env.local     → Local dev (TrueNAS IPs, local secrets)
.env.production → Production (DigitalOcean endpoints, prod secrets)
Never commit .env files. Use .env.example as template.

---

After creating the file, confirm it exists at the repo root and show its
full contents.
```

**Verify:**
- `cat CLAUDE.md` shows all sections
- File is committed: `git add CLAUDE.md && git commit -m "add CLAUDE.md project context"`

---

## Phase 1 — Documentation Platform (Weeks 1–8)

---

### Sprint 0 — Local Dev Environment

---

#### S0-1 · Docker Compose Dev Stack

> **Before running:** TrueNAS must be running with PostgreSQL, MinIO, and Meilisearch already deployed. Note the LAN IP. Have your MinIO access key and secret ready.

```
Read CLAUDE.md first.

Create docker-compose.dev.yml at the repo root for local development on my
workstation. This file runs only the services that need hot reload. Persistent
services (PostgreSQL, MinIO, Meilisearch) run on TrueNAS at 192.168.1.100.

Services to include:

  web:
    build: apps/web with target 'development'
    command: npm run dev
    ports: 3000:3000
    volumes: apps/web source mounted for hot reload (exclude node_modules)
    environment:
      NEXT_PUBLIC_API_URL: http://localhost:4000/v1

  api:
    build: apps/api with target 'development'
    command: npm run start:dev
    ports: 4000:4000
    volumes: apps/api source mounted for hot reload (exclude node_modules)
    environment:
      DATABASE_URL: postgresql://postgres:password@192.168.1.100:5432/evplatform
      REDIS_URL: redis://redis:6379
      S3_ENDPOINT: http://192.168.1.100:9000
      S3_BUCKET: evplatform-dev
      S3_REGION: us-east-1
      MEILISEARCH_HOST: http://192.168.1.100:7700
      JWT_SECRET: dev-jwt-secret-change-in-prod
      JWT_REFRESH_SECRET: dev-refresh-secret-change-in-prod

  docs:
    build: apps/docs with target 'development'
    command: npm run start -- --host 0.0.0.0 --port 3001
    ports: 3001:3001
    volumes: apps/docs source mounted for hot reload (exclude node_modules)

  redis:
    image: redis:7-alpine
    ports: 6379:6379
    (no volume mount — ephemeral in dev is fine)

Also create a Makefile at repo root with these shortcuts:
  make dev      → docker compose -f docker-compose.dev.yml up
  make dev-down → docker compose -f docker-compose.dev.yml down
  make logs     → docker compose -f docker-compose.dev.yml logs -f
  make shell-api → docker compose exec api sh
  make migrate  → docker compose exec api npx prisma migrate dev
  make seed     → docker compose exec api npx prisma db seed

Create a multi-stage Dockerfile for apps/api with 'development' and 'production'
targets. Development target uses ts-node with watch mode.
Create a multi-stage Dockerfile for apps/web with 'development' and 'production'
targets. Development target uses next dev.
Create a simple Dockerfile for apps/docs with a development target.

Then create .env.local.example showing all required vars with placeholder values
and instructions to copy to .env.local and fill in real values.
```

**Verify:**
- `make dev` starts all 4 containers without errors
- `curl http://localhost:4000/health` returns `{ status: "ok" }`
- `curl http://localhost:3001` returns Docusaurus HTML

---

#### S0-2 · Prisma Schema — Core Models

> **Before running:** TrueNAS PostgreSQL must be reachable. Create `.env.local` from `.env.local.example` with real TrueNAS IP and credentials.

```
Read CLAUDE.md first.

Create the complete Prisma schema at prisma/schema.prisma. The database is
PostgreSQL 16 running on TrueNAS. Include all core models for the EV truck
aftersales platform:

Enums:
  UserRole: CUSTOMER | SERVICE_PARTNER | INTERNAL_ENGINEER | ADMIN
  WarrantyStatus: ACTIVE | EXPIRED | VOIDED
  ClaimStatus: SUBMITTED | UNDER_REVIEW | APPROVED | REJECTED | RESOLVED
  TicketCategory: WARRANTY_CLAIM | TECHNICAL_ISSUE | PARTS_REQUEST | GENERAL_INQUIRY
  TicketStatus: OPEN | IN_PROGRESS | AWAITING_CUSTOMER | RESOLVED | CLOSED
  PartAvailability: IN_STOCK | LOW_STOCK | OUT_OF_STOCK | DISCONTINUED

Models:
  Organization  → id, name, country, phone, email, createdAt, updatedAt
                  relations: users[], vehicles[], fleets[]

  User          → id, email (unique), passwordHash, firstName, lastName,
                  role (UserRole, default CUSTOMER), isActive (default true),
                  emailVerified (default false), language (default 'en'),
                  organizationId (optional FK), createdAt, updatedAt
                  relations: organization, refreshTokens[], vehicles (VehicleOwner[]),
                             tickets (creator), assignedTickets, comments, claims

  RefreshToken  → id, token (unique), userId (FK cascade delete), expiresAt, createdAt

  VehicleModel  → id, name, manufacturer, category, batteryCapacityKwh,
                  rangeKm, payloadKg, grossWeightKg, chargePortType, imageUrl,
                  isActive (default true), createdAt, updatedAt
                  relations: vehicles[], compatibleParts (PartCompatibility[])

  Vehicle       → id, vin (unique), registrationNo, purchaseDate, deliveryDate,
                  mileageKm (default 0), modelId (FK), organizationId (optional FK),
                  fleetId (optional FK), telemetryEnabled (default false),
                  lastSeenAt, createdAt, updatedAt
                  relations: model, organization, fleet, owners (VehicleOwner[]),
                             warranties[], tickets[]

  VehicleOwner  → id, userId, vehicleId, isPrimary (default true), since
                  unique: [userId, vehicleId]

  WarrantyType  → id, name, description, durationMonths, maxMileageKm, coverageDetails

  Warranty      → id, vehicleId, warrantyTypeId, startDate, endDate,
                  status (WarrantyStatus, default ACTIVE), notes, createdAt, updatedAt
                  relations: vehicle, warrantyType, claims[]

  WarrantyClaim → id, warrantyId, submittedById, title, description,
                  mileageAtClaim, status (ClaimStatus, default SUBMITTED),
                  resolution, createdAt, updatedAt
                  relations: warranty, submittedBy, attachments[]

  PartCategory  → id, name, slug (unique), parentId (self-relation for tree)
                  relations: parent, children[], parts[]

  Part          → id, partNumber (unique), name, description, categoryId,
                  unitPrice, currency (default 'USD'), availability (PartAvailability),
                  weight, imageUrl, supersededById (optional self-relation),
                  createdAt, updatedAt
                  relations: category, supersededBy, supersedes[], compatibleModels[]

  PartCompatibility → id, partId, modelId, notes
                      unique: [partId, modelId]

  Ticket        → id, ticketNo (unique), title, description,
                  category (TicketCategory), status (TicketStatus, default OPEN),
                  priority (Int, default 2), vehicleId (optional),
                  createdById, assigneeId (optional), resolvedAt,
                  createdAt, updatedAt
                  relations: vehicle, createdBy, assignee, comments[], attachments[]

  TicketComment → id, ticketId, authorId, body, isInternal (default false),
                  createdAt
                  relations: ticket (cascade delete), author, attachments[]

  Attachment    → id, filename, url, mimeType, sizeBytes,
                  claimId (optional FK), ticketId (optional FK),
                  commentId (optional FK), createdAt

  Fleet         → id, name, organizationId, createdAt, updatedAt
                  relations: organization, vehicles[]

After creating the schema:
1. Configure the Prisma client generator with output to node_modules
2. Run: npx prisma migrate dev --name init
3. Confirm the migration ran successfully and all tables were created
4. Show the migration file content
```

**Verify:**
- `npx prisma studio` opens and shows all tables with correct columns
- `npx prisma validate` passes with no errors

---

#### S0-3 · GitHub Actions CI/CD Pipeline

```
Read CLAUDE.md first.

Create .github/workflows/deploy.yml for a CI/CD pipeline that:

1. Triggers on push to the 'main' branch

2. Run tests job (runs on ubuntu-latest):
   - Checkout code
   - Setup Node 20
   - Install dependencies: npm ci
   - Run linting: npm run lint (use turbo)
   - Run unit tests: npm run test (use turbo)
   - Run build check: npm run build (use turbo)

3. Deploy job (runs after tests pass, on ubuntu-latest):
   - Uses GitHub secret SERVER_HOST for the DigitalOcean Droplet IP
   - Uses GitHub secret SSH_PRIVATE_KEY for SSH access
   - Uses appleboy/ssh-action to SSH into the server and run:
       cd /opt/evplatform
       git pull origin main
       docker compose -f docker-compose.prod.yml up -d --build
       docker system prune -f
       npx prisma migrate deploy (inside api container)

Also create docker-compose.prod.yml for production (DigitalOcean):

Services:
  nginx:
    image: nginx:alpine
    ports: 80:80 and 443:443
    volumes: ./nginx/nginx.conf, certbot certs volume
    depends on: web, api

  web:
    build: apps/web, target: production
    environment: NEXT_PUBLIC_API_URL from env

  api:
    build: apps/api, target: production
    environment: DATABASE_URL, REDIS_URL, JWT_SECRET, JWT_REFRESH_SECRET,
                 RESEND_API_KEY, S3_ENDPOINT, S3_KEY, S3_SECRET, S3_BUCKET,
                 MEILISEARCH_HOST, MEILISEARCH_KEY
    depends on: redis

  docs:
    build: apps/docs, target: production

  redis:
    image: redis:7-alpine
    volumes: redis-data:/data

  meilisearch:
    image: getmeili/meilisearch:latest
    volumes: meili-data:/meili_data
    environment: MEILI_MASTER_KEY from env

Create nginx/nginx.conf with:
  - Server blocks for: yourdomain.com, docs.yourdomain.com, api.yourdomain.com
  - Proxy rules to correct Docker services
  - Rate limiting zone: api_limit 100 req/15min; auth_limit 10 req/15min
  - Gzip compression enabled
  - SSL termination (certificate paths from Certbot)
  - Security headers: X-Frame-Options, X-Content-Type-Options, HSTS
```

**Verify:**
- YAML syntax is valid: `yamllint .github/workflows/deploy.yml`
- `docker compose -f docker-compose.prod.yml config` validates without errors

---

### Sprint 1 — Documentation Architecture

---

#### S1-1 · Three-Layer Documentation Structure

> **Before running:** `make dev` should be running. Docusaurus dev server accessible at `http://localhost:3001`.

```
Read CLAUDE.md first. Focus on the Documentation Architecture section.

Scaffold the three-layer documentation folder structure inside apps/docs/docs/.
The T7 electric truck line has 4 platform variants and 5 body types.

Create this complete folder structure with placeholder .mdx files in each folder:

apps/docs/docs/
  _shared/
    _partials/
      hv-safety.mdx
      charging-guide.mdx
      daily-inspection.mdx
      cab-controls.mdx
      warranty-overview.mdx
    components/
      drivetrain.mdx
      braking-system.mdx
      cooling-system.mdx

  platforms/
    t7-90/
      specifications.mdx
      battery-system.mdx
      range-charging.mdx
    t7-90l/
      specifications.mdx
      battery-system.mdx
      range-charging.mdx
    t7-134l/
      specifications.mdx
      battery-system.mdx
      range-charging.mdx
    t7-169l/
      specifications.mdx
      battery-system.mdx
      range-charging.mdx

  applications/
    standard/
      overview.mdx
    arm-roll/
      overview.mdx
      operation.mdx
      maintenance.mdx
      safety.mdx
    dumper/
      overview.mdx
      operation.mdx
      maintenance.mdx
      safety.mdx
    compactor/
      overview.mdx
      operation.mdx
      maintenance.mdx
    sweeper/
      overview.mdx
      operation.mdx
      maintenance.mdx

  manuals/
    t7-134l-standard/
      user-manual.mdx
    t7-134l-dumper/
      user-manual.mdx
      service-manual.mdx

  shared-reference/
    fault-codes.mdx
    parts-numbering.mdx
    warranty-claim-process.mdx

Each placeholder .mdx file should have valid frontmatter with title and
sidebar_label, and a single paragraph: "Content coming soon."

Then create two MDX component files at apps/docs/src/components/:

  SpecTable.tsx — TypeScript React component accepting props:
    batteryKwh: number
    rangeKm: number
    gvwKg: number
    lengthMm: number
    widthMm: number
    wheelbase?: string
  Renders a clean HTML table showing these specs with proper labels and units.

  RangeTable.tsx — TypeScript React component accepting props:
    batteryKwh: number
    rangeCity: number
    rangeHighway: number
    rangeFullLoad?: number
  Renders a range comparison table.

Then update apps/docs/sidebars.ts to show the full product hierarchy:
  T7-90 (90 kWh Standard) → user-manual
  T7-90L (90 kWh Long) → user-manual
  T7-134L (134 kWh) → Standard user-manual, Dumper user-manual + service-manual
  T7-169L (169 kWh) → Standard user-manual
  Reference → fault-codes, parts-numbering, warranty-claim-process
```

**Verify:**
- Docusaurus builds without errors: `cd apps/docs && npm run build`
- Sidebar shows all product categories in browser

---

#### S1-2 · Write Shared Partials (English)

```
Read CLAUDE.md first. Read the existing files in apps/docs/docs/_shared/_partials/
to understand the current placeholder content.

Replace the placeholder content in each shared partial with real, professional
technical documentation for the T7 series electric trucks. Write in clear,
operator-friendly English. Each file should be production-quality content.

hv-safety.mdx — High Voltage Safety
  Cover: HV system voltage (up to 750V DC), mandatory PPE (insulated gloves
  Category 0 rated, safety glasses, insulated tools), isolation procedure
  before any service work, HV interlock system description, emergency
  disconnect location, what to do if HV warning light is on while driving,
  who is authorised to work on HV systems (certified technicians only).
  Include a prominent warning callout box at the top.

charging-guide.mdx — Charging Your T7 Electric Truck
  Cover: supported charge standards (CCS2 DC fast charge, AC Type 2),
  connecting and disconnecting the charge cable safely, charge port location
  on the vehicle, understanding the dashboard charge indicators (SOC%, estimated
  range, time to full charge), recommended charging practice (charge to 80% for
  daily use, 100% before long routes), cold weather charging notes, what to do
  if charging does not start, charge error codes and first steps.

daily-inspection.mdx — Pre-Trip Daily Inspection Checklist
  Format as a numbered checklist. Cover: HV battery indicator check,
  tyre pressure and condition, brake test (service and parking),
  lights and signals, mirrors and cameras, fluid levels (coolant for
  thermal management), charge cable stored correctly, no warning lights
  on dashboard, body and cargo area secure. Include a note that if any
  item fails the checklist, the vehicle must not be operated.

cab-controls.mdx — Cab Controls and Dashboard Guide
  Cover: instrument cluster layout (speedometer, SOC gauge, range estimate,
  odometer, trip meter), regenerative braking selector (eco/normal/strong regen),
  drive mode selector (ECO/NORMAL/SPORT), parking brake (electronic), HVAC
  controls, exterior lights, hazard lights, horn, reverse camera activation.
  Use a description list format for each control.

warranty-overview.mdx — Warranty Coverage Summary
  Cover: what the standard warranty covers (drivetrain 5 years/500,000km,
  battery 8 years/800,000km, general vehicle 2 years/100,000km),
  what is not covered (accident damage, improper maintenance, unauthorised
  modifications, use of non-approved parts), how to check your warranty
  status (log in to the customer portal), how to start a warranty claim
  (portal or contact service partner), service partner obligation to honour
  warranty work.

Each file must use proper Docusaurus MDX: frontmatter with title and
description, at least one Docusaurus admonition (:::warning, :::tip, or
:::info), and be formatted for good readability.
```

**Verify:**
- All 5 files have real content (not placeholders)
- `cd apps/docs && npm run build` completes without MDX parse errors

---

#### S1-3 · Assemble T7-134L Product Manuals + Language Switcher Config

```
Read CLAUDE.md first. Read the existing files in:
  apps/docs/docs/_shared/_partials/ (shared content written in S1-2)
  apps/docs/docs/platforms/t7-134l/ (placeholder specs)
  apps/docs/docs/applications/dumper/ (placeholder operation content)

Do three things:

TASK 1 — Write T7-134L platform specs
Replace placeholder content in apps/docs/docs/platforms/t7-134l/battery-system.mdx
with real content about the 134kWh battery system:
  - LFP (Lithium Iron Phosphate) cell chemistry, why it's chosen for trucks
    (cycle life >3000 cycles, thermal stability, no thermal runaway risk)
  - Battery pack architecture: modules in series/parallel configuration
  - BMS (Battery Management System): cell balancing, temperature monitoring,
    state-of-charge calculation, state-of-health estimation
  - Operating temperature range: charging 0°C to 45°C, discharging -20°C to 55°C
  - Thermal management: liquid-cooled/heated battery pack
  - Battery health indicators in the dashboard
  - What to do if battery temperature warning appears

Replace placeholder in apps/docs/docs/platforms/t7-134l/specifications.mdx:
  Use the SpecTable component with these values:
    batteryKwh=134, rangeKm=280, gvwKg=18000, lengthMm=8450, widthMm=2500
  Also add: max payload 8,000kg, motor power 250kW, peak torque 2,400Nm,
  max speed 100km/h, charge time (AC 22kW: ~7h, DC 120kW: ~70min to 80%)

TASK 2 — Write the T7-134L Dumper user manual assembly file
Replace the placeholder at apps/docs/docs/manuals/t7-134l-dumper/user-manual.mdx
with a proper assembly file that:
  - Imports and renders: HVSafety, ChargingGuide, CabControls from _shared/_partials
  - Renders SpecTable with T7-134L values
  - Imports and renders BatterySystem from platforms/t7-134l/battery-system.mdx
  - Imports DumperOperation and DumperSafety from applications/dumper/
  - Has a logical chapter flow: Safety → Specifications → Battery → Controls →
    Charging → Dumper Body Operation → Dumper Safety
  - Has correct frontmatter: title, description, sidebar_label

TASK 3 — Verify the language switcher
Check that docusaurus.config.ts has:
  i18n.defaultLocale = 'en'
  i18n.locales = ['en', 'zh', 'id']
  navbar item: { type: 'localeDropdown', position: 'right' }

If any of these are missing or incorrect, fix them.
Then run: npm run docusaurus write-translations -- --locale zh
and:      npm run docusaurus write-translations -- --locale id
to generate the translation key files in i18n/zh/ and i18n/id/.
```

**Verify:**
- `http://localhost:3001/docs/manuals/t7-134l-dumper/user-manual` renders full content
- Language switcher dropdown appears in the navbar
- `i18n/zh/` and `i18n/id/` directories exist with translation JSON files

---

#### S1-4 · Translate Shared Partials to Chinese and Indonesian

```
Read CLAUDE.md first. Focus on the translation rules in the Documentation
Architecture section — MDX component tags must be preserved exactly.

Translate the following shared partial files to Chinese Simplified (zh)
and Bahasa Indonesia (id):

Source files (English originals):
  apps/docs/docs/_shared/_partials/hv-safety.mdx
  apps/docs/docs/_shared/_partials/charging-guide.mdx
  apps/docs/docs/_shared/_partials/daily-inspection.mdx
  apps/docs/docs/_shared/_partials/cab-controls.mdx
  apps/docs/docs/_shared/_partials/warranty-overview.mdx

Output paths:
  Chinese:    i18n/zh/docusaurus-plugin-content-docs/current/_shared/_partials/{filename}
  Indonesian: i18n/id/docusaurus-plugin-content-docs/current/_shared/_partials/{filename}

Critical translation rules:
  1. Preserve ALL MDX frontmatter keys (only translate the values: title, description)
  2. Preserve ALL Docusaurus admonition markers (:::warning, :::tip, :::info, :::)
  3. Preserve ALL import statements and component tags exactly (<SpecTable />, etc.)
  4. Preserve ALL markdown formatting (##, **, -, numbered lists)
  5. Use professional technical vocabulary appropriate for truck operators
  6. For Chinese: use Simplified Chinese (mainland standard), not Traditional
  7. For Indonesian: use formal Bahasa Indonesia (baku), not colloquial

Translate the admonition labels too:
  :::warning → :::warning (keep tag), translate text inside
  :::tip     → :::tip (keep tag), translate text inside

After creating all translation files, run:
  cd apps/docs && npm run build
to confirm all three locales build without errors.
```

**Verify:**
- `http://localhost:3001/zh/docs/manuals/t7-134l-dumper/user-manual` renders in Chinese
- `http://localhost:3001/id/docs/manuals/t7-134l-dumper/user-manual` renders in Indonesian
- Language switcher on that page correctly navigates between all three locales

---

### Sprint 2 — Next.js + NestJS Shell

---

#### S2-1 · Next.js Marketing Homepage

```
Read CLAUDE.md first.

Build the marketing homepage for the EV truck platform at
apps/web/src/app/(marketing)/page.tsx.

The marketing site represents a company that manufactures the T7 series
electric trucks (T7-90, T7-90L, T7-134L, T7-169L). Target customers are
fleet operators and logistics companies in Southeast Asia and China.

Create the following sections on the homepage. Use TailwindCSS and shadcn/ui
components. The design should be clean, professional, and industrial — think
dark navy (#1A3557) as primary colour with electric blue (#2563EB) as accent.

Sections:
  1. Navbar
     - Logo: "T7 Electric" text with lightning bolt icon
     - Links: Products, Documentation, Support, Login
     - CTA button: "Request Demo" (outlined)
     - Fully responsive (hamburger menu on mobile)

  2. Hero Section
     - Headline: "The Electric Truck Built for Heavy Work"
     - Subline: "90 to 169 kWh. Zero emissions. Full payload."
     - Two CTAs: "Explore the T7 Series" and "View Documentation"
     - Background: dark gradient (navy to dark blue)

  3. Product Line Strip
     - Four cards in a horizontal row: T7-90, T7-90L, T7-134L, T7-169L
     - Each card shows: model name, battery capacity, range, key spec
     - Cards link to /products/{model}

  4. Key Features Grid (3 columns)
     - Zero Emissions — LFP battery, no tailpipe emissions
     - Lower Operating Cost — 60-70% fuel cost reduction vs diesel
     - Smart Diagnostics — Real-time fault monitoring and alerts
     - Fast Charging — DC fast charge: 80% in 70 minutes
     - Proven Payload — Full GVW ratings maintained
     - Fleet Ready — Telematics and fleet management platform

  5. Body Configuration Section
     - Headline: "One Platform, Many Configurations"
     - Five tiles: Standard, Arm Roll, Dumper, Compactor, Sweeper
     - Brief description of each use case

  6. Lead Capture CTA Banner
     - "Ready to Electrify Your Fleet?"
     - Email input + "Get a Quote" button
     - Form submits to POST /v1/leads via TanStack Query mutation
     - Show success toast on submit

  7. Footer
     - Company name, links: Products, Docs, Aftersales, Privacy Policy
     - Copyright notice

Create a global layout at apps/web/src/app/(marketing)/layout.tsx with the
navbar and footer. Create all components as separate files in
apps/web/src/components/marketing/.

Make the page fully responsive for mobile (375px), tablet (768px), and
desktop (1280px).
```

**Verify:**
- Homepage loads at `http://localhost:3000`
- All sections visible, no layout breaks at 375px mobile width
- Lead capture form submits and shows a toast (even if API not yet ready, mock it)

---

#### S2-2 · NestJS API Foundation

```
Read CLAUDE.md first.

Set up the NestJS API foundation in apps/api/src/ with all the shared
infrastructure that every module will use.

1. Global Configuration
   Create apps/api/src/config/ with:
     app.config.ts      → port, api prefix (/v1), cors origins
     database.config.ts → DATABASE_URL from env
     jwt.config.ts      → JWT_SECRET, JWT_REFRESH_SECRET, expiry times
   Use @nestjs/config with Joi validation so the app crashes at startup if
   required env vars are missing.

2. Prisma Service
   Create apps/api/src/prisma/prisma.module.ts and prisma.service.ts
   PrismaService extends PrismaClient and implements OnModuleInit/OnModuleDestroy.
   Export PrismaModule as a global module so every other module can inject
   PrismaService without importing PrismaModule.

3. Transform Interceptor (response envelope)
   Create apps/api/src/common/interceptors/transform.interceptor.ts
   Wraps every successful response in:
     { success: true, data: <original response>, meta: <if paginated> }
   Apply globally in main.ts.

4. HTTP Exception Filter
   Create apps/api/src/common/filters/http-exception.filter.ts
   Catches all HttpExceptions and formats them as:
     { success: false, error: { code: <enum string>, message: <string>, statusCode: <number> } }
   Apply globally in main.ts.

5. Validation Pipe
   Apply GlobalValidationPipe in main.ts with:
     whitelist: true (strip unknown properties)
     forbidNonWhitelisted: true
     transform: true (auto-transform to DTO types)

6. Rate Limiting
   Install @nestjs/throttler. Configure globally:
     General: ttl 15min, limit 100
     Apply a stricter rate limit (ttl 15min, limit 10) to auth routes later.

7. Swagger Setup
   Configure @nestjs/swagger in main.ts:
     title: "T7 Electric Truck Platform API"
     description: "Aftersales and Fleet Management REST API"
     version: "1.0"
     bearerAuth scheme
     Available at /api/docs

8. CORS
   Enable CORS in main.ts allowing origins from env var CORS_ORIGINS
   (comma-separated list). Default to localhost:3000 in development.

9. Health Check
   Create a HealthModule with GET /v1/health endpoint that returns:
     { status: "ok", timestamp: <ISO string>, version: "1.0.0" }
   No auth required.

After all setup, confirm the API starts clean:
  npm run start:dev
and that http://localhost:4000/v1/health returns correctly.
Show the Swagger UI is accessible at http://localhost:4000/api/docs.
```

**Verify:**
- `curl http://localhost:4000/v1/health` returns `{ success: true, data: { status: "ok" } }`
- `http://localhost:4000/api/docs` shows Swagger UI with bearer auth option
- Unknown route returns formatted error envelope (not raw NestJS error)

---

### Sprint 3 — SEO, Monitoring & Polish

---

#### S3-1 · SEO, Sitemap, and Analytics

```
Read CLAUDE.md first.

Add SEO infrastructure to apps/web (Next.js marketing site):

1. Metadata
   Update apps/web/src/app/(marketing)/layout.tsx to export a metadata object:
     title: "T7 Electric Trucks — Commercial EV Fleet Solutions"
     description: "T7 series electric trucks from 90kWh to 169kWh. Built for
       heavy commercial work in Southeast Asia. Arm roll, dumper, compactor,
       sweeper configurations."
     openGraph: title, description, type 'website', image placeholder
     keywords relevant to commercial EV trucks

   Each page should override title using: { title: { template: '%s | T7 Electric' } }

2. Sitemap
   Create apps/web/src/app/sitemap.ts using Next.js App Router sitemap generation.
   Include: homepage, /products/t7-90, /products/t7-90l, /products/t7-134l,
   /products/t7-169l, /contact, /support

3. Robots.txt
   Create apps/web/src/app/robots.ts allowing all crawlers on marketing pages,
   disallowing /app/* (authenticated portal) and /api/*.

4. JSON-LD Structured Data
   Add Organization and Product schema markup to the homepage using a
   JsonLd component. Organisation: T7 Electric Trucks, url: yourdomain.com.
   Product entries for each truck model with name, description, brand.

5. Plausible Analytics
   Add a Plausible script tag via Next.js Script component in the root layout.
   The domain should come from env var NEXT_PUBLIC_PLAUSIBLE_DOMAIN.
   Only load in production (NODE_ENV === 'production').

6. Docusaurus SEO
   Update apps/docs/docusaurus.config.ts:
     Add url and baseUrl fields
     Add meta description tag
     Enable sitemap plugin (@docusaurus/plugin-sitemap)
     Add Google site verification meta tag (from env var)

Show what the sitemap output looks like by running npm run build and
checking the generated sitemap.xml.
```

**Verify:**
- `http://localhost:3000` page source contains `<title>` and Open Graph tags
- `http://localhost:3000/sitemap.xml` returns valid XML
- `http://localhost:3000/robots.txt` disallows `/app/*`

---

## Phase 2 — Aftersales Portal (Weeks 9–24)

---

### Sprint 4 — Authentication & Users

---

#### S4-1 · Auth Module — Backend

```
Read CLAUDE.md first.

Build the complete authentication system in apps/api/src/modules/auth/.

Install required packages: @nestjs/passport, passport, passport-local,
passport-jwt, @nestjs/jwt, bcrypt, @types/bcrypt, @types/passport-jwt.

Create the following files:

auth.module.ts — imports UsersModule, JwtModule (async with config),
  PassportModule. Registers LocalStrategy and JwtStrategy.

auth.service.ts — methods:
  register(dto): hash password with bcrypt (cost 12), create User,
    enqueue email verification job, return user without passwordHash
  login(user): generate access token (15min) and refresh token (7 days),
    store refresh token hash in RefreshToken table, return both tokens
  refresh(token): find RefreshToken by token, validate not expired,
    invalidate old token, generate new token pair (rotation),
    detect reuse attack (if token already used, invalidate entire family)
  logout(userId, token): delete RefreshToken record
  verifyEmail(token): find user by verification token, set emailVerified=true
  forgotPassword(email): generate reset token, store hashed in User,
    enqueue password reset email job
  resetPassword(token, newPassword): validate token, hash new password, update

auth.controller.ts — endpoints (all prefixed /v1/auth):
  POST /register         → calls auth.service.register
  POST /login            → LocalAuthGuard, returns { accessToken } + sets
                           refresh token as httpOnly cookie (7 days)
  POST /refresh          → reads refresh token from httpOnly cookie,
                           returns new { accessToken }, sets new cookie
  POST /logout           → clears cookie, invalidates refresh token
  GET  /me               → JwtAuthGuard, returns current user profile
  POST /forgot-password  → public endpoint
  POST /reset-password   → public endpoint
  POST /verify-email     → public endpoint

DTOs (in auth/dto/):
  RegisterDto: email (IsEmail), password (MinLength 8), firstName, lastName
  LoginDto: email, password
  ForgotPasswordDto: email
  ResetPasswordDto: token, newPassword (MinLength 8)

Strategies (in auth/strategies/):
  LocalStrategy: validates email+password, returns user object
  JwtStrategy: extracts Bearer token, validates, returns { userId, email, role }

Guards (in common/guards/):
  JwtAuthGuard: extends AuthGuard('jwt')
  RolesGuard: reads @Roles() decorator, checks user.role against allowed roles
  LocalAuthGuard: extends AuthGuard('local')

Decorators (in common/decorators/):
  @Roles(...roles): SetMetadata decorator
  @CurrentUser(): custom param decorator returning req.user

Apply strict rate limiting to all auth endpoints: 10 requests per 15 minutes.
Add all Swagger decorators (@ApiOperation, @ApiResponse, @ApiBearerAuth).
Write the users.service.ts with findById, findByEmail, updateProfile methods.
```

**Verify:**
- `POST /v1/auth/register` creates a user and returns profile (no passwordHash)
- `POST /v1/auth/login` returns `accessToken` in body and sets httpOnly cookie
- `GET /v1/auth/me` with bearer token returns current user
- `GET /v1/auth/me` without token returns 401 with error envelope

---

#### S4-2 · Auth Module — Frontend

```
Read CLAUDE.md first.

Build the authentication UI in apps/web/src/app/(app)/.

1. Auth Store (Zustand)
   Create apps/web/src/store/auth.store.ts with:
     state: user (User | null), accessToken (string | null), isLoading
     actions: setUser, setAccessToken, logout, initFromCookie
   Access token stored in memory only (not localStorage). On page refresh,
   attempt silent refresh via POST /v1/auth/refresh before showing login page.

2. API Client
   Create apps/web/src/lib/api-client.ts using axios or fetch:
     baseURL: NEXT_PUBLIC_API_URL + '/v1'
     Request interceptor: attach Authorization: Bearer {accessToken} header
     Response interceptor: on 401, try POST /refresh, retry original request.
       If refresh fails, clear store and redirect to /login.
     Wrapper for all API calls that extracts .data from response envelope.

3. Login Page at /app/login
   Form fields: email, password. React Hook Form + Zod validation.
   On submit: POST /v1/auth/login → store accessToken → redirect to /app/dashboard
   Show inline field errors. Show toast on login failure.
   Link to "Forgot Password".

4. Register Page at /app/register
   Fields: firstName, lastName, email, password, confirmPassword.
   Zod schema validates passwords match.
   On success: show "Check your email to verify your account" message.

5. Forgot Password Page at /app/forgot-password
   Single email field. On submit shows "If this email exists, a reset link
   has been sent." (don't reveal if email exists or not).

6. Reset Password Page at /app/reset-password?token=...
   Reads token from URL params. Fields: newPassword, confirmPassword.
   On success: redirect to /login with "Password updated" toast.

7. Auth Guard (middleware)
   Create apps/web/src/middleware.ts using Next.js middleware:
   Protected routes (starting with /app/ except /app/login, /app/register):
   redirect to /app/login if no valid session. Public routes pass through.

8. Dashboard Shell at /app/dashboard
   Simple layout with sidebar navigation:
     My Vehicles, Warranties, Parts Catalog, Service Tickets, Profile
   Show user's firstName in the sidebar header.
   Use shadcn/ui Sidebar component.

All forms use TanStack Query mutations for API calls.
```

**Verify:**
- Register a new user → receives verification email (check Resend logs or dev preview)
- Login with credentials → redirected to `/app/dashboard`
- Navigate to `/app/dashboard` while logged out → redirected to `/app/login`
- Refresh page while logged in → session persists via silent refresh

---

### Sprint 5 — Vehicle Registry

---

#### S5-1 · Vehicle Module — Backend + Seed Data

```
Read CLAUDE.md first.

Build the vehicles module in apps/api/src/modules/vehicles/ and seed the
VehicleModel table with real T7 series data.

1. Seed Data (apps/api/prisma/seed.ts)
   Create seed data for VehicleModel:
   - T7-90:   name, batteryCapacityKwh: 90,  rangeKm: 220, grossWeightKg: 14000,
              payloadKg: 5000,  chargePortType: 'CCS2', category: 'Medium Duty'
   - T7-90L:  batteryCapacityKwh: 90,  rangeKm: 210, grossWeightKg: 16000,
              payloadKg: 6000,  chargePortType: 'CCS2', category: 'Medium Duty'
   - T7-134L: batteryCapacityKwh: 134, rangeKm: 280, grossWeightKg: 18000,
              payloadKg: 8000,  chargePortType: 'CCS2', category: 'Heavy Duty'
   - T7-169L: batteryCapacityKwh: 169, rangeKm: 320, grossWeightKg: 25000,
              payloadKg: 12000, chargePortType: 'CCS2', category: 'Heavy Duty'

   Also seed WarrantyType records:
   - Battery Warranty: 96 months (8 years), maxMileageKm: 800000
   - Drivetrain Warranty: 60 months (5 years), maxMileageKm: 500000
   - General Vehicle Warranty: 24 months (2 years), maxMileageKm: 100000

   Create 2 demo vehicles linked to a demo customer user for testing.

2. Vehicles Controller and Service
   GET    /v1/vehicles         → list vehicles owned by current user (CUSTOMER)
                                  or all vehicles (ENGINEER/ADMIN)
   POST   /v1/vehicles         → register a vehicle by VIN (CUSTOMER, SERVICE_PARTNER)
   GET    /v1/vehicles/:id     → get vehicle details (ownership check for CUSTOMER)
   PATCH  /v1/vehicles/:id     → update mileage and registrationNo (owner only)
   GET    /v1/vehicles/:id/summary → vehicle with current warranty status summary

   CreateVehicleDto: vin (validated format: 17 alphanumeric chars),
     modelId (UUID), purchaseDate (optional), registrationNo (optional)

   On vehicle creation: auto-create all three Warranty records using the seeded
   WarrantyType data, setting startDate = purchaseDate (or today), endDate
   calculated from durationMonths.

   VehicleOwner record is also created linking the authenticated user to the vehicle.

3. VehicleModel Controller
   GET /v1/vehicle-models → public endpoint, list all active models with specs

Include all Swagger decorators. Apply JWT auth guard to all vehicle routes.
Scope all queries: CUSTOMER can only access their own vehicles.
```

**Verify:**
- `npx prisma db seed` populates T7 models and warranty types
- `POST /v1/vehicles` with valid VIN creates vehicle + 3 warranty records
- `GET /v1/vehicles` returns only the authenticated user's vehicles

---

#### S5-2 · Vehicle Registry — Frontend

```
Read CLAUDE.md first.

Build the vehicle registry UI in apps/web/src/app/(app)/vehicles/.

1. Vehicle List Page at /app/vehicles
   Fetch GET /v1/vehicles using TanStack Query.
   Display vehicles as cards showing:
     - Vehicle model name and category badge
     - VIN (partially masked: first 4 and last 4 visible)
     - Purchase date
     - Current mileage
     - Warranty status pill: Active (green) / Expiring Soon (amber) / Expired (red)
   "Register a Vehicle" button opens a modal/drawer.
   Empty state: illustration + "No vehicles registered yet. Register your first truck."

2. Register Vehicle Drawer
   Uses shadcn/ui Sheet (slide-in panel from the right).
   Fields:
     - VIN input (validates 17 chars as user types)
     - Model selector (dropdown from GET /v1/vehicle-models)
     - Purchase date (date picker)
     - Registration number (optional)
   On submit: POST /v1/vehicles → close drawer → refetch vehicle list → success toast.
   Show validation errors inline.

3. Vehicle Detail Page at /app/vehicles/[id]
   Tabs: Overview | Warranty | Service History
   Overview tab:
     - Vehicle specs from model (battery, range, GVW, payload)
     - Current mileage with "Update Mileage" inline edit
     - Assigned service partner (if any)
   Warranty tab:
     - List of active warranties with progress bars showing time/mileage remaining
     - "Submit a Claim" button per warranty → links to /app/warranties/claims/new?warrantyId=...
   Service History tab:
     - Chronological list of resolved tickets linked to this vehicle

4. Dashboard Quick Stats
   Update /app/dashboard to show:
     - Total vehicles registered
     - Active warranties count
     - Open tickets count
   Fetch from GET /v1/vehicles and relevant endpoints.

All pages use TanStack Query for data fetching and optimistic updates.
All forms use React Hook Form + Zod.
```

**Verify:**
- Register a vehicle via the drawer → appears in vehicle list with correct model
- Vehicle detail page shows warranty tabs with real data
- Mileage update saves and reflects in the UI

---

### Sprint 6 — Warranty System

---

#### S6-1 · Warranty & Claims — Backend

```
Read CLAUDE.md first.

Build the warranty module in apps/api/src/modules/warranty/ and the file
upload service.

1. File Upload Service (apps/api/src/common/services/file-upload.service.ts)
   Install: @aws-sdk/client-s3, @aws-sdk/s3-request-presigner, multer, @types/multer

   FileUploadService methods:
     uploadFile(file: Express.Multer.File, entityType: string, entityId: string):
       - Validate: allowed types = [image/jpeg, image/png, image/webp, application/pdf]
       - Validate: max size 10MB
       - Generate key: uploads/{entityType}/{entityId}/{uuid}.{ext}
       - Upload to S3/MinIO using PutObjectCommand
       - Return { url, key, mimeType, sizeBytes, filename }
     getSignedUrl(key: string, expiresInSeconds = 3600):
       - Return a pre-signed GET URL valid for 1 hour
     createAttachment(data, prisma): save to Attachment model in DB

   S3 client config: endpoint from S3_ENDPOINT env var (MinIO in dev,
   DO Spaces in prod), forcePathStyle: true for MinIO compatibility.

2. Warranty Module
   GET    /v1/warranties              → warranties for current user's vehicles
   GET    /v1/warranties/:id          → warranty detail with coverage info
   GET    /v1/warranties/:id/claims   → claims for a warranty
   POST   /v1/warranties/:id/claims   → submit a claim (multipart/form-data)
   PATCH  /v1/claims/:id             → update claim status (ENGINEER/ADMIN only)
   GET    /v1/claims                  → all claims (ENGINEER/ADMIN) or own claims (CUSTOMER)

   SubmitClaimDto: title, description, mileageAtClaim (number)
   UpdateClaimDto: status (ClaimStatus), resolution (required if APPROVED or REJECTED)

   POST claim endpoint:
     1. Validate warranty belongs to user's vehicle
     2. Validate warranty status is ACTIVE
     3. Validate mileageAtClaim does not exceed warranty maxMileageKm
     4. Upload attached files using FileUploadService
     5. Create WarrantyClaim record
     6. Create Attachment records linked to claim
     7. Enqueue notification email job

3. BullMQ Email Queue
   Install: @nestjs/bullmq, bullmq, resend
   Create apps/api/src/modules/queue/queue.module.ts
   Create a MailProcessor that handles jobs:
     warranty-claim-submitted: email to assigned engineer with claim details
     claim-status-updated: email to customer with new status and resolution
     warranty-expiry-reminder: email to vehicle owner (30/60/90 days before)

   The warranty-expiry-reminder job is dispatched by a cron job:
   Create a ScheduleModule with a @Cron('0 9 * * *') method that:
     - Finds all ACTIVE warranties expiring in exactly 30, 60, or 90 days
     - Enqueues a reminder email for each

   Use React Email for email templates. Create two templates:
     emails/claim-submitted.tsx: claim details, link to portal
     emails/claim-updated.tsx: new status, resolution text, link to portal
```

**Verify:**
- `POST /v1/warranties/{id}/claims` with a PDF attachment saves to MinIO and creates DB record
- `PATCH /v1/claims/{id}` with `{ status: "APPROVED", resolution: "..." }` updates status
- Email job is queued (visible in BullMQ dashboard if you install bull-board)

---

### Sprint 7 — Parts Catalog

---

#### S7-1 · Parts Catalog — Full Stack

```
Read CLAUDE.md first.

Build the complete parts catalog — backend, Meilisearch integration, and
the frontend list and detail pages.

BACKEND (apps/api/src/modules/parts/):

parts.service.ts:
  search(query, filters): if query string present, use Meilisearch client
    for relevance-ranked results; otherwise use Prisma for filter-only queries.
    Filters: categoryId, modelId, availability. Always paginated.
  findOne(id): return part with category, compatible models, supersession chain
  syncToMeilisearch(partId): index a single part
  reindexAll(): clear Meilisearch index and re-sync all parts from DB

MeilisearchService (common/services/meilisearch.service.ts):
  Configure Meilisearch client from MEILISEARCH_HOST env var.
  On module init: create 'parts' index if not exists, set searchable attributes:
    [partNumber, name, description, category.name]
  and filterable attributes: [categoryId, availability, compatibleModelIds]

parts.controller.ts:
  GET /v1/parts              → search + filter with pagination
  GET /v1/parts/categories   → full category tree (recursive)
  GET /v1/parts/:id          → part detail
  POST /v1/parts/:id/inquiry → creates a Ticket of type PARTS_REQUEST
  POST /v1/admin/parts/reindex → (ADMIN only) trigger full reindex

SEED DATA (add to prisma/seed.ts):
  Create PartCategory tree:
    Electrical System
      → Battery & BMS
      → Motor & Inverter
      → Charging System
    Chassis & Suspension
      → Suspension Components
      → Steering
    Braking System
      → Air Brakes
      → Regenerative Braking
    Cab & Body
      → Cab Glass
      → Seating
    Filters & Fluids
      → Coolant System

  Create 30 sample Parts across these categories with:
    realistic part numbers (format: T7-XXX-NNNNN)
    compatibility with specific T7 models
    mix of IN_STOCK, LOW_STOCK, OUT_OF_STOCK availability
    realistic prices in USD

FRONTEND (apps/web/src/app/(app)/parts/):

Parts List Page at /app/parts:
  Search bar at the top (debounced 300ms, calls API on change)
  Left sidebar: category tree with expandable nodes, checkboxes
  Filter bar: Availability select, Compatible Model select
  Results grid: part card showing part number, name, category, availability
    badge, price, "View Details" and "Request Part" buttons
  Pagination controls
  Loading skeleton state

Parts Detail Page at /app/parts/[id]:
  Part number, name, full description
  Specifications table: weight, compatible models
  Availability status with stock level indicator
  Supersession notice if part is superseded (link to new part)
  "Request this Part" button opens an inquiry form modal
  Inquiry form: vehicle selector (from user's vehicles), message field,
    submits to POST /v1/parts/:id/inquiry

After building, sync seed parts to Meilisearch:
  via POST /v1/admin/parts/reindex endpoint.
```

**Verify:**
- Searching "battery" returns relevant parts
- Filtering by "Out of Stock" shows only unavailable parts
- Part detail page shows compatible models and supersession chain
- "Request this Part" creates a ticket in the DB

---

### Sprint 8 — Service Tickets

---

#### S8-1 · Service Ticket System — Full Stack

```
Read CLAUDE.md first.

Build the complete service ticket system. This is the core support workflow:
customers and service partners open tickets; engineers manage and resolve them.

BACKEND (apps/api/src/modules/tickets/):

tickets.service.ts:
  create(userId, dto): generate ticketNo (format: TKT-YYYY-NNNNN using DB sequence),
    create ticket, enqueue new-ticket notification email
  findAll(user, filters): scope by role:
    CUSTOMER → only own tickets
    SERVICE_PARTNER → tickets for their organisation's vehicles
    ENGINEER/ADMIN → all tickets
    Filter params: status, category, assigneeId, vehicleId
  findOne(id, user): ownership check for CUSTOMER
  update(id, dto, user): update status/assignee/priority, enqueue email on change
  addComment(ticketId, userId, dto): create comment (internal flag restricted to
    ENGINEER/ADMIN role), enqueue notification if public comment
  uploadAttachment(ticketId, file): use FileUploadService, link to ticket

tickets.controller.ts:
  GET    /v1/tickets              → list tickets (role-scoped)
  POST   /v1/tickets              → create ticket
  GET    /v1/tickets/:id          → ticket detail with comments
  PATCH  /v1/tickets/:id          → update status, assignee, priority
  POST   /v1/tickets/:id/comments → add comment
  POST   /v1/tickets/:id/attachments → upload file

BullMQ jobs to add:
  ticket-created: notify assigned engineer (if set) or all engineers
  ticket-comment-added: notify ticket creator (if comment is public and not by them)
  ticket-status-changed: notify ticket creator of new status

FRONTEND (apps/web/src/app/(app)/tickets/):

Ticket List Page at /app/tickets:
  For CUSTOMER: shows own tickets with status badges
  For ENGINEER: shows assigned tickets + unassigned queue
  Columns: Ticket No, Title, Category badge, Status badge, Priority indicator,
    Vehicle (if linked), Created date, Assignee avatar
  Status filter tabs: All | Open | In Progress | Awaiting Customer | Resolved
  "New Ticket" button

New Ticket Page at /app/tickets/new:
  Fields:
    Title (text)
    Category (select: Warranty Claim / Technical Issue / Parts Request / General)
    Description (textarea with markdown hint)
    Vehicle (select from user's registered vehicles, optional)
    Attachments (multi-file upload, max 3 files, shows preview thumbnails)
  On submit: POST /v1/tickets → redirect to /app/tickets/{id}

Ticket Detail Page at /app/tickets/[id]:
  Header: ticket number, title, status badge, priority indicator
  Left column (2/3 width): comment thread
    Each comment shows: avatar, author name, role badge, timestamp, body
    Internal comments (isInternal: true) shown with yellow background only
    to ENGINEER/ADMIN roles — not visible to CUSTOMER
    "Add Comment" form at the bottom (textarea + submit)
    File attachment display with signed URL download links
  Right column (1/3 width): ticket metadata panel
    Status selector (ENGINEER can update)
    Assignee selector (ENGINEER/ADMIN)
    Priority selector (ENGINEER/ADMIN)
    Linked vehicle card
    Created / Updated timestamps
    "Resolve Ticket" button (ENGINEER) → prompts for resolution summary
```

**Verify:**
- Create a ticket as CUSTOMER → appears in ticket list
- Add a comment as ENGINEER with `isInternal: true` → not visible when logged in as CUSTOMER
- Update status to RESOLVED → CUSTOMER receives notification email

---

### Sprint 9 — Admin Dashboard

---

#### S9-1 · Admin Panel

```
Read CLAUDE.md first.

Build the admin dashboard at apps/web/src/app/(app)/admin/ and the
supporting API endpoints. Only ADMIN role users can access this area.
Redirect to /app/dashboard if role is insufficient.

BACKEND — add to existing modules:

GET /v1/admin/stats → summary stats:
  { totalUsers, totalVehicles, openTickets, activeClaims, warrantiesToExpireSoon }
  warrantiesToExpireSoon = warranties expiring in next 30 days

GET /v1/admin/users → paginated user list with search by name/email, filter by role
PATCH /v1/admin/users/:id → update role (cannot demote self), set isActive

GET /v1/admin/reports/warranties → claims grouped by status with counts and
  average resolution time in days for RESOLVED claims

GET /v1/admin/reports/tickets → tickets by category, average first-response time
  (time from createdAt to first comment by ENGINEER), open ticket age histogram

GET /v1/admin/exports/tickets?format=csv → download all tickets as CSV
GET /v1/admin/exports/claims?format=csv  → download all claims as CSV

FRONTEND:

Admin Layout: sidebar with links to all admin sections. Role guard wrapper.

Dashboard Overview at /app/admin:
  4 stat cards: Total Users, Total Vehicles, Open Tickets, Claims Pending Review
  Quick actions: "Impersonate User" (for debugging), "Trigger Reindex" (parts search)

User Management at /app/admin/users:
  Searchable table with columns: Name, Email, Role badge, Status badge,
    Registered date, Last login
  Inline role selector per row (dropdown, saves on change)
  Toggle active/inactive per row
  Export to CSV button

Warranty Reports at /app/admin/reports/warranties:
  Bar chart (using recharts): claims by status
  Stat: average resolution time
  Table: recent 20 claims with status, vehicle, submitter, submitted date

Ticket Reports at /app/admin/reports/tickets:
  Doughnut chart: tickets by category
  Line chart: tickets created per week (last 12 weeks)
  Table: open tickets sorted by age (oldest first) with assignee

All charts use recharts library.
All tables use TanStack Table with sorting.
```

**Verify:**
- `/app/admin` redirects to `/app/dashboard` for non-ADMIN users
- Stats cards show real counts from DB
- CSV export downloads a valid file

---

### Sprint 10 — Polish & Hardening

---

#### S10-1 · Security Audit + Performance

```
Read CLAUDE.md first.

Perform a systematic security and performance hardening pass across
apps/api and apps/web.

SECURITY TASKS:

1. RBAC Ownership Audit
   Review every NestJS service method that touches Vehicle, Warranty,
   WarrantyClaim, Ticket, and TicketComment data.
   For each: confirm there is an explicit ownership check that prevents
   a CUSTOMER from accessing another customer's data.
   Where the check is missing, add it. Log each finding and fix.

2. JWT Security
   Confirm: access token is NOT stored in localStorage or cookies from client
   Confirm: refresh token is httpOnly, Secure, SameSite=Strict cookie
   Confirm: refresh token rotation is working (each refresh invalidates old)
   Confirm: reuse detection works (using an old refresh token invalidates all tokens)
   Write an integration test that verifies reuse detection.

3. Input Sanitisation
   Add @nestjs/helmet to main.ts to set security headers.
   Confirm whitelist: true on ValidationPipe strips unexpected fields.
   Add a test that sends extra fields in a request body and confirms they
   are stripped.

4. File Upload Security
   Confirm FileUploadService validates MIME type from file contents, not
   just the extension or Content-Type header (use 'file-type' npm package).
   Confirm file size limit is enforced before reading the full buffer.
   Confirm uploaded file keys use UUID — never the original filename.

PERFORMANCE TASKS:

5. Database Indexes
   Review all Prisma queries in service files. For each WHERE clause,
   confirm the filtered field has a database index. Add @@index([field])
   to the Prisma schema for any missing indexes. Run migration.
   Focus on: User.email, Vehicle.vin, Ticket.status+createdById,
   Warranty.vehicleId, WarrantyClaim.warrantyId+status

6. API Response Caching
   Add Redis caching (using @nestjs/cache-manager with redis store) to:
   GET /v1/vehicle-models → cache for 1 hour (rarely changes)
   GET /v1/parts/categories → cache for 30 minutes
   Cache is invalidated when admin updates these resources.

7. Next.js Performance
   Run Lighthouse on /app/dashboard. Fix any issue scoring below 80.
   Add loading.tsx skeleton files for:
     /app/vehicles/loading.tsx
     /app/tickets/loading.tsx
     /app/parts/loading.tsx
   Add error.tsx boundary pages for each route group.
   Wrap data-heavy components in React.Suspense with skeleton fallback.

8. Write Integration Tests
   Using Jest + Supertest, write integration tests covering:
   - Authenticated user cannot access another user's vehicle (expect 403)
   - Unauthenticated request to protected route (expect 401)
   - CUSTOMER cannot update claim status (expect 403)
   - Submitting a claim on an expired warranty (expect 400)
   - Submitting a claim on another user's warranty (expect 403)
```

**Verify:**
- All integration tests pass: `npm run test:e2e`
- Lighthouse score on `/app/dashboard` > 80 in all categories
- `helmet` headers visible in response: `X-Content-Type-Options: nosniff`

---

## Phase 3 — FleetOS Foundation (Weeks 25–36)

---

### Sprint 11 — Telemetry Infrastructure

---

#### S11-1 · TimescaleDB + MQTT Broker Setup

```
Read CLAUDE.md first.

Set up the telemetry infrastructure foundation. This is a significant
architectural addition.

STEP 1 — Enable TimescaleDB extension
Connect to the PostgreSQL instance on TrueNAS. Run:
  CREATE EXTENSION IF NOT EXISTS timescaledb CASCADE;

Add a new migration in Prisma that creates the vehicle_telemetry table as
a raw SQL migration (Prisma does not support hypertables natively):

Create file: prisma/migrations/TIMESTAMP_add_timescaledb/migration.sql with:

  CREATE TABLE IF NOT EXISTS vehicle_telemetry (
    id UUID DEFAULT gen_random_uuid(),
    vehicle_id TEXT NOT NULL,
    timestamp TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    soc FLOAT NOT NULL,
    speed_kmh FLOAT,
    odometer_km FLOAT,
    latitude FLOAT,
    longitude FLOAT,
    battery_temp_c FLOAT,
    fault_codes TEXT[] DEFAULT '{}',
    raw_payload JSONB
  );

  SELECT create_hypertable('vehicle_telemetry', 'timestamp',
    if_not_exists => TRUE);

  SELECT add_compression_policy('vehicle_telemetry',
    INTERVAL '7 days', if_not_exists => TRUE);

  SELECT add_retention_policy('vehicle_telemetry',
    INTERVAL '1 year', if_not_exists => TRUE);

  CREATE INDEX ON vehicle_telemetry (vehicle_id, timestamp DESC);

Run the migration. Confirm hypertable is created:
  SELECT * FROM timescaledb_information.hypertables;

STEP 2 — Add EMQX MQTT broker to docker-compose.dev.yml
Add service:
  emqx:
    image: emqx/emqx:5.3.1
    ports:
      - "1883:1883"    (MQTT)
      - "8083:8083"    (WebSocket)
      - "18083:18083"  (Dashboard UI)
    environment:
      EMQX_NODE_NAME: emqx@127.0.0.1
    volumes:
      - emqx-data:/opt/emqx/data

STEP 3 — Telemetry Module in NestJS (apps/api/src/modules/telemetry/)
Install: mqtt, @types/mqtt

telemetry.module.ts: imports PrismaModule, registers the MQTT client.

telemetry.service.ts:
  On module init: connect to MQTT broker (mqtt://emqx:1883 in dev).
  Subscribe to topic: vehicle/+/telemetry (+ is wildcard for vehicleId).
  On message:
    1. Parse JSON payload into TelemetryPayloadDto (validate with class-validator)
    2. Insert into vehicle_telemetry using prisma.$executeRaw (bypass ORM overhead)
    3. Update Redis key fleet:vehicle:{vehicleId}:last_seen with TTL 5 min
    4. Update Redis key fleet:vehicle:{vehicleId}:latest with the telemetry data
    5. Emit a telemetry.received event via NestJS EventEmitter2

TelemetryPayloadDto (validated):
  vehicleId: string (UUID)
  soc: number (min: 0, max: 100)
  speedKmh: number (min: 0)
  odometerKm: number (min: 0)
  latitude: number (optional, min: -90, max: 90)
  longitude: number (optional, min: -180, max: 180)
  batteryTempC: number (optional)
  faultCodes: string[] (optional, default [])
  timestamp: string (ISO 8601, optional — defaults to now)

STEP 4 — Telemetry Simulator Script
Create apps/api/scripts/simulate-telemetry.ts that:
  - Connects to the MQTT broker
  - Publishes realistic telemetry for 5 demo vehicles every 5 seconds
  - Simulates: SOC decreasing slowly, speed varying 0-80kmh, GPS coordinates
    moving along a route, random fault codes appearing occasionally
  - Logs "Published telemetry for vehicle {id}" to console
Run with: npx ts-node scripts/simulate-telemetry.ts
```

**Verify:**
- EMQX dashboard at `http://localhost:18083` shows broker running
- Run simulator → check `SELECT COUNT(*) FROM vehicle_telemetry` increases
- Redis key `fleet:vehicle:{id}:latest` populated with latest telemetry JSON

---

### Sprint 12 — Fleet Dashboard

---

#### S12-1 · Real-Time Fleet Dashboard

```
Read CLAUDE.md first.

Build the real-time fleet dashboard. Vehicles on the map update live as
telemetry arrives.

BACKEND (apps/api/src/modules/fleet/):

fleet.controller.ts:
  GET /v1/fleet/:fleetId/vehicles → list vehicles in fleet with latest telemetry
    For each vehicle: merge Vehicle DB record with Redis latest telemetry.
    Return: id, vin, model, soc, speedKmh, latitude, longitude, isOnline, lastSeenAt

  @Sse() GET /v1/fleet/:fleetId/stream → Server-Sent Events stream
    Uses @nestjs/event-emitter to listen for telemetry.received events.
    Filter events by fleetId (check if vehicleId belongs to this fleet via Redis set).
    Emit MessageEvent with data: { vehicleId, soc, speedKmh, lat, lng, isOnline }
    Send heartbeat every 30 seconds: { type: 'heartbeat' }
    On client disconnect: unsubscribe event listener (memory leak prevention).

  GET /v1/fleet/:fleetId/vehicle/:vehicleId/history → telemetry history
    Query TimescaleDB: SELECT * FROM vehicle_telemetry
    WHERE vehicle_id = ? AND timestamp > NOW() - INTERVAL '24 hours'
    ORDER BY timestamp ASC
    (Uses prisma.$queryRaw)

  GET /v1/fleet/:fleetId/summary → fleet summary stats:
    { totalVehicles, onlineCount, offlineCount, avgSoc, activeFaults }

FRONTEND (apps/web/src/app/(app)/fleet/):

Install: mapbox-gl, @types/mapbox-gl (or use leaflet + react-leaflet if no Mapbox token)

useFleetStream custom hook (src/hooks/use-fleet-stream.ts):
  Uses browser EventSource API to connect to /v1/fleet/{id}/stream
  Updates Zustand fleet store with incoming vehicle positions
  Reconnects automatically with exponential backoff on disconnect
  Cleans up EventSource on unmount

Fleet Dashboard Page at /app/fleet/[id]:
  Top stats bar: Total Vehicles | Online | Offline | Avg SOC | Active Faults
  Main area: Two-column layout on desktop, stacked on mobile:
    Left (40%): Vehicle list cards — sorted by most recently active
      Each card: vehicle name/VIN, model, SOC progress bar with colour gradient
      (green >50%, amber 20-50%, red <20%), speed badge, online/offline dot,
      last seen timestamp. Click → highlight on map and scroll to vehicle detail panel.
    Right (60%): Mapbox/Leaflet map with vehicle markers
      Each marker shows truck icon, coloured by SOC level
      Marker tooltip: VIN, SOC%, speed
      Click marker → opens vehicle side panel
  Vehicle Side Panel (slide-in on vehicle click):
    Latest telemetry values: SOC, speed, battery temp
    SOC history chart (last 24h) using recharts LineChart
    Active fault codes list (badges)
    Link to full vehicle page

Fleet map and vehicle list update in real time via the SSE stream —
no polling required.
```

**Verify:**
- Run the telemetry simulator → vehicle markers appear and move on the map
- SOC progress bars update in real time as simulator changes values
- Disconnect simulator → vehicles show as "Offline" after 5 minutes

---

### Sprint 13 — Fault Monitoring & Alerts

---

#### S13-1 · Fault Code Library + Alert System

```
Read CLAUDE.md first.

Build the fault code monitoring system. When a Critical fault arrives in
telemetry, fleet managers get an immediate alert.

SEED DATA — Add to prisma/seed.ts:
  Create a FaultCode model first:
    Add to schema.prisma:
      model FaultCode {
        id          String   @id @default(cuid())
        code        String   @unique  (e.g. "P0A80")
        system      String   (e.g. "Battery", "Motor", "Charging", "Braking")
        title       String   (e.g. "Battery Pack Degradation Detected")
        description String   (detailed explanation)
        severity    FaultSeverity  (Critical | Warning | Info)
        action      String   (recommended action for operator)
      }
      enum FaultSeverity { CRITICAL | WARNING | INFO }

  Seed 20 common EV fault codes including:
    P0A80 — Battery Pack Degradation Detected (Critical)
    P0AFA — Battery System Voltage Low (Critical)
    P0C73 — Electric Motor Over Temperature (Critical)
    P0D00 — Charging System Fault (Warning)
    P1A0F — Regenerative Braking Reduced (Warning)
    U0100 — CAN Bus Communication Lost (Critical)
    ...and 14 more realistic EV fault codes across all severity levels

Run migration: npx prisma migrate dev --name add-fault-code

BACKEND (extend telemetry module):

fault-monitor.service.ts:
  listenForFaults(): subscribes to telemetry.received events via EventEmitter2
  When a telemetry payload contains fault codes:
    1. Look up each fault code in DB
    2. For CRITICAL faults:
       a. Check Redis: has this fault been alerted for this vehicle in the last 1 hour?
          (key: alert:vehicle:{id}:fault:{code}, TTL 1 hour)
       b. If not in Redis: set the key, enqueue fault-alert job
    3. Store active faults: update Redis set fleet:vehicle:{id}:active-faults

GET /v1/faults → searchable fault code library (public, no auth required)
  Useful for documentation cross-linking.
GET /v1/fleet/:fleetId/faults → active faults for all vehicles in fleet
GET /v1/vehicles/:id/faults/history → fault history from vehicle_telemetry table

BullMQ job processor — fault-alert:
  payload: { vehicleId, faultCode, severity, vehicleVin, fleetManagerEmail,
             fleetManagerPhone (optional) }
  Actions:
    1. Send email via Resend: fault alert email template showing fault code,
       severity, vehicle VIN, recommended action, link to fleet dashboard
    2. If TWILIO_WHATSAPP_FROM env var is set: send WhatsApp message via
       Twilio API to fleetManagerPhone: "🚨 CRITICAL FAULT: {code} on vehicle
       {vin}. {title}. Log in to check: {dashboardUrl}"

Email template (emails/fault-alert.tsx):
  Red alert header for Critical, amber for Warning
  Vehicle VIN prominently displayed
  Fault code, system, description
  Recommended action in a highlighted box
  "View in Fleet Dashboard" CTA button

FRONTEND:
  Fault code library page at /app/faults (accessible without vehicle login)
  Fleet dashboard: show active fault badges on vehicle cards (red for Critical)
  Vehicle detail page: Active Faults section with fault cards
  Alert settings page at /app/fleet/[id]/settings:
    Toggle: enable/disable email alerts
    Toggle: enable/disable WhatsApp alerts
    Phone number field for WhatsApp
```

**Verify:**
- Simulator sends a telemetry payload with `faultCodes: ["P0A80"]`
- Fault alert job is queued (check BullMQ)
- Email received with correct fault details
- Second identical fault within 1 hour does NOT trigger another alert (deduplication works)

---

### Sprint 14 — Predictive Maintenance

---

#### S14-1 · Maintenance Engine

```
Read CLAUDE.md first.

Build the rule-based predictive maintenance engine.

SCHEMA — Add to prisma/schema.prisma:

  model ServiceInterval {
    id              String       @id @default(cuid())
    vehicleModelId  String
    vehicleModel    VehicleModel @relation(fields: [vehicleModelId], references: [id])
    componentName   String       (e.g. "Brake Inspection", "Coolant Flush")
    intervalKm      Float?       (null means time-only interval)
    intervalMonths  Int?         (null means mileage-only interval)
    description     String
    estimatedCost   Float?
  }

  model ServiceRecord {
    id              String    @id @default(cuid())
    vehicleId       String
    vehicle         Vehicle   @relation(fields: [vehicleId], references: [id])
    componentName   String
    performedAt     DateTime
    mileageAtService Float
    notes           String?
    cost            Float?
    performedById   String?
    createdAt       DateTime  @default(now())
  }

Run migration. Seed ServiceInterval records for T7-134L:
  - Brake inspection: every 20,000 km or 6 months (whichever first)
  - Tyre rotation: every 15,000 km
  - Coolant system flush: every 24 months
  - Air filter replacement: every 30,000 km
  - BMS calibration: every 12 months
  - Suspension inspection: every 40,000 km

BACKEND (apps/api/src/modules/maintenance/):

maintenance.service.ts:
  computeVehicleMaintenanceStatus(vehicleId):
    For each ServiceInterval matching vehicle's model:
      1. Find last ServiceRecord for this component (or use vehicle purchaseDate)
      2. Calculate: kmUntilDue = (lastServiceKm + intervalKm) - currentOdometer
      3. Calculate: daysUntilDue = daysBetween(lastServiceDate + intervalMonths, today)
      4. healthScore = min(kmUntilDue/intervalKm, daysUntilDue/(intervalMonths*30)) * 100
         Clamped 0-100. Score below 20 = overdue/due soon.
      5. Return: { componentName, kmUntilDue, daysUntilDue, healthScore, isDue: score < 10 }

  getFleetMaintenanceSummary(fleetId):
    Run computeVehicleMaintenanceStatus for all vehicles in fleet.
    Sort by lowest healthScore. Return paginated.

  BullMQ Cron (@Cron('0 8 * * 1') — every Monday 8am):
    For each active vehicle:
      Run computeVehicleMaintenanceStatus
      For each item with healthScore < 20:
        Check: is there already an OPEN ticket of category MAINTENANCE_DUE
          for this vehicle + componentName? If yes, skip.
        Create Ticket: category GENERAL_INQUIRY, title "Maintenance Due: {component}",
          priority 2, description includes healthScore and km/days remaining.
    Enqueue a weekly-maintenance-summary email to each fleet manager showing
      all vehicles with upcoming maintenance needs.

maintenance.controller.ts:
  GET /v1/vehicles/:id/maintenance     → maintenance status for one vehicle
  GET /v1/fleet/:fleetId/maintenance   → fleet-wide maintenance summary
  POST /v1/vehicles/:id/service        → log a completed service record

FRONTEND — Maintenance tab on vehicle detail page:
  Table of service items with columns:
    Component | Last Service | Next Due (date) | Km Remaining | Health (progress bar)
  Colour coding: green (>50%), amber (20-50%), red (<20%)
  "Log Service" button opens a form to record completed maintenance.

Fleet maintenance overview at /app/fleet/[id]/maintenance:
  Vehicle list sorted by lowest health score
  Summary: X vehicles need attention in next 30 days
```

**Verify:**
- `GET /v1/vehicles/{id}/maintenance` returns maintenance items with healthScores
- A vehicle with mileage approaching a service interval shows healthScore < 50
- The cron job creates a Ticket when healthScore < 20

---

### Sprint 15 — Multi-Tenancy & Billing

---

#### S15-1 · Stripe Subscription Billing

```
Read CLAUDE.md first.

Implement Stripe subscription billing to convert the platform into a SaaS.

Install: stripe (server SDK)

SCHEMA — Add subscription fields to Organization:
  stripeCustomerId    String?  @unique
  stripeSubscriptionId String? @unique
  subscriptionStatus  SubscriptionStatus @default(TRIAL)
  subscriptionTier    SubscriptionTier   @default(FREE)
  trialEndsAt         DateTime?

  enum SubscriptionStatus { TRIAL | ACTIVE | PAST_DUE | CANCELLED | EXPIRED }
  enum SubscriptionTier { FREE | STARTER | GROWTH | SCALE }

Run migration.

BACKEND (apps/api/src/modules/billing/):

billing.service.ts:
  createCheckoutSession(orgId, tier):
    Get or create Stripe customer for org
    Create Stripe Checkout Session with:
      price: STRIPE_PRICE_{tier} (from env vars)
      success_url: {APP_URL}/app/billing/success
      cancel_url: {APP_URL}/app/billing
      metadata: { organizationId: orgId }
    Return { url }

  handleWebhook(event):
    checkout.session.completed:
      Update org: stripeCustomerId, stripeSubscriptionId,
        subscriptionStatus=ACTIVE, subscriptionTier from price metadata
    customer.subscription.updated:
      Update org tier and status from subscription.status
    customer.subscription.deleted:
      Set org subscriptionStatus=CANCELLED, tier=FREE

  createPortalSession(orgId):
    Get org stripeCustomerId
    Create Stripe Customer Portal Session
    Return { url }

billing.controller.ts:
  POST /v1/billing/checkout     → create checkout session (ORG_ADMIN role)
  POST /v1/billing/portal       → create portal session
  POST /v1/billing/webhook      → Stripe webhook (no auth, verify signature)
  GET  /v1/billing/status       → current org subscription status + tier

SubscriptionGuard (common/guards/subscription.guard.ts):
  Decorator @RequiresTier(SubscriptionTier.GROWTH)
  Guard checks org.subscriptionTier against required tier
  If insufficient: throw HttpException 402 with body:
    { error: { code: 'UPGRADE_REQUIRED', requiredTier: 'GROWTH',
               upgradeUrl: '/app/billing' } }

Apply @RequiresTier(GROWTH) to:
  GET /v1/fleet/:id/stream (real-time SSE)
  GET /v1/fleet/:id/maintenance
  All telemetry endpoints

FRONTEND:

Billing Page at /app/billing:
  Current plan display: tier badge, status, renewal date
  Plan comparison table:
    FREE:    Documentation access, warranty tracking, parts catalog — $0/mo
    STARTER: + Service tickets, vehicle registry (up to 10 vehicles) — $99/mo
    GROWTH:  + Real-time fleet dashboard, telemetry, alerts (up to 100) — $499/mo
    SCALE:   + Predictive maintenance, API access (up to 1000) — $1,999/mo
  "Upgrade" button per tier → calls /v1/billing/checkout → redirects to Stripe
  "Manage Subscription" button → calls /v1/billing/portal → redirects to Stripe Portal

Upgrade Prompt Component:
  Shown when SubscriptionGuard returns 402.
  Display: "This feature requires the {requiredTier} plan"
  CTA: "View Plans" → links to /app/billing

Trial Banner:
  If subscriptionStatus is TRIAL and trialEndsAt is within 7 days:
  Show a dismissible yellow banner: "Your trial ends in X days. Upgrade to continue."
```

**Verify:**
- Stripe Checkout opens on clicking "Upgrade to Growth"
- After completing checkout (use Stripe test card 4242 4242 4242 4242):
  org.subscriptionTier updates to GROWTH in DB
- Accessing fleet stream as FREE tier returns 402 with upgrade URL
- GROWTH tier user accesses fleet stream successfully

---

### Sprint 16 — FleetOS Launch

---

#### S16-1 · Onboarding + Production Readiness

```
Read CLAUDE.md first.

Final hardening pass before the FleetOS launch.

1. Fleet Onboarding Wizard
   Create /app/onboarding — a 4-step guided flow for new fleet organisations:
   Step 1: Organisation Setup — name, country, phone, logo upload
   Step 2: Add First Vehicle — VIN input with model auto-detect, or manual select
   Step 3: Configure Alerts — email for fault alerts, optional WhatsApp number,
     toggle which fault severities trigger alerts
   Step 4: Done — celebration screen with links to fleet dashboard and docs

   Show wizard to users whose organisation has zero vehicles registered.
   Can be dismissed and accessed later from Settings.

2. Gateway Setup Documentation
   Add a new page in apps/docs: docs/fleetos/gateway-setup.mdx
   Step-by-step guide for connecting an OBD-II gateway device to the MQTT broker:
   - What hardware is needed (compatible OBD-II adapters list)
   - MQTT broker connection details and where to find them in the portal
   - JSON payload format the gateway must publish
   - Topic format: vehicle/{vehicleId}/telemetry
   - Testing the connection using EMQX dashboard
   - Troubleshooting: no data appearing, authentication errors

3. Monitoring Alerts
   Add Grafana alert rules via docker-compose additions:
   Alert if: API p99 response time > 1 second for 5 minutes
   Alert if: PostgreSQL connections > 80% of max_connections
   Alert if: EMQX broker: connected clients drops to 0 (broker down)
   Alert if: Redis memory usage > 80%
   Alert if: Any Docker container exits unexpectedly

4. Load Test
   Write a k6 load test script (tests/load/fleet-dashboard.js):
   Simulates 50 concurrent fleet managers:
     - Login and get access token
     - Open SSE stream connection to /v1/fleet/{id}/stream
     - Keep connection open for 60 seconds
     - Assert: p95 latency < 500ms, zero 5xx errors

5. Final Pre-Launch Checklist
   Run each of these and confirm they all pass:
   - npm run test (all unit tests)
   - npm run test:e2e (all integration tests)
   - k6 run tests/load/fleet-dashboard.js (load test)
   - npm audit --audit-level=high (no high vulnerabilities)
   - cd apps/web && npm run build (no TypeScript errors)
   - Lighthouse on /app/fleet/{id} page: > 80 all categories
   - Manually verify: register → login → add vehicle → submit warranty claim →
     open ticket → view fleet dashboard → receive fault alert
   Report results of each check.
```

**Verify:**
- Onboarding wizard completes and redirects to fleet dashboard
- Gateway setup doc is visible at `http://localhost:3001/docs/fleetos/gateway-setup`
- Load test shows p95 < 500ms and zero errors
- All checklist items pass

---

## Quick Reference: Prompt Patterns

These patterns work across all prompts throughout the project.

---

### When Claude Code Gets Stuck

If Claude Code produces code that doesn't compile or tests fail, use this recovery prompt:

```
The last task produced errors. Here is the error output:

[paste full error]

Read CLAUDE.md first. Then diagnose the root cause, explain it briefly,
and fix it. Do not change any other working code while fixing this.
After the fix, run the relevant tests to confirm it is resolved.
```

---

### When You Need a Test Written

```
Read CLAUDE.md first.

Write Jest unit tests for apps/api/src/modules/[module]/[service].ts.

Cover these cases:
  1. Happy path: [describe expected behaviour]
  2. [edge case 1]: [what should happen]
  3. [edge case 2]: [what should happen]
  4. Ownership check: [user without access should get 403]

Mock PrismaService and any external services (Resend, S3, Meilisearch).
Use NestJS testing utilities (Test.createTestingModule).
All tests should pass when you run: npm run test -- --testPathPattern=[service]
```

---

### When You Need a Docusaurus Page Translated

```
Read CLAUDE.md first. Pay special attention to the translation rules.

Translate this file to [Chinese Simplified / Bahasa Indonesia]:
Source: [path to English .mdx file]
Output: [path to translated file]

Rules to follow strictly:
  - Preserve ALL MDX import statements exactly as written
  - Preserve ALL component tags (<SpecTable />, <HVSafety />, etc.) exactly
  - Preserve ALL Docusaurus frontmatter keys — only translate the values
  - Preserve ALL admonition markers (:::warning, :::tip, :::)
  - Use professional technical vocabulary for truck operators
  - [Chinese: use Simplified Chinese, mainland standard]
  - [Indonesian: use formal Bahasa Indonesia baku]

After translating, run: cd apps/docs && npm run build
and confirm it compiles without errors.
```

---

### When You Need a New NestJS Module

```
Read CLAUDE.md first.

Create a complete NestJS module for [entity name] in
apps/api/src/modules/[module-name]/.

The Prisma model is:
[paste relevant model from schema.prisma]

Create:
  [module-name].module.ts
  [module-name].controller.ts  with endpoints:
    GET    /v1/[resource]        list (paginated, filtered)
    POST   /v1/[resource]        create
    GET    /v1/[resource]/:id    get one
    PATCH  /v1/[resource]/:id   update
    DELETE /v1/[resource]/:id   soft delete (set isActive=false)
  [module-name].service.ts      business logic, Prisma calls
  dto/create-[name].dto.ts      class-validator decorators
  dto/update-[name].dto.ts      PartialType of create DTO
  dto/[name]-response.dto.ts    safe response shape (no sensitive fields)

Apply:
  - JwtAuthGuard on all routes
  - RolesGuard with appropriate @Roles() per endpoint
  - Ownership scoping: CUSTOMER can only access own records
  - @ApiOperation and @ApiResponse on every endpoint
  - TransformInterceptor wraps all responses automatically

Register the module in AppModule. Confirm no TypeScript errors.
```

---

*Last updated: March 2026 · v1.0 · EV Truck Platform*
