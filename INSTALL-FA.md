# راهنمای نصب Nova Worker

نسخهٔ سورس: `v26.21.101-channel-v3`  
فایل اجرا: `nova-worker-channel-fixed.js`

این برنامه یک **Cloudflare Worker** است، نه یک سایت استاتیک. گیت‌هاب فقط محل نگه‌داشتن سورس است. ربات و پنل وب فقط روی Cloudflare اجرا می‌شوند. آپلود همین فایل روی GitHub Pages ربات را روشن نمی‌کند.

---

## ۱. چه چیزی لازم دارید

| مورد | الزامی؟ | توضیح |
|------|---------|--------|
| حساب Cloudflare | بله | پلن رایگان کافی است |
| توکن ربات تلگرام | بله | از [@BotFather](https://t.me/BotFather) |
| KV با نام `BOT_KV` | بله | حافظه، کش، کلید کاربر، پست کانال |
| Workers AI با نام `AI` | توصیه‌شده | مدل رایگان CF، ترجمه، TTS |
| دامنهٔ `workers.dev` | بله | خود Cloudflare می‌دهد |

رمز ادمین وب (`LOGIN_PASS`) و شناسهٔ عددی ادمین تلگرام (`TG_ADMIN_IDS`) را همان اول بگذارید، وگرنه پنل وب و هشدار ادمین کار نمی‌کنند.

---

## ۲. ساخت Worker

1. وارد [dash.cloudflare.com](https://dash.cloudflare.com) شوید.
2. **Workers & Pages** → **Create** → **Worker**.
3. یک نام انتخاب کنید (مثلاً `nova-ai`) و Worker خالی را بسازید.
4. **Edit code**.
5. تمام محتوای پیش‌فرض را پاک کنید.
6. کل محتوای فایل `nova-worker-channel-fixed.js` را بچسبانید. نام فایل داخل ادیتور مهم نیست؛ کد باید همان اسکریپت اصلی Worker باشد (یک `export default` دارد).
7. **Deploy**.

اگر حجم اسکریپت رد شد، از Wrangler استفاده کنید (بخش ۸). در حالت عادی حدود ۱.۲ مگابایت خام است و بعد از فشرده‌سازی معمولاً قبول می‌شود.

---

## ۳. KV

1. در همان Worker: **Settings** → **Bindings** → **Add** → **KV namespace**.
2. Variable name را دقیقاً بنویسید: `BOT_KV`
3. اگر namespace ندارید، یکی بسازید (مثلاً `nova-kv`) و به همین نام وصل کنید.
4. Save / Deploy.

بدون `BOT_KV` این‌ها از کار می‌افتند: حافظهٔ گفتگو، محدودیت نرخ، کش مدل، کلیدهای کاربران، پست‌های ذخیره‌شدهٔ کانال/گروه، یادآور.

---

## ۴. Workers AI (اختیاری ولی مفید)

1. **Settings** → **Bindings** → **Add** → **Workers AI**.
2. Variable name را دقیقاً بنویسید: `AI`
3. Deploy.

اگر این binding نباشد، چت فقط با کلیدهای خارجی (Groq، Gemini، OpenRouter و بقیه) کار می‌کند. ترجمهٔ داخلی CF و بعضی تست‌ها هم غیرفعال می‌مانند.

---

## ۵. متغیرهای محیطی

در **Settings** → **Variables and Secrets**:

### اجباری

| نام | نوع | مقدار |
|-----|-----|--------|
| `TELEGRAM_BOT_TOKEN` | Secret | توکن BotFather، مثل `123456:ABC...` |
| `LOGIN_PASS` | Secret | رمز ورود پنل وب |
| `TG_ADMIN_IDS` | Text | آیدی عددی تلگرام ادمین‌ها، با کاما. مثال: `123456789` |

آیدی عددی را از [@userinfobot](https://t.me/userinfobot) بگیرید.

### توصیه‌شده

| نام | کاربرد |
|-----|--------|
| `AUTH_SECRET` | امضای نشست. یک رشتهٔ طولانی تصادفی بگذارید. اگر خالی باشد از توکن ربات مشتق می‌شود. |
| `ALLOWED_ORIGIN` | اگر می‌خواهید CORS فقط به دامنهٔ خودتان باز باشد. مثال: `https://nova-ai.YOUR_SUBDOMAIN.workers.dev` |
| `SECRET_CHAT_PATH` | مسیر مخفی چت وب. پیش‌فرض: `/nova-chat-9x7k2` |

### کلید مدل (هرکدام که دارید)

این‌ها Secret هستند و با کاما می‌شود چند کلید گذاشت:

`GROQ_API_KEY` · `GEMINI_API_KEY` · `OPENAI_API_KEY` · `OPENROUTER_API_KEY` · `XAI_API_KEY` · `CEREBRAS_API_KEY` · `NVIDIA_API_KEY` · `MISTRAL_API_KEY` · `DEEPSEEK_API_KEY` · `HF_API_KEY` · `PERPLEXITY_API_KEY`

کلید را داخل کد نگذارید. فقط در Variables.

بعد از هر تغییر، یک بار **Deploy** کنید.

---

## ۶. وصل کردن تلگرام

آدرس Worker را بردارید. شکل آن معمولاً این است:

`https://NAME.YOUR_SUBDOMAIN.workers.dev`

در مرورگر این آدرس را باز کنید (فقط یک بار، بعد از ست شدن توکن):

`https://NAME.YOUR_SUBDOMAIN.workers.dev/set-webhook`

پاسخ باید JSON با `"ok": true` باشد. این کار webhook را روی `/webhook` می‌گذارد و منوی ربات را تنظیم می‌کند.

### تست

| آدرس | انتظار |
|------|--------|
| `/health` | `ok: true` و `bot: true` و `kv: true` |
| `/` | صفحهٔ معرفی |
| در تلگرام: `/start` | پاسخ ربات |

اگر `/health` مقدار `bot` یا `kv` را false نشان داد، binding یا Secret اشتباه است. نام‌ها باید دقیقاً `BOT_KV` و `TELEGRAM_BOT_TOKEN` باشند.

---

## ۷. خواندن کانال و گروه

| حالت | چه کار می‌کند |
|------|----------------|
| کانال عمومی | `/channel @name 5 پست` — از پیش‌نمایش `t.me/s/...` |
| یک پیام مشخص | لینک `t.me/name/123` را بفرستید |
| گروه یا کانال خصوصی | ربات باید **عضو** باشد (ترجیحاً ادمین). تلگرام تاریخچهٔ قبلی را نمی‌دهد؛ فقط پیام‌های بعد از عضویت ذخیره می‌شوند |
| فقط نام گروه، بدون عضویت و بدون لینک پیام | چیزی پیدا نمی‌شود. این محدودیت تلگرام است |

جملهٔ خیلی کلی مثل «بفرست» دیگر به‌تنهایی وارد خواندن کانال نمی‌شود. یا لینک / `@username` بفرستید، یا تا ۱۰ دقیقه بعد از ذخیرهٔ کانال بگویید «آخرین ۵ پست».

Cron (یادآور، خلاصهٔ صبح) اختیاری است. در تنظیمات Worker یک Trigger بگذارید، مثلاً `0 5 * * *` (ساعت ۰۵:۰۰ UTC، حدود ۰۸:۳۰ تهران).

---

## ۸. اگر با Wrangler نصب می‌کنید

در پوشهٔ پروژه:

```toml
name = "nova-ai"
main = "nova-worker-channel-fixed.js"
compatibility_date = "2026-10-01"

[[kv_namespaces]]
binding = "BOT_KV"
id = "شناسه-namespace"

[ai]
binding = "AI"

[triggers]
crons = ["0 5 * * *"]
```

Secretها را با CLI بگذارید، نه داخل فایل:

```bash
npx wrangler secret put TELEGRAM_BOT_TOKEN
npx wrangler secret put LOGIN_PASS
npx wrangler deploy
```

`TG_ADMIN_IDS` می‌تواند در `[vars]` باشد چون رمز نیست، ولی توکن و رمز عبور را هرگز در گیت commit نکنید.

---

## ۹. گذاشتن سورس روی گیت‌هاب

1. ریپوی جدید بسازید.
2. فقط این‌ها را آپلود کنید:
   - `nova-worker-channel-fixed.js`
   - همین `INSTALL-FA.md`
   - در صورت استفاده، `wrangler.toml` **بدون** هیچ توکن
3. یک `.gitignore` با این خط:

```gitignore
.dev.vars
.env
*.secret
```

GitHub Pages را برای اجرای ربات روشن نکنید. بعد از پوش کردن، باز هم Deploy واقعی همان دکمهٔ Deploy در Cloudflare یا `wrangler deploy` است.

---

## ۱۰. اگر بالا نیامد

| نشانه | کار |
|--------|-----|
| ربات ساکت است | `/set-webhook` را دوباره باز کنید. توکن را چک کنید |
| `/health` می‌گوید kv نیست | نام binding باید `BOT_KV` باشد، نه `KV` |
| پنل وب رمز را رد می‌کند | `LOGIN_PASS` را Secret کنید و دوباره Deploy |
| کانال عمومی خالی است | نام را با `@` دقیق بفرستید؛ کانال باید عمومی باشد |
| گروه خالی است | ربات عضو نیست، یا هنوز پیام جدیدی بعد از عضویت نیامده |
| خطای 1102 / CPU | درخواست خیلی سنگین است؛ تعداد پست را کم کنید (مثلاً ۵ تا) |
| Too many subrequests | نسخهٔ v3 پروب را کم کرده؛ اگر باز شد، فقط لینک یک پیام بفرستید نه اسکن گروه |

لاگ لحظه‌ای: Cloudflare → Worker → **Observability** یا **Logs** → Live.

---

## حداقل پیکربندی که باید سبز باشد

- فایل: `nova-worker-channel-fixed.js`
- Binding: `BOT_KV`
- Secret: `TELEGRAM_BOT_TOKEN`
- Secret: `LOGIN_PASS`
- Text: `TG_ADMIN_IDS`
- یک بار باز کردن `/set-webhook`
- `/health` با `ok`، `bot: true`، `kv: true`
