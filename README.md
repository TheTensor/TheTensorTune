<div align="center">

# TheTensorTune

**Fine-tune a language model, start to finish, in one file.**

A free, open-source LoRA/QLoRA workbench — single file, no build step, no external database.

[![License: Apache-2.0](https://img.shields.io/badge/license-Apache--2.0-3DA639.svg)](LICENSE)
[![Release](https://img.shields.io/badge/release-v1.0-0969DA.svg)](https://github.com/TheTensor/TheTensorTune/releases/tag/v1.0)
[![Python](https://img.shields.io/badge/python-3.10%2B-3776AB.svg)](https://www.python.org)
[![Docs](https://img.shields.io/badge/docs-thetensor.site-8A2BE2.svg)](https://thetensor.site)

</div>

## Overview

TheTensorTune packs the complete fine-tuning lifecycle into a single Python file — roughly 4,800 lines with zero build tooling: dataset ingestion and quality analysis, LoRA/QLoRA training with checkpoint resume, interactive chat, before/after evaluation, and quantized GGUF export for llama.cpp, Ollama, and LM Studio.

Everything runs locally. A bilingual (English / Persian RTL) node-graph UI, a complete REST API, HMAC-signed webhooks, and a built-in 12-lesson illustrated course are included, so a first-time user can go from a raw text file to a quantized GGUF model without touching a config file.

The entire v1.0 release was developed and verified on a 2-core / 4 GB machine with no GPU. The 0.5B preset trains, evaluates, and exports entirely on CPU — no cloud account, no GPU rental, and no data leaving your machine required.

## Features

| Area | What you get |
|---|---|
| **Training** | Real LoRA / QLoRA on 8 model presets (0.5B → 7B) or any Hugging Face model; resume from checkpoint; automatic memory release between jobs |
| **Datasets** | Upload via UI or API; pre-flight quality analysis for duplicates, length outliers, and format issues — before you spend GPU-hours |
| **Planning** | Wall-clock time and cost estimation across 25 GPU profiles |
| **Job queue** | Asynchronous jobs with cancellation, SQLite-persisted history, and resume after restart |
| **Evaluation** | In-browser chat plus before/after Perplexity comparison |
| **Export** | GGUF in `f16`, `q8_0`, and K-quants (via `llama-quantize`) — ready for llama.cpp, Ollama, and LM Studio |
| **Automation** | Complete REST API and HMAC-SHA256-signed webhooks on job completion |
| **Learning** | Built-in 12-lesson illustrated course, from first dataset to first webhook |

## Quick start

```bash
# 1) Install dependencies
python3 -m venv .venv
source .venv/bin/activate           # Windows: .venv\Scripts\activate
pip install -r requirements.txt     # CPU-only torch: see INSTALL.md

# 2) Launch
python TheTensorTune.py
# → opens at http://127.0.0.1:<port>; a free port is chosen automatically

# 3) Open the Learn tab and follow the 12-lesson course.
```

Prefer the terminal? The same flow through the REST API:

```bash
TT_TOKEN=my-secret python TheTensorTune.py   # add TT_HEADLESS=1 for servers

# Upload a dataset
curl -X POST http://127.0.0.1:8653/api/dataset \
  -H "X-TT-Token: my-secret" -H "Content-Type: application/json" \
  -d '{"text": "{\"messages\":[{\"role\":\"user\",\"content\":\"hi\"},{\"role\":\"assistant\",\"content\":\"hello!\"}]}"}'

# Start a training run
curl -X POST http://127.0.0.1:8653/api/run/start \
  -H "X-TT-Token: my-secret" -H "Content-Type: application/json" \
  -d '{"model":{"preset":"Qwen/Qwen2.5-0.5B-Instruct"},"dataset_id":"<id>","lora":{"r":16,"alpha":32},"params":{"lr":2e-4,"epochs":3,"bs":2,"gacc":1,"eval_pct":20}}'

# Chat with the result, then export GGUF
curl -X POST http://127.0.0.1:8653/api/infer \
  -H "X-TT-Token: my-secret" -H "Content-Type: application/json" \
  -d '{"run_id":1,"messages":[{"role":"user","content":"hi"}],"max_new":32}'
curl -X POST http://127.0.0.1:8653/api/gguf/export \
  -H "X-TT-Token: my-secret" -H "Content-Type: application/json" \
  -d '{"run_id":1,"quant":"q8_0"}'
```

## Configuration

| Variable | Default | Purpose |
|---|---|---|
| `TT_TOKEN` | *(empty = no auth)* | Service token — **set this on any shared network**. Protects all `/api/*` endpoints. |
| `TT_HOST` | `127.0.0.1` | Bind address. For LAN access: `0.0.0.0` **plus** `TT_TOKEN`. |
| `TT_PORT` | free port | Fixed HTTP port. |
| `TT_HEADLESS` | `0` | `1` = do not open a browser (servers and containers). |
| `TT_LLAMA_CPP` | *(empty)* | Path to `llama-quantize` for K-quants (`f16`/`q8_0` need nothing). |
| `HF_TOKEN` | *(empty)* | Hugging Face token for gated models. |

## Documentation

The full guide — complete REST API reference, webhook signature verification, maintenance and backup notes, and a troubleshooting table — lives in **[INSTALL.md](INSTALL.md)**. Additional material is available at [thetensor.site](https://thetensor.site).

## Requirements

| Need | Minimum | Recommended |
|---|---|---|
| Python | 3.10 | 3.12 |
| RAM | 4 GB (0.5B model on CPU) | 16 GB+, or a GPU with 8 GB+ VRAM |
| Disk | ~2 GB | 20 GB+ for larger models and GGUF output |
| GPU | not required | CUDA for full-speed training |

## Security

TheTensorTune is built to be safe to expose on a trusted network: dataset routes are path-traversal-locked, webhooks are SSRF-filtered and CRLF-sanitized with HMAC-SHA256 signatures, token comparison is constant-time, browser POSTs carry Origin checks (CSRF), all client-side rendering is HTML-escaped, HF tokens are never written to disk, and GGUF quant names are whitelist-validated. The complete checklist is in [INSTALL.md](INSTALL.md).

## Project layout

| Path | Role |
|---|---|
| `TheTensorTune.py` | The entire application — server, UI, trainer, exporter |
| `datasets/` | Uploaded datasets (created at runtime) |
| `runs/` | One folder per training run: adapter, merged weights, config, GGUF |
| `thetensortune.db` | SQLite — run and job history |

## Changelog

- **v1.0** — First public release: complete platform validated against a 50-scenario test suite (30 functional + 20 security), 6/6 security regression checks, bilingual 12-lesson Learn course.

## License

Released under the [Apache License 2.0](LICENSE). Base models you fine-tune — and the adapters and GGUF files you produce — remain subject to their own licenses.
