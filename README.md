<h1 align="center">BLACKFURY Exact v1.0.2</h1>
<p align="center"><strong>Qwen3.8-27B NVFP4 with DFlash2 on one RTX 5090</strong><br>Native TensorFold CUDA inference for Windows · 131,072-token context · FP8 KV cache</p>

<p align="center">
  <a href="https://github.com/Maharajahu/qwen3.8-27b-nvfp4-blackfury-exact-tensorfold/releases/tag/v1.0.2"><img src="https://img.shields.io/badge/Download-v1.0.2-77e7f4?style=for-the-badge" alt="Download BLACKFURY Exact v1.0.2"></a>
  <a href="#measurements"><img src="https://img.shields.io/badge/View-Measurements-30363d?style=for-the-badge" alt="View measurements"></a>
  <a href="README-v1.0.1.md"><img src="https://img.shields.io/badge/Previous-v1.0.1-30363d?style=for-the-badge" alt="Previous release and recordings"></a>
  <a href="https://x.com/ToolBraidComp"><img src="https://img.shields.io/badge/Follow-%40ToolBraidComp-111111?style=for-the-badge&amp;logo=x&amp;logoColor=white" alt="Follow ToolBraid on X"></a>
  <a href="https://buymeacoffee.com/dumitrescup"><img src="https://img.shields.io/badge/Buy_me_a_coffee-FFDD00?style=for-the-badge&amp;logo=buymeacoffee&amp;logoColor=000000" alt="Support the project"></a>
</p>

BLACKFURY runs a local OpenAI-compatible server for coding agents. During a 14-minute Oh My Pi session, Qwen built an offline viewer that compares NVIDIA Nsight traces. The 32 requests that wrote or edited code averaged **290.20 tok/s decode** and **2,725.61 tok/s fresh prefill**. The session ran with low reasoning effort, a four-bit DFlash2 drafter and a single stream.

## Compared with v1.0.1

| Generated output | v1.0.1 | v1.0.2 | Change |
|---|---:|---:|---:|
| Prose | 214.25 tok/s | 224.73 tok/s | +4.9% |
| Source code | 343.23 tok/s | 359.81 tok/s | +4.8% |

Three prose prompts and three source-generation prompts were run twice per version on one RTX 5090. Each request had 1,992–2,001 fresh input tokens and generated 1,024 tokens, including reasoning. The four engine launches ran in A–B–B–A order, with the prompt order reversed on the second pass. All 12 cross-version pairs produced identical output token IDs. Fresh prefill on these short inputs averaged 3,825.06 tok/s in v1.0.1 and 3,823.01 tok/s in v1.0.2. [Matched workloads and timings](results/blackfury-v1.0.2/prose-code-comparison.json) · [Prompts](results/blackfury-v1.0.2/prose-code-prompts.json)

On the archived 129,024-token prompt, median decode after cold prefill rose from **305.32 to 333.30 tok/s (+9.2%)**, warm decode rose from **309.53 to 330.77 tok/s (+6.9%)**, and cold prefill took **48.48 s**, down from **50.37 s**. These were separate qualification runs using identical request contents and cache conditions. All samples produced the same 2,048 output token IDs. [Per-run timings and token hashes](results/blackfury-v1.0.2/long-context-comparison.json)

## Run on Windows

1. Download and extract [the Windows x64 SM120a runtime](https://github.com/Maharajahu/qwen3.8-27b-nvfp4-blackfury-exact-tensorfold/releases/download/v1.0.2/blackfury-exact-v1.0.2-windows-x64-sm120a.zip). Use Python 3.11+ and an RTX 5090 with 32 GB of VRAM and a compatible NVIDIA driver.
2. Obtain the target and drafter checkpoints listed in the [model manifest](models-manifest.json). If you already have this repository's model setup, reuse those directories.
3. From the extracted directory, start the server with the paths to your local checkpoints.

```powershell
python launch.py --model-path 'C:\models\Qwen3.8-27B-NVFP4' --drafter-path 'C:\models\Qwen3.8-27B-DFlash2'
```

Point your client at `http://127.0.0.1:18830/v1` and select `blackfury-exact-v1.0.2`. The model weights are distributed separately. Find the checksums and build inputs on the [release page](https://github.com/Maharajahu/qwen3.8-27b-nvfp4-blackfury-exact-tensorfold/releases/tag/v1.0.2).

## Measurements

| Workload | Decode (tok/s) | Fresh prefill (tok/s) |
|---|---:|---:|
| Fixed prompt, 129,024 input + 2,048 output, three cold requests | 333.33 | — |
| Coding requests with file writes or edits, 32 requests | 290.20 | 2,725.61 |
| Coding requests with at least 90,000 input tokens, 8 of the 32 | 277.22 | 2,699.75 |
| Complete agent session, 99 model requests | 274.38 | 2,701.71 |

The first agent prompt processed **109,211 fresh tokens in 39.27 s**. The largest input was 111,600 tokens. Across 99 requests the model generated 86,553 tokens and OMP compacted its context twice. An independent reviewer reported two issues during the session. Qwen fixed the clock alignment and round selection. The final viewer passed 27 tests and matched all 40 numeric reference values, including a held-out capture.

Decode uses output-token intervals and native decode time. Fresh prefill divides newly processed prompt tokens by native prefill time. Both rates exclude tool execution. [Coding measurements](results/blackfury-v1.0.2/omp-coding.json) · [Release engineering and qualification](docs/blackfury-v1.0.2.md)

## Previous releases and credits

[BLACKFURY Exact v1.0.1 with install guide, benchmarks and recordings](README-v1.0.1.md) · [BLACKFURY Exact](variants/blackfury-exact/README.md) · [Twelve-branch engine](variants/experimental12/README.md) · [Shared-context agent](variants/shared-context/demo/README.md)

Built on [TensorFold](https://github.com/ashhart/TensorFold) by Ash (@ashhart) and its contributors. Model and drafter attribution is in [CREDITS.md](CREDITS.md). Local changes are licensed under [Apache-2.0](LICENSE). [Upstream and third-party notices](licenses/TensorFold) apply to their respective components.
