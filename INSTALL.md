# TheTensorTune — Installation & Setup Guide

> Documented release: **TheTensorTune 1.0** (first public release) — single file, no build step, no external database.
> The step-by-step illustrated walkthrough lives inside the platform under the **Learn** tab.

---

## 1) Introduction

TheTensorTune is a **LoRA fine-tuning workbench** for language models that fits the entire workflow into a single Python file:

- Bilingual node-graph UI (full RTL Persian + English)
- Real LoRA / QLoRA training on 8 international model presets (0.5B to 7B) or any Hugging Face model
- Dataset quality reports, cost/time estimation across 25 GPUs, job queue, training resume from checkpoints
- Test chat, before/after comparison (Perplexity), **GGUF** export for llama.cpp / Ollama / LM Studio
- Webhooks signed with HMAC-SHA256 + a complete REST API for automation
- A built-in **Learn** section with a fully illustrated 12-lesson course

---

## 2) Prerequisites

| Requirement | Minimum | Recommended |
|---|---|---|
| Python | 3.10 | 3.12 |
| RAM | 4 GB (0.5B model on CPU) | 16+ GB, or a GPU with 8 GB+ VRAM |
| Disk | ~2 GB (dependencies + 0.5B model) | 20+ GB for larger models and GGUF |
| GPU | not required (CPU works) | CUDA for real speed |
| OS | Linux / macOS / Windows | Linux |

> **Real-world note:** the entire development and testing of this release was done on a 2-core / 4 GB RAM machine with no GPU — 0.5B training, comparison, and GGUF export all work on CPU.

---

## 3) Installing dependencies

```bash
python3 -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

pip install flask==3.1.3
pip install torch                 # CPU build: pip install torch --index-url https://download.pytorch.org/whl/cpu
pip install transformers==5.17.0
pip install peft==0.21.0
pip install accelerate==1.15.0
pip install safetensors==0.8.0
pip install "huggingface_hub>=0.30"   # 1.x versions are also supported
```

Exact tested versions in this environment: `torch 2.14.0+cpu · transformers 5.17.0 · peft 0.21.0 · accelerate 1.15.0 · safetensors 0.8.0 · huggingface_hub 1.9.2`

K-quants (such as `q4_k_m`) require `llama-quantize` during GGUF export — build llama.cpp or set `TT_LLAMA_CPP`. `f16` and `q8_0` need nothing extra.

---

## 4) Running

```bash
python TheTensorTune.py
```

Default: the UI opens at `http://127.0.0.1:<port>` (a free port is chosen automatically; pin it with `TT_PORT`). The `datasets/` and `runs/` folders and the `thetensortune.db` file are created next to the main file.

### Environment variables

| Variable | Default | Purpose |
|---|---|---|
| `TT_TOKEN` | *(empty = no auth)* | Service token. If set, every `/api/*` route requires it (`X-TT-Token` header or `Authorization: Bearer`). **Always set it on a shared network** — binding to a non-local address without a token prints a warning. |
| `TT_PORT` | free port | HTTP port |
| `TT_HOST` | `127.0.0.1` | Bind address. For LAN: `0.0.0.0` + definitely set `TT_TOKEN` |
| `TT_HEADLESS` | `0` | `1` = do not open a browser (servers/containers). `TT_NO_BROWSER` is the same |
| `TT_DEBUG` | `0` | `1` = more verbose logging |
| `TT_LLAMA_CPP` | *(empty)* | Path to llama-quantize for K-quants |
| `TT_WEBHOOK_ALLOW_PRIVATE` | `0` | `1` = allow webhooks to internal addresses (local automation) |
| `HF_TOKEN` | *(empty)* | Hugging Face token for gated models (also available in the model block) |

Secure network example:

```bash
TT_TOKEN="a-long-random-string" TT_HOST=0.0.0.0 TT_PORT=8653 TT_HEADLESS=1 \
python TheTensorTune.py
```

---

## 5) First launch

1. Open the address printed in the terminal in your browser.
2. If the server started with `TT_TOKEN`, click the **Service token** button in the top bar, paste the token, and save it (it stays in your browser).
3. Use the **FA/EN** key to switch the interface language (full right-to-left Persian is supported).
4. Click **Learn** in the top bar — 12 complete illustrated lessons, from uploading a dataset to webhooks.
5. The `/api/health` route is public (for load-balancer probes) and returns no sensitive data.

---

## 6) Quick start (5 minutes)

```bash
# 1) Run
TT_TOKEN=my-secret python TheTensorTune.py

# 2) Upload a dataset (or do it from the UI)
curl -X POST http://127.0.0.1:8653/api/dataset \
  -H "X-TT-Token: my-secret" -H "Content-Type: application/json" \
  -d '{"text": "{\"messages\":[{\"role\":\"user\",\"content\":\"hi\"},{\"role\":\"assistant\",\"content\":\"hello!\"}]}"}'
# -> {"id": "ab12cd34ef56", ...}

# 3) Start training
curl -X POST http://127.0.0.1:8653/api/run/start \
  -H "X-TT-Token: my-secret" -H "Content-Type: application/json" \
  -d '{"model":{"preset":"Qwen/Qwen2.5-0.5B-Instruct"},"dataset_id":"ab12cd34ef56","lora":{"r":16,"alpha":32},"params":{"lr":2e-4,"epochs":3,"bs":2,"gacc":1,"eval_pct":20}}'

# 4) Monitor
curl http://127.0.0.1:8653/api/run/status -H "X-TT-Token: my-secret"

# 5) Chat
curl -X POST http://127.0.0.1:8653/api/infer \
  -H "X-TT-Token: my-secret" -H "Content-Type: application/json" \
  -d '{"run_id":1,"messages":[{"role":"user","content":"hi"}],"max_new":32}'

# 6) GGUF export (progress: /api/task/status)
curl -X POST http://127.0.0.1:8653/api/gguf/export \
  -H "X-TT-Token: my-secret" -H "Content-Type: application/json" \
  -d '{"run_id":1,"quant":"q8_0"}'
```

The same flow, illustrated step by step, is inside the platform: **Learn → lessons 3 to 11**.

---

## 7) Complete REST API map

Authentication: all `/api/*` routes (except `/api/health`) are protected by `TT_TOKEN` — header `X-TT-Token: <token>` or `Authorization: Bearer <token>`.

**Public:** `GET /` (UI) · `GET /api/health` (health, public) · `GET /learn_img/<Lxx>` (lesson images, public)

**System & models:** `GET /api/system` · `GET /api/models` · `GET /api/models/search?q=` · `GET|POST /api/settings`

**Dataset:** `POST /api/dataset` (JSON `{text}` or multipart file) · `GET /api/dataset/info?id=` · `POST /api/dataset/analyze`

**Training:** `POST /api/run/start` · `POST /api/run/stop` · `POST /api/run/resume` · `GET /api/run/status` · `POST /api/estimate` · `POST /api/jobs` (async + webhook required) · `GET /api/jobs` · `GET /api/jobs/<id>` · `POST /api/jobs/<id>/cancel` · `GET /api/runs` · `POST /api/runs/clear`

**Inference & export:** `POST /api/infer` (auto/adapter/merged/base modes) · `POST /api/compare` · `POST /api/gguf/export` · `GET /api/task/status`

### Webhooks (async)

When a job finishes, a POST is sent to your endpoint with these headers:

```
Content-Type: application/json
X-TT-Signature: sha256=<hmac-sha256(body, TT_TOKEN)>
User-Agent: TheTensorTune/<version>
```

Verifying the signature at the destination (Python):

```python
import hmac, hashlib
expected = "sha256=" + hmac.new(TT_TOKEN.encode(), raw_body_bytes, hashlib.sha256).hexdigest()
if hmac.compare_digest(expected, request.headers["X-TT-Signature"]):
    payload = json.loads(raw_body_bytes)
```

> Private/loopback destinations are blocked; for fully local automation use `TT_WEBHOOK_ALLOW_PRIVATE=1`.

---

## 8) Maintenance

| Path | Role |
|---|---|
| `datasets/` | Uploaded datasets (one file per id) |
| `runs/run-<timestamp>/` | Each training run: `adapter/`, `merged/` (after export), `config.json` (no HF token), `model-*.gguf` |
| `thetensortune.db` | SQLite — run and job history (survives restarts) |
| `~/.cache/huggingface/` | Cache of downloaded models |

Backups = copying these folders. Disk cleanup: `runs/*/merged` is the biggest consumer (fp32 weights) — it can be deleted once you have the GGUF.

---

## 9) Troubleshooting

| Symptom | Fix |
|---|---|
| `401 unauthorized` in the UI | Token not set or wrong — use the Service token button in the top bar |
| `no training stack` in the status bar | `pip install torch transformers peft accelerate` |
| OOM in back-to-back trainings | This release frees memory between jobs; if it still runs short, lower `bs` and raise `gacc` |
| Empty Hub search | Network/filtering — a Mirror option (hf-mirror.com) is available in the model block |
| llama-quantize error for K-quant | Set `TT_LLAMA_CPP`; `f16`/`q8_0` do not need it |
| `[Errno 28] No space left on device` | Disk full — delete `runs/*/merged` and old GGUF files |
| Meaningless replies after training | The learning rate was too high (e.g., 1.0) — retrain with `2e-4` |

---

## 10) Security of this release

- Dataset id routes are locked to the `datasets/` folder (no path traversal)
- `out_dir` is only accepted inside `runs/`
- All client-side HTML rendering is escaped (no stored XSS)
- Webhooks: SSRF filtering + CRLF sanitization + HMAC-SHA256 signature
- Constant-time token comparison; tokens removed from query strings
- The HF token is never written to `config.json` on disk
- Origin checks on browser POSTs (anti-CSRF)
- Whitelist on GGUF export quant names
- File upload in the UI sends the token header

---

## 11) Version history

| Version | Changes |
|---|---|
| **1.0** | First public release — complete platform tested against 50 scenarios (30 + 20), 6/6 security regressions passed, Learn section with a bilingual illustrated 12-lesson course, this guide |

---

## 12) License

TheTensorTune is released under the **Apache License 2.0** (see the `LICENSE` file in this package).

In plain terms:

- Use, modification, distribution, and **commercial** use are permitted and free
- You must include a copy of the `LICENSE` with your distribution and mark modified files
- The name and brand "TheTensorTune" are not transferred under this license (Trademarks clause)
- Contributors grant an explicit **patent license** + automatic termination in case of patent litigation (Section 3)
- The code is provided "as is"; no warranty and no liability for damages (Sections 7 and 8)

Runtime dependencies are not bundled and remain under their own licenses: Flask and PyTorch (BSD-3-Clause) · transformers, peft, accelerate, safetensors, and huggingface_hub (Apache-2.0).

Base models fine-tuned with this tool — and the adapter/GGUF outputs produced — remain subject to the base model's own license, and compliance with it is the user's responsibility.
