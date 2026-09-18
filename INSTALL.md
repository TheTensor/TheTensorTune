# TheTensorTune — راهنمای نصب و راه‌اندازی

> نسخه‌ی مستند: **TheTensorTune 1.0** (اولین انتشار عمومی) — تک‌فایل، بدون نیاز به build، بدون دیتابیس خارجی.
> آموزش گام‌به‌گامِ تصویریِ استفاده، داخل خود پلتفرم در بخش **Learn (آموزش)** قرار دارد.

---

## ۱) معرفی

TheTensorTune یک **ورک‌بنچ تنظیم دقیق (Fine-tuning) LoRA** برای مدل‌های زبانی است که کل چرخه‌ی کار را در یک فایل Python جمع می‌کند:

- رابط کاربری نود-گرافی دوزبانه (فارسی RTL کامل + انگلیسی)
- آموزش واقعی LoRA / QLoRA روی ۸ مدل بین‌المللی (۰٫۵B تا ۷B) یا هر مدل Hugging Face
- گزارش کیفیت دیتاست، تخمین هزینه/زمان روی ۲۵ GPU، صف job، ادامه‌ی آموزش از checkpoint
- چت آزمایشی، مقایسه‌ی قبل/بعد (Perplexity)، خروجی **GGUF** برای llama.cpp / Ollama / LM Studio
- وب‌هوک با امضای HMAC-SHA256 + REST API کامل برای اتوماسیون
- بخش **Learn** داخلی با آموزش تصویری ۱۲ درسی

---

## ۲) پیش‌نیازها

| نیاز | حداقل | توصیه |
|---|---|---|
| Python | 3.10 | 3.12 |
| RAM | ۴ گیگابایت (مدل ۰٫۵B روی CPU) | ۱۶+ گیگابایت، یا GPU با ۸GB+ VRAM |
| دیسک | ~۲ گیگابایت (وابستگی‌ها + مدل ۰٫۵B) | ۲۰+ گیگابایت برای مدل‌های بزرگتر و GGUF |
| GPU | لازم نیست (CPU کار می‌کند) | CUDA برای سرعت واقعی |
| سیستم‌عامل | لینوکس / macOS / ویندوز | لینوکس |

> **تجربه‌ی واقعی:** کل توسعه و تست این نسخه روی یک ماشین ۲ هسته‌ای / ۴ گیگابایت RAM بدون GPU انجام شده — آموزش ۰٫۵B، مقایسه و خروجی GGUF همگی روی CPU کار می‌کنند.

---

## ۳) نصب وابستگی‌ها

```bash
python3 -m venv .venv
source .venv/bin/activate        # ویندوز: .venv\Scripts\activate

pip install flask==3.1.3
pip install torch                 # نسخه CPU:  pip install torch --index-url https://download.pytorch.org/whl/cpu
pip install transformers==5.17.0
pip install peft==0.21.0
pip install accelerate==1.15.0
pip install safetensors==0.8.0
pip install "huggingface_hub>=0.30"   # نسخه‌های 1.x هم پشتیبانی می‌شوند
```

نسخه‌های تست‌شده‌ی دقیق این محیط: `torch 2.14.0+cpu · transformers 5.17.0 · peft 0.21.0 · accelerate 1.15.0 · safetensors 0.8.0 · huggingface_hub 1.9.2`

K-quants (مثل `q4_k_m`) هنگام خروجی GGUF به `llama-quantize` نیاز دارند — llama.cpp را build کنید یا `TT_LLAMA_CPP` را ست کنید. `f16` و `q8_0` نیازی ندارند.

---

## ۴) اجرا

```bash
python TheTensorTune.py
```

پیش‌فرض: رابط در `http://127.0.0.1:<port>` باز می‌شود (پورت آزاد خودکار؛ با `TT_PORT` ثابت کنید). پوشه‌های `datasets/` و `runs/` و فایل `thetensortune.db` کنار فایل ساخته می‌شوند.

### متغیرهای محیطی

| متغیر | پیش‌فرض | توضیح |
|---|---|---|
| `TT_TOKEN` | *(خالی = بدون احراز هویت)* | توکن سرویس. اگر ست شود همه‌ی `/api/*` نیازمند آن است (هدر `X-TT-Token` یا `Authorization: Bearer`). **در شبکه‌ی مشترک حتماً ست کنید** — bind غیرمحلی بدون توکن، هشدار چاپ می‌کند. |
| `TT_PORT` | پورت آزاد | پورت HTTP |
| `TT_HOST` | `127.0.0.1` | آدرس bind. برای LAN: `0.0.0.0` + حتماً `TT_TOKEN` |
| `TT_HEADLESS` | `0` | `1` = مرورگر باز نشود (سرور/کانتینر). `TT_NO_BROWSER` همان است |
| `TT_DEBUG` | `0` | `1` = لاگ جزئی‌تر |
| `TT_LLAMA_CPP` | *(خالی)* | مسیر llama-quantize برای K-quants |
| `TT_WEBHOOK_ALLOW_PRIVATE` | `0` | `1` = اجازه‌ی وب‌هوک به آدرس داخلی (اتوماسیون محلی) |
| `HF_TOKEN` | *(خالی)* | توکن Hugging Face برای مدل‌های gated (از بلوک مدل هم می‌شود) |

نمونه‌ی امن روی شبکه:

```bash
TT_TOKEN="یک-رشته‌ی-تصادفی-بلند" TT_HOST=0.0.0.0 TT_PORT=8653 TT_HEADLESS=1 \
python TheTensorTune.py
```

---

## ۵) اولین ورود

1. آدرس چاپ‌شده در ترمینال را در مرورگر باز کنید.
2. اگر سرور با `TT_TOKEN` بالا آمده، روی دکمه‌ی **Service token / توکن سرویس** در نوار بالا کلیک کنید، مقدار توکن را وارد و ذخیره کنید (در مرورگر شما ذخیره می‌ماند).
3. با کلید **FA/EN** زبان رابط را عوض کنید (فارسی راست‌به‌چپ کامل پشتیبانی می‌شود).
4. روی **آموزش (Learn)** در نوار بالا کلیک کنید — ۱۲ درس تصویری کامل، از آپلود دیتاست تا وب‌هوک.
5. مسیر `/api/health` عمومی است (برای probe لود‌بالانسر) و هیچ داده‌ی حساسی برنمی‌گرداند.

---

## ۶) شروع سریع (۵ دقیقه)

```bash
# 1) اجرا
TT_TOKEN=my-secret python TheTensorTune.py

# 2) آپلود دیتاست (یا از داخل UI)
curl -X POST http://127.0.0.1:8653/api/dataset \
  -H "X-TT-Token: my-secret" -H "Content-Type: application/json" \
  -d '{"text": "{\"messages\":[{\"role\":\"user\",\"content\":\"hi\"},{\"role\":\"assistant\",\"content\":\"hello!\"}]}"}'
# -> {"id": "ab12cd34ef56", ...}

# 3) شروع آموزش
curl -X POST http://127.0.0.1:8653/api/run/start \
  -H "X-TT-Token: my-secret" -H "Content-Type: application/json" \
  -d '{"model":{"preset":"Qwen/Qwen2.5-0.5B-Instruct"},"dataset_id":"ab12cd34ef56","lora":{"r":16,"alpha":32},"params":{"lr":2e-4,"epochs":3,"bs":2,"gacc":1,"eval_pct":20}}'

# 4) پایش
curl http://127.0.0.1:8653/api/run/status -H "X-TT-Token: my-secret"

# 5) چت
curl -X POST http://127.0.0.1:8653/api/infer \
  -H "X-TT-Token: my-secret" -H "Content-Type: application/json" \
  -d '{"run_id":1,"messages":[{"role":"user","content":"hi"}],"max_new":32}'

# 6) خروجی GGUF (پیشرفت: /api/task/status)
curl -X POST http://127.0.0.1:8653/api/gguf/export \
  -H "X-TT-Token: my-secret" -H "Content-Type: application/json" \
  -d '{"run_id":1,"quant":"q8_0"}'
```

همین جریان به‌صورت تصویری و گام‌به‌گام داخل پلتفرم: **آموزش ← درس‌های ۳ تا ۱۱**.

---

## ۷) نقشه‌ی کامل REST API

احراز هویت: همه‌ی مسیرهای `/api/*` (به‌جز `/api/health`) با `TT_TOKEN` محافظت می‌شوند — هدر `X-TT-Token: <token>` یا `Authorization: Bearer <token>`.

**عمومی:** `GET /` (رابط کاربری) · `GET /api/health` (سلامت، عمومی) · `GET /learn_img/<Lxx>` (تصاویر آموزشی، عمومی)

**سیستم و مدل‌ها:** `GET /api/system` · `GET /api/models` · `GET /api/models/search?q=` · `GET|POST /api/settings`

**دیتاست:** `POST /api/dataset` (JSON `{text}` یا multipart فایل) · `GET /api/dataset/info?id=` · `POST /api/dataset/analyze`

**آموزش:** `POST /api/run/start` · `POST /api/run/stop` · `POST /api/run/resume` · `GET /api/run/status` · `POST /api/estimate` · `POST /api/jobs` (آسنکرون + وب‌هوک الزامی) · `GET /api/jobs` · `GET /api/jobs/<id>` · `POST /api/jobs/<id>/cancel` · `GET /api/runs` · `POST /api/runs/clear`

**استنتاج و خروجی:** `POST /api/infer` (حالت‌های auto/adapter/merged/base) · `POST /api/compare` · `POST /api/gguf/export` · `GET /api/task/status`

### وب‌هوک (آسنکرون)

پایان job به آدرس شما POST می‌شود با هدرها:

```
Content-Type: application/json
X-TT-Signature: sha256=<hmac-sha256(body, TT_TOKEN)>
User-Agent: TheTensorTune/<version>
```

تأیید امضا در مقصد (Python):

```python
import hmac, hashlib
expected = "sha256=" + hmac.new(TT_TOKEN.encode(), raw_body_bytes, hashlib.sha256).hexdigest()
if hmac.compare_digest(expected, request.headers["X-TT-Signature"]):
    payload = json.loads(raw_body_bytes)
```

> مقاصد خصوصی/loopback بلاک می‌شوند؛ برای اتوماسیون کاملاً محلی `TT_WEBHOOK_ALLOW_PRIVATE=1`.

---

## ۸) نگهداری

| مسیر | نقش |
|---|---|
| `datasets/` | دیتاست‌های آپلود‌شده (هر فایل = یک id) |
| `runs/run-<timestamp>/` | هر آموزش: `adapter/`، `merged/` (بعد از export)، `config.json` (بدون توکن HF)، `model-*.gguf` |
| `thetensortune.db` | SQLite — تاریخچه‌ی runها و jobها (بعد از ری‌استارت باقی می‌ماند) |
| `~/.cache/huggingface/` | کش مدل‌های دانلود‌شده |

بکاپ = کپی همین پوشه‌ها. پاک‌سازی دیسک: `runs/*/merged` بزرگترین مصرف‌کننده است (وزن fp32) — پس از گرفتن GGUF قابل حذف است.

---

## ۹) رفع اشکال

| نشانه | راه‌حل |
|---|---|
| `401 unauthorized` در UI | توکن ست نشده/غلط — دکمه‌ی توکن سرویس در نوار بالا |
| `no training stack` در نوار پایین | `pip install torch transformers peft accelerate` |
| OOM در آموزش‌های پشت‌سرهم | این نسخه حافظه را بین jobها آزاد می‌کند؛ اگر باز کم آورد، `bs` کم و `gacc` زیاد |
| جستجوی Hub خالی | شبکه/فیلتر — گزینه‌ی Mirror (hf-mirror.com) در بلوک مدل |
| خطای llama-quantize برای K-quant | `TT_LLAMA_CPP` را ست کنید؛ `f16`/`q8_0` نیاز ندارند |
| `[Errno 28] No space left on device` | دیسک پر — `runs/*/merged` و GGUFهای قدیمی را پاک کنید |
| پاسخ‌های بی‌معنا بعد از آموزش | lr زیادی بوده (مثل 1.0) — با `2e-4` دوباره آموزش بدهید |

---

## ۱۰) امنیت این نسخه

- مسیرهای dataset id به پوشه‌ی `datasets/` قفل شده‌اند (بدون path traversal)
- `out_dir` فقط داخل `runs/` پذیرفته می‌شود
- همه‌ی رندرهای HTML سمت کلاینت escape می‌شوند (بدون XSS ذخیره‌ای)
- وب‌هوک: فیلتر SSRF + پاک‌سازی CRLF + امضای HMAC-SHA256
- مقایسه‌ی توکن constant-time؛ توکن در query string حذف شده
- توکن HF هرگز در `config.json` روی دیسک نوشته نمی‌شود
- بررسی Origin روی POSTهای مرورگری (ضد CSRF)
- whitelist روی نام quant خروجی GGUF
- آپلود فایل در UI هدر توکن را ارسال می‌کند

---

## ۱۱) تاریخچه‌ی نسخه

| نسخه | تغییرات |
|---|---|
| **1.0** | اولین انتشار عمومی — پلتفرم کامل با تست ۵۰ سناریو (۳۰ + ۲۰)، رگرسیون امنیتی ۶/۶، بخش Learn با آموزش تصویری ۱۲ درسی دوزبانه، این راهنما |

---

## ۱۲) لایسنس

TheTensorTune تحت **Apache License 2.0** منتشر می‌شود (فایل `LICENSE` در همین بسته).

به زبان ساده یعنی:

- استفاده، تغییر، توزیع و استفاده‌ی **تجاری** مجاز و رایگان است
- باید نسخه‌ای از `LICENSE` را همراه توزیع خود ببرید و فایل‌های تغییریافته را علامت‌گذاری کنید
- نام و برند «TheTensorTune» تحت این لایسنس منتقل نمی‌شود (بند Trademarks)
- اعطای صریح **حق ثبت اختراع** از طرف مشارکت‌کنندگان + خاتمه خودکار آن در صورت دعوای اختراعی (بند ۳)
- کد «همان‌طور که هست» ارائه می‌شود؛ بدون ضمانت و بدون مسئولیت خسارت (بندهای ۷ و ۸)

وابستگی‌های اجرایی همراه بسته نمی‌آیند و هر کدام تحت لایسنس خودشان باقی می‌مانند: Flask و PyTorch (BSD-3-Clause) · transformers، peft، accelerate، safetensors و huggingface_hub (Apache-2.0).

مدل‌های پایه‌ای که با این ابزار تنظیم دقیق می‌شوند و خروجی‌های adapter/GGUF که تولید می‌شود، تابع لایسنس خود مدل پایه هستند و رعایت آن با کاربر است.
