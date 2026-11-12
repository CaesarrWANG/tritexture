TriTex · 原生网格上的逐面纹理 VAE 与纹理生成
=============================================

**在线页面：https://caesarrwang.github.io/tritexture/** — 入口是 `index.html`；本地预览直接用浏览器打开各 `.html`（3D 查看器需要通过 http 访问，可 `python -m http.server`）。

TriTex 把纹理表示为三角网格每个面上的格点颜色（不展 UV、不烘焙多视角图），用一组落在表面上的 latent token 读写。这个仓库是实验报告站：每个页面对应一个阶段的实验，含训练曲线、定量表、three.js 交互式 3D 对比和细节裁剪。页面之间按时间和依赖关系排列如下，建议按顺序读；只看结论可以直接看 `ftpbr.html` 的第 1 节。

内容
----

按阅读顺序（也是实验的时间顺序）：

*   **Round P：Objaverse 全规模基线** — `index.html` — 把纹理 VAE 从 2.3 万双鞋扩展到 Objaverse 12 万个物体（4 节点 × 4 × H100，10 万步）。数据筛选、训练曲线、按面类型的误差诊断、GT / 重建 / 逐面均值的同屏 3D 对比，以及 Round Q 对高频瓶颈的定位。
*   **Round Q 续：表 latent** — `roundq.html` — 把"latent 为什么丢面内细节"追到机制层面（排除 → 定位 → 构造），据此把 latent 从 8192 个注意力向量改成按锚点、槽位、角点组织的表。小规模验证与全规模结果（held-out 25.89 → 28.83 dB）。
*   **旧模型探索：采样密度** — `oldmodel.html` — 注意力 latent 的旧模型是否被每面 28 个采样点、每步 8 个监督点限制：加大输入点、监督点、递归切分大面，各训 10k 步对照；200k 模型在切分网格上的推理。
*   **纹理 VAE 汇报** — `vae.html` — 注意力 latent 与表 latent 的架构对照与汇总结果（held-out 24.73 → 32.43 dB），按面数分档、3D 渲染 PSNR、推理时改锚点上限、KL 权重与 latent 信息量，交互式 3D / 多视角 / 细节裁剪三种定性对比。
*   **微调对照与 PBR VAE** — `ftpbr.html` — 目前最活跃的页面。单变量微调对照 A–E；5 通道（albedo + 金属度 + 粗糙度）PBR VAE 从头训 100k，续训到 200k（放开坍缩通道的 KL、加 EMA）再到 300k（去掉 KL 和后验采样）；与 TRELLIS.2 纹理 VAE 的同口径对比；推理时加密锚点、按纹理细分面；单个 latent token 的作用范围可视化；KL 权重对照。
*   **纹理 DiT** — `dit.html` — 在表 latent 上训练扩散 transformer：无条件生成的面间一致性问题（"迪斯科球"）、图像条件接入、12 个版本的迭代与全规模训练状态。

各页顶部有跳到其他页面的导航条。页面里的数字都由生成脚本从训练产物（`metrics.jsonl`、评测 json）实时读出，不是手填的。

目录
----

    index.html            Round P 报告（入口）
    roundq.html           Round Q 续：表 latent
    oldmodel.html         旧模型采样密度探索
    vae.html              纹理 VAE 汇报
    ftpbr.html            微调对照与 PBR VAE
    dit.html              纹理 DiT
    cases/<uid>/          交互式 3D 对比用的资产：几何 .glb、GT / 重建 / 基线贴图、锚点点云、切分网格、latent 影响图
    ftpbr/ vae/ dit/ oldmodel/ roundq/   各页的静态图：训练曲线、多视角渲染、细节裁剪、架构图
    README.md             本文件

生成
----

页面由 TriTex 代码库 `outputs/report/` 下的脚本生成，每页一个：`build_report.py`（index）、`build_roundq_page.py`、`build_oldmodel_page.py`、`build_vae_page.py`、`build_ft_pbr_page.py`、`build_dit_page.py`，公共样式、图表和 3D 查看器在 `report_common.py`。贴图和 3D 资产由 `scripts/cases_vis_pbr.py`、`scripts/bake_and_view.py`、`scripts/latent_influence_bake.py` 等烘焙到 `cases/`。重新生成某页只需在代码库根目录运行对应脚本，再提交本仓库。

引用
----

*   Deitke et al., *Objaverse: A Universe of Annotated 3D Objects*, CVPR 2023. https://objaverse.allenai.org/
*   Zhang et al., *3DShape2VecSet: A 3D Shape Representation for Neural Fields and Generative Diffusion Models*, SIGGRAPH 2023.
*   Microsoft, *TRELLIS.2*（其纹理 VAE 是 `ftpbr.html` 3.3b 节的对照基线）。https://github.com/microsoft/TRELLIS.2

3D 资产署名
-----------

交互式对比中的网格来自 Objaverse 1.0（Sketchfab 上传），以原始基色贴图、我们的重建和逐面均值基线展示；几何与贴图按原许可证再分发：

- `522f50cae5cc4a6c9a7fbd5b4e36bbbf` — SPIDERMAN PS4 DEMANDED BUNDLE — CC-BY-4.0 (http://creativecommons.org/licenses/by/4.0/)
- `6746867266f04bf5aed3b4f0a6df278c` — Soldier — CC-BY-4.0 (http://creativecommons.org/licenses/by/4.0/)
- `994aec4131c243dfa4cd2598e79eabaa` — WWII ambulance rétro futuriste — CC-BY-4.0 (http://creativecommons.org/licenses/by/4.0/)
- `ecbab41a56c24b7fa88203f415b56d5a` — Enternity Demon — CC-BY-4.0 (http://creativecommons.org/licenses/by/4.0/)
- `53bf4971eccb4d5dbe20db083919c44d` — Hello Neighbor: Hide and Seek - Tiger Aaron — CC-BY-4.0 (http://creativecommons.org/licenses/by/4.0/)
- `e610ab728a124989882c209d764f046d` — Miniature of a dancing mask — CC-BY-4.0 (http://creativecommons.org/licenses/by/4.0/)
- `1aac7b79651d4bad96e49c13d3a8975e` — Rippersnapper — CC-BY-4.0 (http://creativecommons.org/licenses/by/4.0/)
- `209865ec00284af5b34d3faa999e1baf` — Commander Shadowsun — CC-BY-4.0 (http://creativecommons.org/licenses/by/4.0/)
- `9a77cdf9dac043039d516fc3a764e8f5` — Dog — CC-BY-4.0 (http://creativecommons.org/licenses/by/4.0/)
- `60ef2e59ee2a492a9677c75b745cb9a1` — Bee — CC-BY-4.0 (http://creativecommons.org/licenses/by/4.0/)

许可证以 Objaverse 元数据记录为准，作者信息见 https://objaverse.allenai.org/。
