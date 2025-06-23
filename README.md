# 多视角重建与新视图合成项目

本项目包含两个主要任务，分别实现了基于 NeRF 及其加速技术（如 TensoRF）和基于 3D Gaussian Splatting 的物体三维重建与新视角渲染。两个任务都从拍摄多角度图像开始，结合 COLMAP 获取相机参数，再通过训练不同方法实现三维重建与新视角渲染。

## 📁 项目结构

```
project-root/
├── task1_nerf/                        # 任务1：NeRF 或其加速版本实现
│   ├── data/                          # 输入图像及 COLMAP 输出
│   ├── configs/                       # 模型配置文件
│   ├── train.py                       # 训练脚本
│   ├── render.py                      # 渲染脚本
│   ├── eval.py                        # PSNR、SSIM 等评估脚本
│   └── README.md                      # 任务1使用说明文档
│
├── task2_gaussian_splatting/         # 任务2：3D Gaussian Splatting 实现
│   ├── data/                          # 图像输入 + COLMAP 输出
│   ├── convert.py                     # COLMAP 预处理
│   ├── train.py                       # 训练脚本
│   ├── render.py                      # 渲染脚本
│   ├── metrics.py                     # 评估脚本
│   ├── viewers/                       # 可视化工具（Windows 平台）
│   └── README.md                      # 任务2使用说明文档
│
├── requirements.txt                  # 项目依赖
└── README.md                          # 项目总览文档（本文件）
```

##  任务概览

### Task 1：基于 NeRF 的物体重建与新视角合成

- 使用 COLMAP 获取相机位姿；
- 采用 NeRF 或加速技术（如 Plenoxels、TensoRF）进行体积渲染模型训练；
- 在新轨迹下渲染高质量图像；
- 使用 PSNR、SSIM、LPIPS 等指标对比 NeRF 与其变体的效果与效率；
- 支持 TensorBoard 可视化训练过程。

 详细使用说明请见：`task1_nerf/README.md`

---

### Task 2：基于 3D Gaussian Splatting 的物体重建与新视角合成

- 使用相同图像输入，通过 COLMAP 获得相机参数；
- 使用 SIGGRAPH 2023 提出的 Gaussian Splatting 方法进行训练；
- 支持实时渲染、高效可视化，输出清晰图像；
- 提供 viewer 工具查看训练结果，并使用 `metrics.py` 自动评估性能。

 详细使用说明请见：`task2_gaussian_splatting/README.md`

---

##  注意事项

- 建议 GPU 显存≥24GB（如 3090/4090）；
- 所有任务均推荐在 Linux 或 Windows WSL 环境下运行；
- 训练及渲染命令均包含在各任务子文件夹的 README.md 中；
- 若使用 Windows，可直接运行可视化工具查看训练模型。

##  相关链接

- COLMAP：https://colmap.github.io/
- Gaussian Splatting：https://repo-sam.inria.fr/fungraph/3d-gaussian-splatting/
- NeRF 基础论文：https://arxiv.org/abs/2003.08934
