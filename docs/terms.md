# DLSS 5 专业术语 / Technical Glossary

## 中文

| 术语 | 含义 |
|---|---|
| 3D-guided neural rendering | 由渲染器输出的 RGB、运动矢量和时域状态约束的生成式最终渲染阶段；“3D-guided”指约束来自 3D 渲染管线，而不是文本提示。 |
| Renderer-conditioned | 模型以当前 renderer frame 为主要条件，保持像素对齐的物体、边界、构图和遮挡证据。 |
| Pixel-space diffusion | 在 RGB/像素空间进行扩散或去噪，不经过有损 VAE latent 编解码；不等于所有内部特征都保持 4K 分辨率。 |
| One-step diffusion | 每个输入帧只执行一次确定性的扩散推理，而不是多步随机采样。 |
| Causal streaming | 当前输出只依赖当前帧和过去状态，不等待未来帧；适合交互式渲染。 |
| Carried temporal state | 从过去帧延续到当前帧的有限状态，用于时间稳定性；报告未公开其具体表示和容量。 |
| Appearance prior | 从真实照片统计学习到的材质、照明、皮肤、头发和植被外观分布。 |
| Artistic direction | 开发者对 model subset、structure strength、tone strength、颜色和遮罩的控制。 |
| Structure intensity | 主要影响高空间频率细节和局部对比，例如 ambient occlusion、反射、次表面散射。 |
| Tone intensity | 主要影响低空间频率的亮度、颜色和大尺度环境照明响应。 |
| Manual masking | 引擎为每个像素提供区域强度/色调控制，精度高但增加开发与渲染成本。 |
| Automatic semantic masking | 模型内部学习“skin”等语义区域，免去显式手工 mask；未来可扩展到头发、 foliage 和材质。 |
| Albedo | 与照明无关的表面固有颜色；训练期用于约束模型不要改变作者指定的材质色彩。 |
| Surface normal | 表面局部朝向向量；用于衡量几何朝向与边界保持。 |
| Depth ordering | 像素之间的相对远近关系；DLSS 5 评测使用 Spearman ρ、SSIM、Edge F1 和 Chamfer distance。 |
| DINOv2 patch distance | 在对应 ViT-B/14 patch 特征空间衡量内容/身份差异；越低表示越接近 CG 输入，不表示照片真实感更高。 |
| LPIPS | 感知特征距离；报告将 image LPIPS 用于内容/身份保持，将 albedo LPIPS 用于结构一致性。越低越好。 |
| SSIM / PSNR | 图像结构相似度 / 峰值信噪比；通常越高越好。 |
| Mean angular error | 生成图像估计的表面 normal 与 CG 输入的平均夹角误差，单位为度；越低越好。 |
| Edge F1 / Edge Chamfer | 几何边界检测的 F1 与边缘距离；分别衡量边界保留和位置偏移。 |
| Tensor Memory Accelerator (TMA) | Blackwell 的数据搬运机制，用于把 global memory 数据高效 staged 到片上存储。 |
| FP8 / FP16 accumulation | FP8 输入 GEMM 的累加可以采用 FP16 或 FP32；两者改变峰值吞吐口径，报告没有公开具体比例。 |

## English

| Term | Meaning |
|---|---|
| 3D-guided neural rendering | A generative final-rendering stage constrained by renderer RGB, motion vectors and temporal state. |
| Renderer-conditioned | The current rendered frame is the primary, pixel-registered condition that anchors objects, boundaries, composition and visibility. |
| Pixel-space diffusion | Diffusion operates in image/RGB space without a lossy VAE round trip. It does not prove that every internal feature remains at 4K. |
| One-step diffusion | One deterministic denoising evaluation per rendered frame instead of iterative stochastic sampling. |
| Causal streaming | A frame is produced from the current frame and past state without waiting for future frames. |
| Appearance prior | Statistical knowledge of real-world material, illumination, skin, hair and foliage appearance learned from photographs. |
| Structure / tone intensity | Learned high-frequency detail/local contrast control and low-frequency illumination/color control, respectively. |
| DINOv2 patch distance | A content/identity distance in corresponding ViT-B/14 feature patches; lower is better, but it is not a realism score. |
| LPIPS | A learned perceptual feature distance. Lower values indicate closer appearance to the CG input. |
| Normal angular error | Mean angle between estimated output and input surface normals, in degrees. Lower is better. |
| Edge F1 / Chamfer | Boundary preservation F1 and edge displacement distance. |
| TMA | Blackwell Tensor Memory Accelerator for staging global-memory data into on-chip storage. |
| FP8 accumulation mode | FP8 tensor inputs may accumulate in FP16 or FP32; this changes the correct peak-throughput denominator. |


## 补充解释 / Extended notes

### DINOv2 image feature distance

DINOv2 是自监督 ViT 图像编码器。报告先将输入 CG 与输出 RGB 缩放到 672×378，再比较对应 ViT-B/14 patch 特征；DLSS 5 的 0.0364 越低表示越接近作者输入。它衡量内容/身份保持，不是照片真实度。

DINOv2 is a self-supervised vision encoder. The report compares corresponding ViT-B/14 patch features after resizing to 672×378. Lower distance means closer content/identity alignment, not higher photographic realism.

### Image LPIPS 与 albedo LPIPS

Image LPIPS 在最终 RGB 上比较感知特征；albedo LPIPS 先从输入/输出估计 albedo，再比较固有色与材质布局。二者都是 lower-is-better distance，但不等价于人类照片真实度盲测。LPIPS、SSIM、PSNR 应结合读取：感知距离、结构相似度和像素误差的侧重点不同。

Image LPIPS compares final RGB perceptual features. Albedo LPIPS compares estimated intrinsic-color maps. The latter is a targeted check against changing authored material color while adding photographic lighting cues.

### Albedo / normal / depth

- **Albedo / 反照率**：中性光照解释下的固有表面颜色，检查材质本色与区域布局。
- **Normal / 表面法线**：局部 3D 朝向；mean angular error 越低越好，Edge F1 检查几何边界。
- **Depth / 深度**：可见表面的距离排序；Spearman ρ 检查相对深度顺序，SSIM/Edge F1/Chamfer 检查局部结构与断边。
- **Estimator / 估计器**：同时处理输入和输出 RGB、生成可比的 albedo/normal/depth 诊断图；它不是 DLSS 5 的部署输入。

### PT 与 non-PT

PT（path tracing）通过采样光线路径估计光传输，通常还包含 temporal accumulation 与 denoising；non-PT 是报告中的非 path-traced 子集，可能采用 raster、RT 或混合管线。两者都在 DLSS 5 之前提供 RGB 与运动对应。报告的 20 个 PT 与 89 个 non-PT 场景不是同一场景配对，因此只能描述覆盖，不能解释为 PT 的因果收益。
