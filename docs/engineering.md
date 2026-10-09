# Engineering note

## Overview

Kasbafzar is a sales desk and a financial report for a business. Sales, customers, and expenses are recorded. The day's report is on the same desk. An invoice can carry a payment link. The desk opens on the web, and from Telegram and Bale.

## Stack

Web app:

- Next.js ^14.2.21
- React ^18.3.1
- TypeScript ^5.7.2
- Tailwind CSS 3
- Zod ^4.4.3
- ioredis ^5.10.1

Repository:

- Node.js >= 20
- pnpm 9.15.0 workspace
- Turbo ^2.9.14
- TypeScript ^5.9.3

Database:

- PostgreSQL
- Prisma 6.19.3

Telegram bot:

- grammY ^1.30.0
- ioredis ^5.10.1

Other libraries:

- Invoice PDF: pdfmake ^0.2.15
- QR: qrcode ^1.5.4
- Redis cache and queue clients: ioredis ^5.10.1

## Specialties

- Sales, customer, and expense records on PostgreSQL through Prisma
- Invoicing with public share links, PDF output through pdfmake, and QR codes
- Messenger entry: Telegram through grammY, and Bale
- Cache and background queues: Redis through ioredis
