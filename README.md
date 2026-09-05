# local_LLM_workspace
Running a 4B LLM fully offline on 16 GB RAM using Ollama + Docker + Open WebUI and CPU-only which  is benchmarked across 7 real study tasks.


## Why I Built This

My internet in the evenings and afternoons is around 200 KB/s hich is  too slow and unstable to rely on cloud AI reliably.

I wanted a setup where I could ask questions about my coursework, debug code, and study AI concepts without needing a working connection.

So I started asking: Which model can actually be useful for this? Not just run eating my ram. Can I actually study with it? Debug with it? Understand complex concepts such as transformers with it?

Instead of assuming yes or no, I measured it started with qwen3:8b but it took so much time to download then I switched to qwen3.5:4b and I don't regret it.

---

## What I Set Up

```
Windows 11
│
├── Ollama (Port 11434) ──── Qwen3.5:4B (CPU-only, 0% GPU)
│
└── Docker Desktop + WSL2
         └── Open WebUI (Port 3000)
                  └── connects to Ollama on host
```

Inference path: `You → Open WebUI → Ollama → Qwen3.5:4B → CPU → Response`

No internet required after this setup, without any API key and no data leaving the machine.

---

## Hardware

| | |
|---|---|
| CPU | AMD Ryzen 7 7735HS (8 cores / 16 logical processors) |
| RAM | 16 GB (usable ~14.8 GB) |
| GPU | Integrated AMD Radeon — **0% used during inference** |
| OS | Windows 11 + Docker Desktop + WSL2 |

---

## Model

**Qwen3.5:4B** via Ollama

| | |
|---|---|
| Disk | ~3.4 GB |
| RAM when loaded (idle) | ~12.8 GB (~86%) |
| RAM during active inference | ~13–14 GB (85–88%) |
| Inference device | CPU only |

---

## The Most Interesting Finding

I expected thinking mode ON to be slower because each token would be slower but I was wrong.

| Mode | Token speed |
|------|------------|
| Thinking OFF | ~3.5 tok/s |
| Thinking ON | ~3.3 tok/s |

The raw generation rate barely changed.

But thinking ON took **~100 minutes** for a task that thinking OFF completed in under 2 minutes.

The reason: thinking ON generated ~2,004 reasoning tokens — internal chain-of-thought before the answer. At 3.3 tok/s on CPU, 2,000 tokens is a long time.

**The bottleneck wasn't speed. It was how many tokens the model decided to generate.**

On this hardware, for any practical study task: thinking OFF is the right default.

---

## Benchmark: 7 Practical Tasks, Fully Offline

Wi-Fi disconnected. `Test-NetConnection 1.1.1.1` returned `False`. Thinking OFF.

| # | What I tested | Time |
|---|--------------|------|
| 1 | Python dictionaries — concept explanation | 1m 56s |
| 2 | Write `find_duplicates()` function | 1m 19s |
| 3 | Find and fix a loop mutation bug | 1m 38s |
| 4 | Explain self-attention: Q, K, V, softmax, weighted sum | 2m 31s |
| 5 | Tensor shape (32, 128, 768) — dimensions and linear layer math | 1m 40s |
| 6 | Summarize a Transformer passage in 6 bullet points | 1m 08s |
| 7 | Teach softmax with intuition, formula, example, quiz | 4m 02s |

**Total: 14m 14s. Average: ~2m 02s per task.**

These are real workloads from my AI engineering study sessions not artificial prompts chosen to make the numbers look good.

---

## Resource Usage

**During active inference:**
- CPU: ~72–75%
- RAM: ~85–88%
- GPU: ~2–3% (integrated, not doing compute)

**After generation finishes:**
- CPU drops back to ~2%
- RAM stays high — model stays loaded

You can't comfortably run heavy applications while it's generating. Browser + IDE + Ollama works. Anything heavier starts competing for RAM.

---

## What Failed (I'm Keeping This In)

**Terminal math notation** — weighted sum formulas, LaTeX-style equations don't render properly in a PowerShell terminal. Use Open WebUI for anything math-heavy.

**Truncated output** — the final test (RAM vs. VRAM, 5 points requested) stopped after 3. Small models on CPU will sometimes cut off long structured responses.

**Factual errors in generated answers:**
- Dictionary explanation oversimplified Python's hash collision handling (not accurate for modern CPython)
- Softmax explanation had an incorrect statement about gradient behavior

I'm not hiding these. This benchmark measures **whether the setup works for practical study**, not whether every answer is correct. Local LLMs still need verification — that's part of what I learned by actually using it.

---

## How to Reproduce This

### What you need
- Windows 11 with WSL2 enabled
- Docker Desktop (AMD64 version)
- At least 16 GB RAM

### Step 1 — Install Ollama

Download from [ollama.com](https://ollama.com) and install.

```powershell
# Pull the model (~3.4 GB — do this when you have decent internet)
ollama pull qwen3.5:4b

# Verify it works
ollama run qwen3.5:4b
# Type /bye to exit
```

### Step 2 — Install Docker Desktop

Download the AMD64 version from [docker.com](https://docker.com). Enable WSL2 integration during setup.

### Step 3 — Run Open WebUI

```powershell
docker run -d -p 3000:8080 `
  --add-host=host.docker.internal:host-gateway `
  -v open-webui:/app/backend/data `
  --name open-webui `
  --restart always `
  ghcr.io/open-webui/open-webui:main
```

Open `http://localhost:3000`. Create a local account. Qwen3.5:4B appears automatically.

### Step 4 — Test offline

```powershell
# Disconnect Wi-Fi, then:
Test-NetConnection 1.1.1.1 -InformationLevel Quiet
# Returns: False

# Ollama still works
ollama run qwen3.5:4b
```

### Controlling thinking mode

```
# Inside ollama run qwen3.5:4b

/set nothink    # faster — use this for study
/set think      # extended reasoning — much slower on CPU
```

---

## What I Actually Learned

The setup works ~3.5 tok/s on CPU, under 2 minutes average per study task and that's usable.

The interesting finding wasn't the benchmark numbers. It was understanding *why* thinking mode was slow. I assumed token rate was the variable. It wasn't. That kind of misunderstanding only shows up when you measure.

The limitations matter too. Terminal is the wrong interface for math. The model truncates occasionally. Factual errors appear. These aren't reasons to not use local LLMs — they're things to know going in.

The original problem is solved: I can study and debug without needing internet.

---



---

*Started because my evening internet was 200 KB/s and I needed to study anyway.*
