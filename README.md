# DLSS 5 Interactive Technical Review / 交互式技术综述

基于 NVIDIA《DLSS 5: Real-Time Generative Rendering》技术报告的双语交互式综述。

This repository contains a Chinese and an English interactive technical review of NVIDIA's DLSS 5 report.

在线阅读（GitHub Pages）：<https://ldcyes.github.io/tech_blog/>

- [中文交互报告](zh/index.html)
- [English interactive report](en/index.html)
- [专业术语 / Technical glossary](docs/terms.md)
- [指标与 benchmark / Metrics and benchmarks](docs/benchmarks.md)

内容覆盖：

- DLSS 5 与传统重建式 DLSS 的范式差异
- RGB、运动矢量、时域状态和艺术控制组成的推理管线
- 154M Transformer、单步像素空间扩散、FP8 与 Blackwell TMA
- Chunk-based 与逐帧因果推理的延迟模拟
- Structure、Tone、区域遮罩交互演示
- 109 个场景、12 项结构指标和 794 次有效盲测
- 4K 约 8 ms、731 MB 显存的帧预算计算器
- 物理正确性、输入质量、风格适用性和分布漂移局限

## 文件

- `index.html`：中英文入口页
- `zh/index.html`：中文版交互式报告
- `en/index.html`：English interactive report
- `docs/terms.md`：中英专业术语解释
- `docs/benchmarks.md`：报告指标、benchmark 数值与独立测试计划
- `.github/workflows/pages.yml`：GitHub Pages 自动部署工作流
- `dist/index.html`：备用静态报告副本

## 资料来源

- [DLSS 5 项目页](https://research.nvidia.com/labs/adlr/DLSS5/)
- [DLSS 5 技术报告](https://research.nvidia.com/labs/adlr/files/DLSS5_Report.pdf)
- [RTX Blackwell GPU Architecture](https://images.nvidia.com/aem-dam/Solutions/geforce/blackwell/nvidia-rtx-blackwell-gpu-architecture.pdf)

交互模拟用于解释报告中的关系，不代表 NVIDIA 未公开的实际网络实现或额外实测结果。The calculator values and architecture candidates are explicitly engineering inferences.
