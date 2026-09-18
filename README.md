<div align="center">

# TheTensorTune

**Fine-tune a language model, start to finish, in one file.**

A free, open-source LoRA/QLoRA workbench — single-file, no build step, no external database.

`v1.0` · Apache-2.0 · Python 3.10+ · CPU-friendly

[Website](https://thetensor.site) · [Install Guide](INSTALL.md) · [Releases](https://github.com/TheTensor/TheTensorTune/releases)

</div>

---

## What is it?

TheTensorTune packs the entire fine-tuning lifecycle into **one Python file** (~4,800 lines, zero build tooling): dataset ingestion, quality checks, LoRA/QLoRA training, interactive chat, before/after evaluation, and GGUF export for llama.cpp, Ollama, and LM Studio. It ships with a bilingual (English / Persian RTL) node-graph UI, a full REST API, signed webhooks, and a built-in 12-lesson illustrated course — so a complete beginner can go from a raw text dataset to a quantized GGUF model without touching a config file.

It was designed and tested on modest hardware: the whole v1.0 release was developed and verified on a 2-core / 4 GB RAM machine with no GPU. The 0.5B preset trains, evaluates, and exports entirely on CPU.

## Features

- **Node-graph UI, bilingual** — full English + Persian (RTL) interface; switch with one click.
- **Real LoRA / QLoRA training** — 8 international model presets (0.5B → 7B) or any Hugging Face model; resume from checkpoint.
- **Dataset quality checks** — analyze a dataset before burning GPU-hours; get warnings on duplicates, length outliers, and format problems.
- **Cost & time estimation** — estimate wall-clock time and cost across 25 GPU profiles before you start.
- **Job queue** — asynchronous jobs with cancellation, history in SQLite, and resume-after-restart.
- **Chat & evaluation** — test your model in the browser; before/after comparison with Perplexity.
- **GGUF export** — `f16`, `q8_0`, and K-quants (via `llama-quantize`) ready for llama.cpp / Ollama / LM Studio.
- **Automation-ready** — complete REST API plus HMAC-SHA256-signed webhooks for job completion.
- **Learn section** — an illustrated, 12-lesson course built into the platform, from first dataset to webhook.

## Quick start (5 minutes)

```bash
# 1) Dependencies
python3 -m venv .venv
source .venv/bin/activate           # Windows: .venv\Scripts\activate
pip install -r requirements.txt     # CPU-only torch: see INSTALL.md

# 2) Run
python TheTensorTune.py
# → opens at http://127.0.0.1:<port> (a free port is chosen automatically)

# 3) Open the Learn tab in the UI and follow lessons 1–12.
```

Prefer the terminal? The same flow via the REST API:

```bash
TT_TOKEN=my-secret python TheTensorTune.py   # headless: add TT_HEADLESS=1

# Upload a dataset
curl -X POST http://127.0.0.1:8653/api/dataset \
  -H "X-TT-Token: my-secret" -H "Content-Type: application/json" \
  -d '{"text": "{\"messages\":[{\"role\":\"user\",\"content\":\"hi\"},{\"role\":\"assistant\",\"content\":\"hello!\"}]}"}'

# Start training
curl -X POST http://127.0.0.1:8653/api/run/start \
  -H "X-TT-Token: my-secret" -H "Content-Type: application/json" \
  -d '{"model":{"preset":"Qwen/Qwen2.5-0.5B-Instruct"},"dataset_id":"<id>","lora":{"r":16,"alpha":32},"params":{"lr":2e-4,"epochs":3,"bs":2,"gacc":1,"eval_pct":20}}'

# Chat with the result, then export GGUF
curl -X POST http://127.0.0.1:8653/api/infer   -H "X-TT-Token: my-secret" -H "Content-Type: application/json" -d '{"run_id":1,"messages":[{"role":"user","content":"hi"}],"max_new":32}'
curl -X POST http://127.0.0.1:8653/api/gguf/export -H "X-TT-Token: my-secret" -H "Content-Type: application/json" -d '{"run_id":1,"quant":"q8_0"}'
```

## Configuration (environment variables)

| Variable | Default | Purpose |
|---|---|---|
| `TT_TOKEN` | *(empty = no auth)* | Service token — **set this on any shared network**. Protects all `/api/*`. |
| `TT_HOST` | `127.0.0.1` | Bind address. For LAN: `0.0.0.0` **plus** `TT_TOKEN`. |
| `TT_PORT` | free port | Fixed HTTP port. |
| `TT_HEADLESS` | `0` | `1` = don't open a browser (servers/containers). |
| `TT_LLAMA_CPP` | *(empty)* | Path to `llama-quantize` for K-quants (`f16`/`q8_0` need nothing). |
| `HF_TOKEN` | *(empty)* | Hugging Face token for gated models. |

The full reference — including the complete REST API map, webhook signature verification, maintenance notes, and a troubleshooting table — lives in [INSTALL.md](INSTALL.md).

## Security posture

Path-traversal-locked dataset routes · SSRF-filtered + CRLF-sanitized webhooks with HMAC-SHA256 signatures · constant-time token comparison · Origin checks on browser POSTs (CSRF) · HTML-escaped client rendering · HF tokens never written to disk · quant-name whitelist on GGUF export.

## Requirements

| Need | Minimum | Recommended |
|---|---|---|
| Python | 3.10 | 3.12 |
| RAM | 4 GB (0.5B model on CPU) | 16 GB+, or GPU with 8 GB+ VRAM |
| Disk | ~2 GB | 20 GB+ for larger models and GGUF |
| GPU | not required | CUDA for real speed |

## License

Released under the [Apache License 2.0](LICENSE). Base models you fine-tune — and the adapters/GGUF files you produce — remain subject to their own licenses.

---

<div dir="rtl">

## فارسی — خلاصه

**TheTensorTune** یک ورک‌بنچ متن‌باز و رایگان تنظیم دقیق (Fine-tuning) مدل‌های زبانی است که کل چرخه‌ی کار را در **یک فایل پایتون** جمع می‌کند: رابط کاربری نود-گرافی دوزبانه (فارسی راست‌به‌چپ کامل + انگلیسی)، آموزش واقعی LoRA/QLoRA روی ۸ مدل آماده (۰٫۵B تا ۷B) یا هر مدل Hugging Face، گزارش کیفیت دیتاست، تخمین هزینه/زمان روی ۲۵ GPU، صف job با ادامه‌ی آموزش از checkpoint، چت آزمایشی و مقایسه‌ی قبل/بعد (Perplexity)، خروجی **GGUF** برای llama.cpp / Ollama / LM Studio، وب‌هوک با امضای HMAC-SHA256، REST API کامل، و بخش **آموزش (Learn)** با ۱۲ درس تصویری.

**شروع سریع:** `pip install -r requirements.txt` و سپس `python TheTensorTune.py` — رابط در مرورگر باز می‌شود. راهنمای کامل نصب، نگهداری، API و رفع اشکال در [INSTALL.md](INSTALL.md) است.

</div>
