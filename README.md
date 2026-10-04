# کسب‌افزار

کسب‌افزار پیشخوان فروش و گزارش مالی برای کسب‌وکار است. فروش، مشتری و هزینه ثبت می‌شود و گزارش روز در همان پیشخوان است. فاکتور می‌تواند لینک پرداخت داشته باشد. پیشخوان از وب باز می‌شود و ثبت از تلگرام و بله هم در صفحهٔ عمومی آمده است.

سایت: [kasbafzar.ir](https://kasbafzar.ir)

نسخهٔ بستهٔ وب `@kasbafzar/web` در `apps/web/package.json` برابر `0.28.0` است و صفحهٔ عمومی همان رشتهٔ نسخه را نشان می‌دهد.

## رفتار عمومی

صفحهٔ [kasbafzar.ir](https://kasbafzar.ir) پیشخوان را برای فروشگاه و شرکت کوچک توصیف می‌کند: فاکتور، هزینه، بدهی مشتری، گزارش امروز، لینک پرداخت، یادآوری بدهی، و ثبت از تلگرام یا بله. شروع را رایگان و بدون نصب می‌نویسد. دعوت همکار با نقش (صندوق، حسابداری، مدیریت) و چند شعبه در همان متن آمده است. ویترین، محصول، و موجودی هم در فهرست کارهای صفحه هست. اعداد روی پیشخوان نمونهٔ همان صفحه، دادهٔ نمایشی‌اند و آمار عملیاتی نیستند.

## English

Kasbafzar is a sales desk and a financial report for a business. Sales, customers, and expenses are recorded. The day's report is on the same desk. An invoice can carry a payment link. The desk opens on the web. The public page also describes recording from Telegram and Bale.

Site: [kasbafzar.ir](https://kasbafzar.ir)

The web package version in `apps/web/package.json` is `0.28.0`, and the public homepage shows that same version string.

The homepage describes invoicing, expenses, customer debt, today's report, a payment link, a debt reminder, and entry from Telegram or Bale. It describes a free start with nothing to install, inviting a colleague into a role, and more than one branch. Storefront, product, and stock appear in that same public description. Figures drawn inside the sample desk on that page are sample figures, not operational metrics.

## Parts

The repository is a pnpm workspace. Root `package.json` sets Node.js `>=20`, `packageManager` `pnpm@9.15.0`, Prisma `6.19.3` (also pinned in `pnpm.overrides` for `prisma` and `@prisma/client`), TypeScript `^5.9.3`, and Turbo `^2.9.14`.

The desk UI and HTTP API are the Next.js app `apps/web` (`@kasbafzar/web`). Panel pages under `apps/web/src/app/app/(panel)/` include:

- `invoices/page.tsx`, `invoices/composer/page.tsx`, and `invoices/composer/[publicSlug]/page.tsx`
- `customers/page.tsx` and `customers/[id]/page.tsx`
- `accounting/page.tsx`
- `products/page.tsx`
- `store/page.tsx` and `store/products/new/page.tsx`
- `storefront/page.tsx`, `storefront/products/page.tsx`, and `storefront/settings/page.tsx`
- `offers/page.tsx`
- `billing/page.tsx`
- `bots/page.tsx`
- `settings/page.tsx`, `settings/business/page.tsx`, `settings/invoice/page.tsx`, and `settings/growth/page.tsx`

A public invoice page is `apps/web/src/app/(distribution)/invoice/[publicSlug]/page.tsx`. Storefront and market pages live under `apps/web/src/app/(distribution)/`.

The messenger process is `apps/bot` (`@kasbafzar/bot`). PostgreSQL is Prisma, schema `packages/db/schema.prisma`, provider `postgresql`. Client package `@kasbafzar/db` depends on `@prisma/client` `6.19.3`.

## Invoice request

`POST /api/invoice/create` is `apps/web/src/app/api/invoice/create/route.ts` on the Node.js runtime. The body is checked with Zod:

- `storeId` (cuid)
- `customerName` (1–100 characters)
- optional `customerPhone`, matched to `^(0|\+98)9[0-9]{9}$`
- `items`: 1 to 50 lines, each with `title` (1–200), integer `qty` greater than 0, integer `price` greater than 0, optional integer `taxRate` from 0 to 100
- optional `autoFinalize` boolean
- optional `visibility` of `private` or `public`

`requireStoreForUser("invoice.create")` must succeed, and the store id in the body must be that store. `assertActiveBilling(storeId)` and `assertCanCreateInvoice(storeId)` run before the draft. `autoFinalize` defaults to true when the header `x-bot-source` is `telegram`, `bale`, or `rubika`.

`createDraftInvoice` writes the invoice. If `publicSlug` is empty, `uniqueInvoicePublicId` generates one and checks `prisma.invoice.findUnique` on `publicSlug` (up to eight tries). For a non-private invoice the handler builds a public URL `/invoice/<publicSlug>` and share links through `buildInvoiceShareLinks`. `syncCustomerFromInvoice` receives the store id, customer name, phone, and total. When status is `finalized` and the invoice is not private, `enqueuePDFJob(invoice.id, publicSlug)` is called. The handler returns HTTP 201 with id, invoiceNumber, displayNumber, status, visibility, publicSlug (null when private), publicUrl, shareLinks, and pdfStatus (`pending` when finalized and not private, otherwise `draft`).

Other invoice routes in the same app include `apps/web/src/app/api/app/invoices/route.ts`, `apps/web/src/app/api/invoice/[publicSlug]/route.ts`, and finalize, correct, cancel, and event routes under `apps/web/src/app/api/invoice/[publicSlug]/`.

Expense routes seen beside that flow: `apps/web/src/app/api/commerce/expense/route.ts`, `apps/web/src/app/api/accounting/expense/route.ts`, and `apps/web/src/app/api/app/dashboard/expense/route.ts`. Customer pages call `apps/web/src/app/api/app/customers/[id]/timeline/route.ts`.

## Data

Prisma models used by the desk, in `packages/db/schema.prisma`:

- `Store`: name, slug, status, address, phone, tax fields, invoice profile, and relations to invoices, customers, products, and expenses.
- `Customer`: store, name, phone, address, national id, economic code, `totalSpent`, `invoiceCount`, `lastPurchaseAt`.
- `Expense`: store, category, amount, note, `createdAt`.
- `Invoice`: store, invoice number, display number, status, customer name and phone, items, subtotal, discount, shipping, tax, duties, total, payment status, amount paid, and timestamps including `finalizedAt` and `canceledAt`. The create route also reads and writes `publicSlug`.
- `Product`: store, name, price, description, images, stock, sku, active flag.
- `InventoryItem`: store, name, `stockQty`, `lowStockAt`, purchase and sell price, sku.
- `LedgerTransaction`: store, type, amount, category, source, optional linked invoice, note.
- `PaymentIntent`: store, optional invoice, amount, currency, status, `paymentUrl`, and paid/failed timestamps.

`@kasbafzar/payments` (`packages/payments/package.json`) depends on workspace packages only. No third-party payment SDK is declared there. The public page's payment link is the product behavior; a gateway product name is not taken from that package file.

PDF output is `@kasbafzar/pdf`. `packages/pdf/package.json` depends on `pdfmake` `^0.2.15` and `ioredis` `^5.10.1`. `packages/pdf/src/index.ts` exports `enqueuePDFJob`, status helpers, and `buildCeoReportPDF`, which imports pdfmake only when a report is generated. The bot process calls `startPDFWorker()` from `apps/bot/src/index.ts`.

QR images are `@kasbafzar/qr`, library `qrcode` `^1.5.4`. `generateQRBuffer` and `generateQRFile` are in `packages/qr/src/index.ts`.

## Messengers and login

`apps/bot/package.json` depends on `grammy` `^1.30.0` and `ioredis` `^5.10.1`. `apps/bot/src/index.ts` either polls (`startBotPolling`, and its log line names Bale and Telegram `getUpdates`) or initializes webhook mode, where updates arrive at the Next.js bot webhook routes. The same process starts the PDF worker, the outgoing worker, the daily push worker, habit workers, and `startAutonomousWorker`.

Bale login is application code. `packages/auth-bale/package.json` depends only on `@kasbafzar/auth-core`. `packages/auth-bale/src/index.ts` exports `buildBaleLoginDeepLink`, which calls `buildMessengerLoginDeepLink` with method `bale`. No third-party Bale SDK is in that package. The web app also depends on `@kasbafzar/auth-rubika`. One-time passwords on the web app use `otplib` `^13.4.0`.

Redis clients for cache and queue are `ioredis` `^5.10.1` in `packages/cache/package.json` and `packages/queue/package.json`. `@kasbafzar/realtime` is a workspace package depending on `@kasbafzar/event-bus`; its `package.json` does not add another network library.

## Libraries on the web app

From `apps/web/package.json`:

- next ^14.2.21
- react ^18.3.1, react-dom ^18.3.1
- typescript ^5.7.2
- tailwindcss 3
- @tanstack/react-query ^5.62.8
- @tanstack/react-virtual ^3.13.0
- zod ^4.4.3
- ioredis ^5.10.1
- otplib ^13.4.0
- grammy ^1.30.0
- nanoid ^5.1.11
- nodemailer ^6.10.1
- xlsx ^0.18.5

Workspace dependencies of that app include `@kasbafzar/db`, `@kasbafzar/auth-core`, `@kasbafzar/auth-bale`, `@kasbafzar/payments`, `@kasbafzar/pdf`, `@kasbafzar/qr`, `@kasbafzar/sales`, `@kasbafzar/ledger`, `@kasbafzar/customers-lite`, and `@kasbafzar/store`, among others declared in the same file.

## Boundaries

Hosting and credentials are omitted. Modules outside the desk, the messengers, invoices, expenses, customers, and the short-link routes are not described here. Short links are the subject of the Afzi introduction.

## پروژه‌های مرتبط

- [hamejoo-intro](https://github.com/mnhashemabadi/hamejoo-intro): همه‌جو نتیجهٔ جستجوی آگهی را یک‌جا نشان می‌دهد و جزئیات آگهی روی منبع اصلی می‌ماند.
- [alwer-intro](https://github.com/mnhashemabadi/alwer-intro): آلور بازار نیازمندی است؛ درخواست خرید ثبت می‌شود، فروشنده پیشنهاد قیمت می‌فرستد، و آگهی فروش هم در همان بازار است.
- [azadchi-intro](https://github.com/mnhashemabadi/azadchi-intro): آزادچی بازار نیازمندی مناطق آزاد و ویژهٔ اقتصادی است؛ آگهی در همان محدوده جستجو می‌شود و گفتگو داخل همان محصول است.
- [afzi-intro](https://github.com/mnhashemabadi/afzi-intro): افزی یک نشانی http یا https را به لینک کوتاه تبدیل می‌کند و باز کردن آن لینک به همان صفحه می‌رود.
- [alweryar-intro](https://github.com/mnhashemabadi/alweryar-intro): آلوریار شبکهٔ همکاران آلور برای بررسی آگهی و همکاری در فروش است.
- [alwerchi-intro](https://github.com/mnhashemabadi/alwerchi-intro): آلورچی خانهٔ فروشگاه‌هایی است که کالا و موجودی‌شان در بازار آلور دیده می‌شود و خریدار در آلور می‌ماند.

## Related

- [hamejoo-intro](https://github.com/mnhashemabadi/hamejoo-intro): Hamejoo shows listing search results in one place, and the listing detail stays on the original source.
- [alwer-intro](https://github.com/mnhashemabadi/alwer-intro): Alwer is a classifieds marketplace: a buyer posts a request, sellers send price offers, and a sale listing can be posted on the same market.
- [azadchi-intro](https://github.com/mnhashemabadi/azadchi-intro): Azadchi is a classifieds market for free zones and special economic zones: listings are searched in that area, and the conversation stays in the product.
- [afzi-intro](https://github.com/mnhashemabadi/afzi-intro): Afzi turns an http or https address into a short link, and opening that link goes to the same page.
- [alweryar-intro](https://github.com/mnhashemabadi/alweryar-intro): Alweryar is Alwer's collaborator network for listing review and for sales collaboration.
- [alwerchi-intro](https://github.com/mnhashemabadi/alwerchi-intro): Alwerchi is the home of shops whose goods and stock appear on the Alwer marketplace while the buyer stays on Alwer.

یادداشت مهندسی کوتاه‌تر: [docs/engineering.md](docs/engineering.md)

Shorter engineering note: [docs/engineering.md](docs/engineering.md)

