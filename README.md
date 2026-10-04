# کسب‌افزار

من کسب‌افزار را پیشخوان فروش گذاشتم، نه یک صفحهٔ گزارش جدا. فروش و مشتری و هزینه باید همان‌جا ثبت شود که گزارش امروز خوانده می‌شود. فاکتور اگر عمومی باشد لینک دارد تا مشتری بدون ورود به پیشخوان آن را ببیند. پیشخوان وب است. ثبت از تلگرام و بله را روی صفحهٔ عمومی نوشتم چون صاحب فروشگاه همیشه پشت مرورگر نیست.

سایت: [kasbafzar.ir](https://kasbafzar.ir)

نسخهٔ `@kasbafzar/web` در `apps/web/package.json` برابر `0.28.0` است. همان رشته روی صفحهٔ عمومی است.

## آنچه صفحهٔ عمومی می‌گوید

صفحه پیشخوان را برای فروشگاه و شرکت کوچک توصیف می‌کند: فاکتور، هزینه، بدهی مشتری، گزارش امروز، لینک پرداخت، یادآوری بدهی، و ثبت از تلگرام یا بله. شروع را رایگان و بدون نصب نوشته است. دعوت همکار با نقش و چند شعبه در همان متن است. ویترین و محصول و موجودی هم در فهرست کارهای صفحه هست. صفحه برچسب ERP و CRM دارد و یک دستیار تصمیم را «مبتنی بر LLM» می‌نامد و می‌گوید داده روی سرور ایران است. نام مدل را نه صفحه نوشته و نه من اینجا می‌نویسم. عددهایی که داخل پیشخوان نمونه کشیده شده، مثل فروش امروز آن قاب، دادهٔ نمایشی همان صفحه است نه آمار عملیاتی که من از پایگاه خوانده باشم.

## یک مخزن، پیشخوان و ربات

ریشه workspace با pnpm است. `package.json` ریشه Node.js `>=20`، `packageManager` برابر `pnpm@9.15.0`، Prisma `6.19.3` (و همان نسخه در `pnpm.overrides` برای `prisma` و `@prisma/client`)، TypeScript `^5.9.3` و Turbo `^2.9.14` را اعلام می‌کند. Prisma را پین کردم تا طرح پایگاه و کلاینت از هم جدا نشوند.

پیشخوان و HTTP هر دو اپ Next.js `apps/web` هستند (`@kasbafzar/web`): Next ^14.2.21، React ^18.3.1، TypeScript ^5.7.2، Zod ^4.4.3، `ioredis` ^5.10.1، `otplib` ^13.4.0. صفحه‌های پنل زیر `apps/web/src/app/app/(panel)/` را برای کار روزانه گذاشتم، نه برای تنظیم پنهان: فاکتورها و سازندهٔ فاکتور، مشتری و صفحهٔ یک مشتری، حسابداری، محصول، فروشگاه، ویترین، پیشنهادها، صورتحساب، ربات‌ها، و تنظیم کسب‌وکار و فاکتور. فاکتور عمومی `apps/web/src/app/(distribution)/invoice/[publicSlug]/page.tsx` است. ویترین و بازار زیر `apps/web/src/app/(distribution)/` هستند. فاکتور عمومی را بیرون پنل گذاشتم تا لینک مشتری به صفحهٔ مدیریت نخورد.

فرآیند پیام‌رسان `apps/bot` است (`@kasbafzar/bot`). Postgres با Prisma است، طرح `packages/db/schema.prisma`، provider برابر `postgresql`. بستهٔ `@kasbafzar/db` به `@prisma/client` نسخهٔ `6.19.3` وابسته است.

## ساخت فاکتور

`POST /api/invoice/create` در `apps/web/src/app/api/invoice/create/route.ts` روی runtime برابر `nodejs` است. بدنه را با Zod می‌بندم:

- `storeId` از نوع cuid
- `customerName` از ۱ تا ۱۰۰ نویسه
- `customerPhone` اختیاری، فقط موبایل ایران با `^(0|\+98)9[0-9]{9}$`
- `items` از ۱ تا ۵۰ سطر؛ هر سطر `title` از ۱ تا ۲۰۰، `qty` صحیح بزرگ‌تر از صفر، `price` صحیح بزرگ‌تر از صفر، `taxRate` اختیاری از ۰ تا ۱۰۰
- `autoFinalize` اختیاری
- `visibility` برابر `private` یا `public`

`requireStoreForUser("invoice.create")` باید موفق شود و شناسهٔ فروشگاه بدنه همان فروشگاه باشد. قبل از پیش‌نویس `assertActiveBilling(storeId)` و `assertCanCreateInvoice(storeId)` را می‌زنم تا فاکتور بدون فروشگاه فعال ساخته نشود. اگر `autoFinalize` نیامده باشد و هدر `x-bot-source` برابر `telegram` یا `bale` یا `rubika` باشد، نهایی‌کردن را روشن می‌کنم. از پیام‌رسان یک پیش‌نویس نیمه‌کاره به دست فروشنده نمی‌دهم؛ از وب، پیش‌فرض همان مقداری است که کلاینت فرستاده.

`createDraftInvoice` فاکتور را می‌نویسد. اگر `publicSlug` خالی باشد، `uniqueInvoicePublicId` تا هشت بار شناسه می‌سازد و با `prisma.invoice.findUnique` روی `publicSlug` تکراری نبودن را چک می‌کند. شناسه را از `generateInvoicePublicId` در `src/lib/afzi/ids` می‌گیرم تا لینک عمومی فاکتور با کد لینک کوتاه افزی تداخل نکند. برای فاکتور غیرخصوصی نشانی `/invoice/<publicSlug>` و لینک‌های اشتراک از `buildInvoiceShareLinks` ساخته می‌شود. `syncCustomerFromInvoice` شناسهٔ فروشگاه، نام، تلفن و جمع را می‌گیرد تا مشتری از روی فاکتور دوباره دستی ساخته نشود. اگر وضعیت `finalized` باشد و فاکتور خصوصی نباشد، `enqueuePDFJob` را صدا می‌زنم. پاسخ HTTP 201 است: id، invoiceNumber، displayNumber، status، visibility، publicSlug (برای خصوصی null)، publicUrl، shareLinks، و pdfStatus برابر `pending` وقتی نهایی و غیرخصوصی است وگرنه `draft`.

مسیرهای دیگر فاکتور کنار همین جریان‌اند: `apps/web/src/app/api/app/invoices/route.ts`، `apps/web/src/app/api/invoice/[publicSlug]/route.ts`، و مسیرهای finalize و correct و cancel و event زیر `apps/web/src/app/api/invoice/[publicSlug]/`. هزینه را جدا نگه داشتم: `apps/web/src/app/api/commerce/expense/route.ts`، `apps/web/src/app/api/accounting/expense/route.ts`، و `apps/web/src/app/api/app/dashboard/expense/route.ts`. صفحهٔ مشتری خط زمان را از `apps/web/src/app/api/app/customers/[id]/timeline/route.ts` می‌خواند.

## داده‌ای که پیشخوان به آن تکیه می‌کند

در `packages/db/schema.prisma` مدل‌هایی که این پیشخوان را سر پا نگه می‌دارند:

- `Store`: نام، نامک، وضعیت، نشانی، تلفن، فیلدهای مالیاتی، نمایهٔ فاکتور، و رابطه با فاکتور و مشتری و محصول و هزینه.
- `Customer`: فروشگاه، نام، تلفن، نشانی، و جمع‌های `totalSpent` و `invoiceCount` و `lastPurchaseAt` تا گزارش مشتری از جمع فاکتورها هر بار حساب نشود.
- `Expense`: فروشگاه، دسته، مبلغ، یادداشت، `createdAt`.
- `Invoice`: فروشگاه، شماره، شمارهٔ نمایشی، وضعیت، نام و تلفن مشتری، اقلام، جمع جزء، تخفیف، ارسال، مالیات، عوارض، جمع، وضعیت پرداخت، مبلغ پرداخت‌شده، و زمان‌ها از جمله `finalizedAt` و `canceledAt`. مسیر ساخت `publicSlug` را هم می‌خواند و می‌نویسد.
- `Product`: فروشگاه، نام، قیمت، توضیح، تصویر، موجودی، sku، پرچم فعال.
- `InventoryItem`: فروشگاه، نام، `stockQty`، `lowStockAt`، قیمت خرید و فروش، sku. آستانهٔ موجودی کم را روی خود ردیف گذاشتم تا گزارش امروز به یک جدول جدا برای هشدار محتاج نباشد.
- `LedgerTransaction`: فروشگاه، نوع، مبلغ، دسته، منبع، فاکتور اختیاری، یادداشت.
- `PaymentIntent`: فروشگاه، فاکتور اختیاری، مبلغ، ارز، وضعیت، `paymentUrl`، و زمان پرداخت یا شکست.

`@kasbafzar/payments` در `packages/payments/package.json` فقط به بسته‌های همین workspace وابسته است. SDK درگاه شخص ثالث آنجا اعلام نشده. لینک پرداختی که صفحهٔ عمومی می‌گوید رفتار محصول است؛ نام درگاه را از آن فایل برنمی‌دارم.

PDF بستهٔ `@kasbafzar/pdf` است. `packages/pdf/package.json` به `pdfmake` ^0.2.15 و `ioredis` ^5.10.1 وابسته است. `packages/pdf/src/index.ts` تابع `enqueuePDFJob` و کمک‌وضعیت و `buildCeoReportPDF` را صادر می‌کند و pdfmake را فقط وقتی گزارش ساخته می‌شود وارد می‌کند. فرآیند ربات `startPDFWorker()` را از `apps/bot/src/index.ts` صدا می‌زند. صف PDF را داخل فرآیند وب نگذاشتم تا ساخت فایل درخواست صفحه را نگه ندارد.

QR بستهٔ `@kasbafzar/qr` است، کتابخانه `qrcode` ^1.5.4. `generateQRBuffer` و `generateQRFile` در `packages/qr/src/index.ts` هستند تا لینک فاکتور روی کاغذ هم خوانده شود.

ربات تلگرام را با grammY ^1.30.0 در `apps/bot/package.json` نوشتم. `apps/bot/src/router.ts` به‌روزرسانی grammY را به یک شکل داخلی تبدیل می‌کند و منبع را همراهش نگه می‌دارد. ورود بله بستهٔ `@kasbafzar/auth-bale` است و به `@kasbafzar/auth-core` وابسته است؛ SDK جداگانهٔ بله در آن بسته اعلام نشده. صفحهٔ عمومی تلگرام و بله را نام می‌برد. هدر ساخت فاکتور روبیکا را هم به‌عنوان `x-bot-source` می‌پذیرد.

میزبانی و رمزها را اینجا نیاوردم. افزی لینک کوتاه است و مسیرهایش داخل همین اپ وب است؛ معرفی خودش جداست.

## English

I built Kasbafzar as the sales desk, not as a separate report page. A sale, a customer, and an expense have to be recorded in the same place the day's report is read. If an invoice is public it carries a link so the customer can open it without entering the desk. The desk is the web. I wrote Telegram and Bale entry on the public page because the shop owner is not always in a browser.

Site: [kasbafzar.ir](https://kasbafzar.ir)

The `@kasbafzar/web` version in `apps/web/package.json` is `0.28.0`. The public page shows that same string.

### What the public page says

The page describes the desk for a shop and a small company: invoice, expense, customer debt, today's report, a payment link, a debt reminder, and entry from Telegram or Bale. It says the start is free and nothing is installed. Inviting a colleague into a role, and more than one branch, are in that same text. Storefront, product, and stock are in the page's list of work. The page labels the desk ERP and CRM, names a decision assistant as LLM-based, and says the data is on a server in Iran. The page does not name a model, and I am not naming one here. Figures drawn inside the sample desk, including the "sales today" frame, are sample figures on that page, not operational totals I read from the database.

### One repository, desk and bot

The root is a pnpm workspace. Root `package.json` declares Node.js `>=20`, `packageManager` `pnpm@9.15.0`, Prisma `6.19.3` (the same pin in `pnpm.overrides` for `prisma` and `@prisma/client`), TypeScript `^5.9.3`, and Turbo `^2.9.14`. I pinned Prisma so the schema and the client do not drift apart.

The desk and the HTTP API are both the Next.js app `apps/web` (`@kasbafzar/web`): Next ^14.2.21, React ^18.3.1, TypeScript ^5.7.2, Zod ^4.4.3, `ioredis` ^5.10.1, `otplib` ^13.4.0. I put the panel pages under `apps/web/src/app/app/(panel)/` for the day's work, not for a hidden settings tree: invoices and the invoice composer, customers and one customer, accounting, products, the store, the storefront, offers, billing, bots, and business and invoice settings. The public invoice is `apps/web/src/app/(distribution)/invoice/[publicSlug]/page.tsx`. Storefront and market pages live under `apps/web/src/app/(distribution)/`. I put the public invoice outside the panel so a customer link does not open the management UI.

The messenger process is `apps/bot` (`@kasbafzar/bot`). Postgres is Prisma, schema `packages/db/schema.prisma`, provider `postgresql`. `@kasbafzar/db` depends on `@prisma/client` `6.19.3`.

### Creating an invoice

`POST /api/invoice/create` in `apps/web/src/app/api/invoice/create/route.ts` runs with `runtime` `nodejs`. I check the body with Zod:

- `storeId` as a cuid
- `customerName`, 1 to 100 characters
- optional `customerPhone`, Iranian mobile only, `^(0|\+98)9[0-9]{9}$`
- `items`, 1 to 50 lines; each line `title` 1 to 200, integer `qty` greater than 0, integer `price` greater than 0, optional integer `taxRate` from 0 to 100
- optional `autoFinalize`
- `visibility` of `private` or `public`

`requireStoreForUser("invoice.create")` must succeed, and the store id in the body must be that store. Before the draft I run `assertActiveBilling(storeId)` and `assertCanCreateInvoice(storeId)` so an invoice is not created for a store that is not allowed to bill. If `autoFinalize` is omitted and the header `x-bot-source` is `telegram`, `bale`, or `rubika`, I turn finalizing on. From a messenger I do not hand the seller a half-finished draft. From the web, the default is whatever the client sent.

`createDraftInvoice` writes the invoice. If `publicSlug` is empty, `uniqueInvoicePublicId` generates an id up to eight times and checks `prisma.invoice.findUnique` on `publicSlug`. I take the id from `generateInvoicePublicId` in `src/lib/afzi/ids` so a public invoice link does not collide with an Afzi short code. For a non-private invoice I build `/invoice/<publicSlug>` and share links through `buildInvoiceShareLinks`. `syncCustomerFromInvoice` receives the store id, name, phone, and total, so the customer is not typed again by hand from the invoice. When status is `finalized` and the invoice is not private, I call `enqueuePDFJob`. The response is HTTP 201: id, invoiceNumber, displayNumber, status, visibility, publicSlug (null when private), publicUrl, shareLinks, and pdfStatus `pending` when finalized and not private, otherwise `draft`.

Other invoice routes sit beside that flow: `apps/web/src/app/api/app/invoices/route.ts`, `apps/web/src/app/api/invoice/[publicSlug]/route.ts`, and finalize, correct, cancel, and event routes under `apps/web/src/app/api/invoice/[publicSlug]/`. I kept expenses separate: `apps/web/src/app/api/commerce/expense/route.ts`, `apps/web/src/app/api/accounting/expense/route.ts`, and `apps/web/src/app/api/app/dashboard/expense/route.ts`. The customer page reads the timeline from `apps/web/src/app/api/app/customers/[id]/timeline/route.ts`.

### The data the desk actually uses

In `packages/db/schema.prisma`, the models this desk stands on:

- `Store`: name, slug, status, address, phone, tax fields, invoice profile, and relations to invoices, customers, products, and expenses.
- `Customer`: store, name, phone, address, and the rollups `totalSpent`, `invoiceCount`, and `lastPurchaseAt`, so a customer report is not recomputed from every invoice on each read.
- `Expense`: store, category, amount, note, `createdAt`.
- `Invoice`: store, number, display number, status, customer name and phone, items, subtotal, discount, shipping, tax, duties, total, payment status, amount paid, and timestamps including `finalizedAt` and `canceledAt`. The create route also reads and writes `publicSlug`.
- `Product`: store, name, price, description, images, stock, sku, active flag.
- `InventoryItem`: store, name, `stockQty`, `lowStockAt`, purchase and sell price, sku. I put the low-stock threshold on the row itself so today's report does not need a separate alert table.
- `LedgerTransaction`: store, type, amount, category, source, optional linked invoice, note.
- `PaymentIntent`: store, optional invoice, amount, currency, status, `paymentUrl`, and paid or failed timestamps.

`@kasbafzar/payments` in `packages/payments/package.json` depends only on workspace packages. No third-party gateway SDK is declared there. The payment link on the public page is the product behavior. I am not taking a gateway name from that file.

PDF is `@kasbafzar/pdf`. `packages/pdf/package.json` depends on `pdfmake` ^0.2.15 and `ioredis` ^5.10.1. `packages/pdf/src/index.ts` exports `enqueuePDFJob`, status helpers, and `buildCeoReportPDF`, and it imports pdfmake only when a report is generated. The bot process calls `startPDFWorker()` from `apps/bot/src/index.ts`. I did not put the PDF queue inside the web request, so building the file does not hold the page.

QR is `@kasbafzar/qr`, library `qrcode` ^1.5.4. `generateQRBuffer` and `generateQRFile` are in `packages/qr/src/index.ts`, so an invoice link can be read off paper.

I wrote the Telegram bot with grammY ^1.30.0 in `apps/bot/package.json`. `apps/bot/src/router.ts` turns a grammY update into one internal shape and keeps the source on it. Bale login is `@kasbafzar/auth-bale`, which depends on `@kasbafzar/auth-core`. That package does not declare a separate Bale SDK. The public page names Telegram and Bale. The invoice create header also accepts Rubika as `x-bot-source`.

I am not putting hosting or credentials here. Afzi is the short-link product and its routes live in this same web app. Its introduction is separate.

## پروژه‌های مرتبط

- [hamejoo-intro](https://github.com/mnhashemabadi/hamejoo-intro): همه‌جو را برای بازدیدکننده‌ای گذاشتم که با یک عبارت آگهی را یک‌جا ببیند و برای جزئیات به سایت منبع برود.
- [alwer-intro](https://github.com/mnhashemabadi/alwer-intro): آلور را برای خریداری گذاشتم که درخواست بنویسد و پیشنهاد قیمت‌ها را کنار هم ببیند؛ آگهی فروش هم روی همان بازار است.
- [azadchi-intro](https://github.com/mnhashemabadi/azadchi-intro): آزادچی را برای آگهی و جستجو در مناطق آزاد گذاشتم؛ گفتگو با طرف معامله داخل خود آزادچی می‌ماند.
- [afzi-intro](https://github.com/mnhashemabadi/afzi-intro): افزی را برای کوتاه کردن یک نشانی http یا https گذاشتم؛ باز کردن لینک همان صفحه را باز می‌کند.
- [alweryar-intro](https://github.com/mnhashemabadi/alweryar-intro): آلوریار را برای همکاری در بررسی آگهی و همکاری در فروش آلور گذاشتم.
- [alwerchi-intro](https://github.com/mnhashemabadi/alwerchi-intro): آلورچی را برای فروشگاهی گذاشتم که کالا و موجودی‌اش در بازار آلور دیده شود و خریدار در آلور بماند.

## Related

- [hamejoo-intro](https://github.com/mnhashemabadi/hamejoo-intro): I built Hamejoo for a visitor who types one phrase, sees listings in one place, and opens the original site for the listing itself.
- [alwer-intro](https://github.com/mnhashemabadi/alwer-intro): I built Alwer for a buyer who posts a request and compares price offers side by side; a sale listing sits on the same market.
- [azadchi-intro](https://github.com/mnhashemabadi/azadchi-intro): I built Azadchi for listings and search inside free zones; the conversation with the other party stays in Azadchi.
- [afzi-intro](https://github.com/mnhashemabadi/afzi-intro): I built Afzi to shorten an http or https address; opening the short link opens that same page.
- [alweryar-intro](https://github.com/mnhashemabadi/alweryar-intro): I built Alweryar for collaboration on listing review and on sales for Alwer.
- [alwerchi-intro](https://github.com/mnhashemabadi/alwerchi-intro): I built Alwerchi for a shop whose goods and stock show on the Alwer market while the buyer stays on Alwer.

یادداشت مهندسی کوتاه‌تر: [docs/engineering.md](docs/engineering.md)

Shorter engineering note: [docs/engineering.md](docs/engineering.md)
