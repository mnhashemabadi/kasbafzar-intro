# کسب‌افزار

کسب‌افزار را پیشخوان فروش ساختم، نه صفحهٔ گزارشی جدا. فروش و مشتری و هزینه باید همان‌جا ثبت شود که گزارش امروز خوانده می‌شود. اگر فاکتور عمومی باشد لینک دارد تا مشتری بی‌آنکه وارد پیشخوان شود آن را ببیند. پیشخوان وب است. ثبت از تلگرام و بله را روی صفحهٔ عمومی نوشتم چون صاحب فروشگاه همیشه پشت مرورگر نیست.

سایت: [kasbafzar.ir](https://kasbafzar.ir)

نسخهٔ فعلی اپ وب `0.28.0` است و همان رشته روی صفحهٔ عمومی دیده می‌شود.

## آنچه صفحهٔ عمومی می‌گوید

صفحهٔ عمومی، پیشخوان را برای فروشگاه و شرکت کوچک این‌طور توصیف می‌کند: فاکتور، هزینه، بدهی مشتری، گزارش امروز، لینک پرداخت، یادآوری بدهی، و ثبت از تلگرام یا بله. شروع را رایگان و بدون نصب نوشته است. دعوت همکار با نقش و چند شعبه در همان متن است. ویترین و محصول و موجودی هم در فهرست کارهای صفحه هست. صفحه برچسب ERP و CRM دارد و دستیار تصمیم را «مبتنی بر LLM» می‌نامد.

## یک مخزن، پیشخوان و ربات

ریشهٔ پروژه یک workspace با pnpm 9.15.0 و Turbo ^2.9.14 است، روی Node.js 20 به بالا و TypeScript ^5.9.3. Prisma را روی `6.19.3` پین کردم و همان نسخه را برای CLI و کلاینت در overrides گذاشتم تا طرح پایگاه و کلاینت از هم جدا نشوند.

پیشخوان و API هر دو یک اپ Next.js هستند: Next ^14.2.21، React ^18.3.1، TypeScript ^5.7.2، Zod ^4.4.3 و `ioredis` ^5.10.1. صفحه‌های پنل برای کار روزانه است: فاکتور و سازندهٔ فاکتور، مشتری و پروندهٔ هر مشتری، حسابداری، محصول، فروشگاه، ویترین و تنظیمات فاکتور. فاکتور عمومی و ویترین بیرون پنل هستند تا لینک مشتری به صفحهٔ مدیریت نخورد.

ربات پیام‌رسان فرآیند جدایی است. داده روی PostgreSQL است و از راه Prisma خوانده و نوشته می‌شود.

## ساخت فاکتور

بدنهٔ ساخت فاکتور را با Zod اعتبارسنجی می‌کنم: نام مشتری از ۱ تا ۱۰۰ نویسه، شمارهٔ موبایل ایرانی اختیاری، و ۱ تا ۵۰ سطر کالا که تعداد و قیمت هر سطر عدد صحیح مثبت است و نرخ مالیات اختیاری بین ۰ تا ۱۰۰. پیش از ساخت پیش‌نویس، دسترسی کاربر به همان فروشگاه و مجاز بودن فروشگاه به صدور فاکتور سنجیده می‌شود.

فاکتور عمومی یک شناسهٔ تصادفی یکتا می‌گیرد و لینک اشتراک از روی همان ساخته می‌شود. مشتری از روی فاکتور به‌روز می‌شود تا کسی آن را دوباره دستی وارد نکند. اگر فاکتور نهایی و عمومی باشد، ساخت PDF در صف می‌رود. هزینه مسیر جدای خودش را دارد و صفحهٔ هر مشتری خط زمانی خریدها را نشان می‌دهد.

## داده‌ای که پیشخوان به آن تکیه می‌کند

جمع خرید و تعداد فاکتور و زمان آخرین خرید روی خود مشتری نگه داشته می‌شود تا گزارش مشتری هر بار از روی همهٔ فاکتورها حساب نشود. آستانهٔ موجودی کم را روی خود ردیف کالا گذاشتم تا گزارش امروز برای هشدار به جدول جدا محتاج نباشد. دفتر تراکنش، هزینه، محصول، موجودی و درخواست پرداخت هر کدام مدل خودشان را دارند.

PDF با `pdfmake` ^0.2.15 ساخته می‌شود و کتابخانه فقط وقتی بارگذاری می‌شود که گزارشی ساخته شود. صف PDF را بیرون از درخواست وب گذاشتم تا ساخت فایل، صفحه را معطل نکند.

QR با `qrcode` ^1.5.4 ساخته می‌شود تا لینک فاکتور روی کاغذ هم خوانده شود.

ربات تلگرام را با grammY ^1.30.0 نوشتم. هر به‌روزرسانی ورودی به یک شکل داخلی واحد تبدیل می‌شود و منبعش همراهش می‌ماند. ثبت از بله هم پشتیبانی می‌شود.

## English

I built Kasbafzar as the sales desk, not as a separate report page. A sale, a customer, and an expense have to be recorded in the same place the day's report is read. If an invoice is public it carries a link so the customer can open it without entering the desk. The desk is the web. I wrote Telegram and Bale entry on the public page because the shop owner is not always in a browser.

Site: [kasbafzar.ir](https://kasbafzar.ir)

The current web app version is `0.28.0`, and the public page shows that same string.

### What the public page says

The page describes the desk for a shop and a small company: invoice, expense, customer debt, today's report, a payment link, a debt reminder, and entry from Telegram or Bale. It says the start is free and nothing is installed. Inviting a colleague into a role, and more than one branch, are in that same text. Storefront, product, and stock are in the page's list of work. The page labels the desk ERP and CRM and names a decision assistant as LLM-based.

### One repository, desk and bot

The root is a pnpm 9.15.0 workspace with Turbo ^2.9.14, on Node.js 20 or later and TypeScript ^5.9.3. I pinned Prisma at `6.19.3` and put the same version for the CLI and the client in overrides, so the schema and the client do not drift apart.

The desk and the API are one Next.js app: Next ^14.2.21, React ^18.3.1, TypeScript ^5.7.2, Zod ^4.4.3, and `ioredis` ^5.10.1. The panel pages are for the day's work: invoices and the invoice composer, customers and each customer's record, accounting, products, the store, the storefront, and invoice settings. The public invoice and the storefront sit outside the panel so a customer link does not open the management UI.

The messenger bot is a separate process. Data is on PostgreSQL, read and written through Prisma.

### Creating an invoice

I validate the invoice body with Zod: a customer name of 1 to 100 characters, an optional Iranian mobile number, and 1 to 50 lines where quantity and price are positive integers and the tax rate is optional, from 0 to 100. Before the draft is written, the user's access to that store and the store's right to issue invoices are checked.

A public invoice gets a random unique id, and the share links are built from it. The customer is updated from the invoice so nobody types it in again by hand. When an invoice is final and public, PDF generation is queued. Expenses have their own path, and each customer page shows a purchase timeline.

### The data the desk relies on

Total spent, invoice count, and last purchase time are kept on the customer, so a customer report is not recomputed from every invoice on each read. I put the low-stock threshold on the stock row itself, so today's report does not need a separate alert table. Ledger transactions, expenses, products, stock, and payment requests each have their own model.

PDF is built with `pdfmake` ^0.2.15, and the library is loaded only when a report is generated. I kept the PDF queue outside the web request, so building the file does not hold the page.

QR is built with `qrcode` ^1.5.4, so an invoice link can be read off paper.

I wrote the Telegram bot with grammY ^1.30.0. Each incoming update becomes one internal shape that keeps its source. Entry from Bale is supported too.

## پروژه‌های مرتبط

- [hamejoo-intro](https://github.com/mnhashemabadi/hamejoo-intro): همه‌جو را برای بازدیدکننده‌ای ساختم که با یک عبارت آگهی را یک‌جا ببیند و برای جزئیات به سایت منبع برود.
- [alwer-intro](https://github.com/mnhashemabadi/alwer-intro): آلور را برای خریداری ساختم که درخواست بنویسد و پیشنهاد قیمت‌ها را کنار هم ببیند؛ آگهی فروش هم روی همان بازار است.
- [azadchi-intro](https://github.com/mnhashemabadi/azadchi-intro): آزادچی را برای آگهی و جستجو در مناطق آزاد ساختم؛ گفتگو با طرف معامله داخل خود آزادچی می‌ماند.
- [afzi-intro](https://github.com/mnhashemabadi/afzi-intro): افزی را برای کوتاه کردن یک نشانی http یا https ساختم؛ باز کردن لینک کوتاه همان صفحه را باز می‌کند.
- [alweryar-intro](https://github.com/mnhashemabadi/alweryar-intro): آلوریار را برای همکاری در بررسی آگهی و همکاری در فروش آلور ساختم.
- [alwerchi-intro](https://github.com/mnhashemabadi/alwerchi-intro): آلورچی را برای فروشگاهی ساختم که کالا و موجودی‌اش در بازار آلور دیده شود و خریدار در آلور بماند.

## Related

- [hamejoo-intro](https://github.com/mnhashemabadi/hamejoo-intro): I built Hamejoo for a visitor who types one phrase, sees listings in one place, and opens the original site for the listing itself.
- [alwer-intro](https://github.com/mnhashemabadi/alwer-intro): I built Alwer for a buyer who posts a request and compares price offers side by side; a sale listing sits on the same market.
- [azadchi-intro](https://github.com/mnhashemabadi/azadchi-intro): I built Azadchi for listings and search inside free zones; the conversation with the other party stays in Azadchi.
- [afzi-intro](https://github.com/mnhashemabadi/afzi-intro): I built Afzi to shorten an http or https address; opening the short link opens that same page.
- [alweryar-intro](https://github.com/mnhashemabadi/alweryar-intro): I built Alweryar for collaboration on listing review and on sales for Alwer.
- [alwerchi-intro](https://github.com/mnhashemabadi/alwerchi-intro): I built Alwerchi for a shop whose goods and stock show on the Alwer market while the buyer stays on Alwer.

یادداشت مهندسی کوتاه‌تر: [docs/engineering.md](docs/engineering.md)

Shorter engineering note: [docs/engineering.md](docs/engineering.md)
