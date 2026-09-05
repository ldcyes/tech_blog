# DLSS 5 指标与 Benchmark / Metrics and Benchmarks

以下数字来自 NVIDIA 技术报告本身；推测值明确标注为 inference。

## Reported system facts

| 项目 | 报告值 | 口径 |
|---|---:|---|
| Transformer parameters | approximately 154M | §2.6；网络层数、宽度和注意力窗口未公开 |
| 4K model execution | approximately 8 ms/frame | 3840×2160 on one GeForce RTX 5090；不是完整游戏 frame time |
| Peak GPU memory | 731 MB | 与游戏共享 GPU；不是模型权重大小 |
| Precision | mostly FP8, some FP16 | 多数 GEMM/attention FP8，少量敏感 GEMM FP16 |
| Evaluation scenes | 109 | 20 path-traced + 89 non-path-traced；两个子集场景不同 |
| Valid user-study judgments | 794 | 21 HUD-biased judgments excluded |

## Content and structure alignment, 109 scenes

| Family | Metric | DLSS 5 | GPT Image 2 | Gemini 3 Pro | Direction |
|---|---|---:|---:|---:|---|
| Content/identity | DINOv2 patch distance | **0.0364** | 0.1405 | 0.1223 | lower |
| Content/identity | Image LPIPS | **0.0618** | 0.3386 | 0.3112 | lower |
| Albedo | SSIM | **0.888** | 0.787 | 0.804 | higher |
| Albedo | LPIPS | **0.121** | 0.204 | 0.199 | lower |
| Albedo | PSNR | **24.79 dB** | 20.98 dB | 21.17 dB | higher |
| Normal | Mean angular error | **12.15°** | 18.27° | 17.84° | lower |
| Normal | Acc@11.25° | **70.5%** | 57.7% | 58.4% | higher |
| Normal | Edge F1 | **0.935** | 0.808 | 0.848 | higher |
| Depth | Spearman ρ | **0.933** | 0.919 | 0.898 | higher |
| Depth | SSIM | **0.963** | 0.919 | 0.921 | higher |
| Depth | Edge F1 | **0.820** | 0.756 | 0.782 | higher |
| Depth | Edge Chamfer | **3.16 px** | 4.98 px | 3.95 px | lower |

These alignment metrics answer “did the generated result stay close to the authored CG scene?” They do not answer “is it more photorealistic?”

## Blinded pairwise photographic-realism study

The participant saw two anonymous outputs from the same CG input and judged which appeared more photographic. Content differences were explicitly excluded from the quality judgment. A tie contributes half a point.

| Comparison | Subset | Judgments | DLSS 5 preference |
|---|---|---:|---:|
| GPT Image 2 | Overall | 394 | **50.63%** |
| GPT Image 2 | Character | 265 | **57.74%** |
| Gemini 3 Pro | Overall | 400 | **62.00%** |
| Gemini 3 Pro | Character | 272 | **66.91%** |

The study supports a claim about perceived photographic realism under this protocol. It does not establish physical-lighting correctness, dynamic stability, or generalization to all games.

## Engineering inference from 154M + 8 ms

For a standard Transformer block with FFN ratio (r=4):

\[
P_{block}\approx(4+2r)d^2=12d^2
\]

One parameter-compatible candidate is 12 blocks with (d=1024):

\[
12\times12\times1024^2=151.0M
\]

This is only a candidate. Pixel-space I/O does not force every internal layer to run at full 4K. A full-4K application of 154M dense weights would require roughly:

\[
2\times3840\times2160\times154M=2554.7\;TFLOP/frame
\]

which cannot fit into 8 ms on an RTX 5090. Therefore the implementation must reuse parameters at reduced spatial groups, use multi-scale paths, use tiling, or combine high-resolution detail paths with lower-resolution compute. The report does not disclose which design is used.

## Useful RTX 5090 denominator

The RTX Blackwell architecture document lists approximately 838 TFLOPS dense FP8 with FP16 accumulation and 419 TFLOPS dense FP8 with FP32 accumulation for the RTX 5090. The 3352 AI TOPS headline is a sparse FP4-style number and should not be used as the default DLSS 5 denominator.

## Independent benchmark plan

For a reproducible engineering benchmark, measure the following separately:

1. **GPU time:** model-only CUDA event time, graphics queue time, and end-to-end present latency.
2. **Tensor utilization:** FP8/FP16 Tensor Core active cycles, issued versus eligible instructions, achieved FLOP/s.
3. **Memory:** peak allocation, DRAM bytes read/written, L2 hit rate, TMA transactions, and activation spill.
4. **Quality:** DINOv2/LPIPS content distance, albedo/normal/depth alignment, and a separate blinded realism score.
5. **Temporal behavior:** flicker, ghosting, disocclusion error, camera cuts, particles, transparency and long-sequence drift.
6. **Ablations:** native renderer, 4× ray budget, DLSS 5 disabled, model/strength/mask variants, raster/PT inputs and output resolutions.

The official report provides the first group of still-image benchmarks, but not the full dynamic and hardware-counter benchmark needed to identify the exact network.
