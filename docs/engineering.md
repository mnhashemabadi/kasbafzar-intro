# Engineering note

## Overview

Kasbafzar is a sales desk and a financial report for a business. Sales, customers, and expenses are recorded. The day's report is on the same desk. An invoice can carry a payment link. The desk opens on the web, and from Telegram and Bale.

## Stack

Web app, declared in `apps/web/package.json`:

- Next.js ^14.2.21
- React ^18.3.1
- TypeScript ^5.7.2
- Tailwind CSS 3
- ioredis ^5.10.1
- otplib ^13.4.0

Repository root `package.json`:

- Node.js >= 20
- pnpm 9.15.0
- Prisma 6.19.3
- TypeScript ^5.9.3

Database, `packages/db/schema.prisma` and `packages/db/package.json`:

- PostgreSQL (`datasource db` provider `postgresql`)
- Prisma Client 6.19.3

Telegram bot, `apps/bot/package.json`:

- grammY ^1.30.0
- ioredis ^5.10.1

Bale login, `packages/auth-bale/package.json` and `packages/auth-bale/src/index.ts`:

- Application package `@kasbafzar/auth-bale`
- Depends on `@kasbafzar/auth-core`
- No third-party Bale SDK is declared in that package

Other libraries tied to the public desk:

- Invoice PDF: pdfmake ^0.2.15 in `packages/pdf/package.json`
- Redis cache and queue clients: ioredis ^5.10.1 in `packages/cache/package.json` and `packages/queue/package.json`

## Specialties

- Sales, customer, and expense records: Prisma models on PostgreSQL, including `Customer`, `Expense`, and `Invoice` in `packages/db/schema.prisma`
- Invoicing: those invoice models, plus PDF output through pdfmake
- Payments: application package `@kasbafzar/payments`, stored through Prisma. `packages/payments/package.json` declares no third-party payment SDK
- Authentication: `@kasbafzar/auth-core` and otplib on the web app; Telegram through grammY; Bale through `@kasbafzar/auth-bale`
- Cache and queues: Redis through ioredis

## Boundaries

This note covers the public desk: web app, Telegram, Bale, records, invoices, and payment links. It does not describe hosting or credentials.
