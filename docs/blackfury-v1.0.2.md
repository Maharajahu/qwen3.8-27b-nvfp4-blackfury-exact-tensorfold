# BLACKFURY Exact v1.0.2

The release profile pairs QKV and Z FP8 projections in a single contiguous CUDA launch, retaining each projection's split-K accumulation sequence. The SM120a build also uses packed conversion for the FP4 gate/up path. The Windows executable used for the measurements has SHA-256 `0742736bdb430e5a4d25cc300beee759a79278b62d406a2fc9331972ea98a5bf`.

## Fixed-prompt qualification

Windows, one 32 GB RTX 5090, native CUDA, Qwen3.8-27B NVFP4, FP8 KV, four-bit DFlash2, greedy decoding, low reasoning, 2,048 reasoning-token budget, context 131,072. The fixture supplied 129,024 input tokens and generated 2,048 tokens. Each cold request started with an empty prefix. Warm requests reused 128,896 tokens. The fixed-prompt table uses four control process starts and three release process starts.

| Measurement | Pre-release control | v1.0.2 |
|---|---:|---:|
| Cold-request decode, weighted | 331.57 tok/s | 333.33 tok/s |
| Warm-request decode, weighted | 328.88 tok/s | 330.55 tok/s |
| Cold prefill, mean | 48.417 s | 48.474 s |
| Eight-case low-effort suite, weighted decode | 305.07 tok/s | 309.03 tok/s |

[Per-request timings and output hashes](../results/blackfury-v1.0.2/fixed-qualification.json) cover the control and release runs. The internal control had executable SHA-256 `56d6796ce6a09640a28755b586f80de4248123707fdbd6387a7596a9b41aa83f`. The release matched the control's recorded output bytes on the qualification fixtures. The earlier build's [numerical behavior and inherited prefill diagnostic](blackfury-stage44.md#source-build-and-limitations) remain relevant when interpreting these checks.

## Real agent session

Oh My Pi 18.20.2 ran the release engine with the same context capacity and low reasoning effort. Inputs included Nsight captures and a 99,982-token reference packet of source, schema and kernel samples. The agent built a Python trace viewer with an offline HTML report. Chromium checks covered switching rounds, zoom and telemetry. Kernel counts and GPU times were compared to integer reference values.

| Measurement | Result |
|---|---:|
| First cold prompt | 109,211 fresh tokens in 39.27 s |
| Largest actual model input | 111,600 tokens |
| Elapsed agent time, including tools and review | 14 min 02 s |
| Generated tokens / model requests | 86,553 / 99 |
| Write/edit requests | 32 |
| Write/edit request decode range | 230.34–427.61 tok/s |
| Write/edit request decode, weighted | 290.20 tok/s |
| Write/edit fresh prefill, weighted | 2,725.61 tok/s |

The decode group includes all output of requests issuing at least one write or edit, including reasoning and tool syntax. Decode uses the sum of token intervals after each first output token divided by the total native decode time. Fresh prefill uses the sum of fresh prompt tokens divided by the total native prefill time. A reviewer supplied two follow-ups while testing the viewer. [Measured agent-session data](../results/blackfury-v1.0.2/omp-coding.json)

## Native build provenance

The measured build passed 45 host tests across environment and Qwen attention-path targets. It used Zig 0.17.0, ReleaseFast, the retained CUDA device images and NVTX3 headers. The build-inputs archive on the [release page](https://github.com/Maharajahu/qwen3.8-27b-nvfp4-blackfury-exact-tensorfold/releases/tag/v1.0.2) includes the modified source overlay for TensorFold commit `6e9af930261a88bb28ac0bfb157ce8e57e9eea07`, the retained images and native build instructions. An independent Windows build reconstructed from the packaged overlay passed 15/15 build steps and 45/45 host tests. Its executable differs in file hash from the measured image. GPU benchmark figures refer to the measured image in the downloadable runtime. [Build verification](../results/blackfury-v1.0.2/build-validation.json)
