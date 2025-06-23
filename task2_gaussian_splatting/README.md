# Task 2: 基于 3D Gaussian Splatting 的物体重建与新视图合成

本项目实现了基于 3D Gaussian Splatting 的高质量物体重建与新视角合成。完整包括数据采集、COLMAP 位姿估计、模型训练、图像渲染及定量评估。

## 1. 数据准备

- 使用手机围绕静态物体拍摄约 **150 张**照片；
- 或从视频中提取图像帧；
- 将图像整理为如下结构：

```
data/
  input/
    000000.jpg
    000001.jpg
    ...
```

## 2. 使用 COLMAP 估计相机参数

- 安装 [COLMAP](https://colmap.github.io/)（建议版本 ≥ 3.8）；
- 执行以下命令完成 SfM 重建并生成训练输入：

```bash
python convert.py --input data/input --output data/splatting
```

该脚本会自动完成相机位姿估计、图像去畸变和格式转换。

## 3. 环境配置

建议使用支持 CUDA 的 NVIDIA GPU（推荐 RTX 3090 及以上，24GB 显存）。

安装依赖项：

```bash
pip install torch torchvision --extra-index-url https://download.pytorch.org/whl/cu118
pip install plyfile tqdm lpips
pip install git+https://github.com/graphdeco-inria/diff-gaussian-rasterization.git
pip install git+https://github.com/graphdeco-inria/simple-knn.git
```

也可参考 [Inria 官方仓库](https://github.com/graphdeco-inria/gaussian-splatting) 的环境配置指南。

## 4. 模型训练

使用以下命令进行 30000 次迭代训练，训练时间视硬件约为 30~60 分钟：

```bash
python train.py \
  --data data/splatting \
  --output output/gaussian_model \
  --iters 30000 \
  -w
```

参数说明：
- `--data`：预处理后的输入数据路径；
- `--output`：输出模型保存目录；
- `--iters`：训练迭代次数；
- `-w`：可选，表示使用白色背景。

训练过程中会自动保存中间检查点（如第 7000 步）。

## 5. 图像渲染

训练完成后，可在指定轨迹下渲染新视角图像：

```bash
python render.py \
  --model output/gaussian_model \
  --iters 30000 \
  --traj circular
```

结果图像保存在以下目录：

```
output/gaussian_model/renders/
```

## 6. 模型评估

可使用以下命令在测试集上评估模型性能：

```bash
python metrics.py \
  --model output/gaussian_model
```

示例输出：

```
Scene: gaussian_model
Method: ours_30000
  PSNR : 29.1
  SSIM : 0.91
  LPIPS: 0.18
```

评估结果也会保存为 `.json` 文件，便于后续分析。

## 7. 模型可视化

训练完成后，你可以使用官方提供的可视化工具直观查看模型效果。

当前模型文件夹结构如下：

```
data/
 ┣ images/
 ┣ sparse/
 ┣ output/
    ┣ cameras.json
    ┣ cfg_args
    ┣ input.ply
    ┣ point_cloud/
       ┣ iteration_7000/point_cloud.ply
       ┣ iteration_30000/point_cloud.ply
```

在 Windows 系统中，可访问以下链接下载可视化工具：

[https://repo-sam.inria.fr/fungraph/3d-gaussian-splatting/binaries/viewers.zip](https://repo-sam.inria.fr/fungraph/3d-gaussian-splatting/binaries/viewers.zip)

将压缩包解压到项目根目录下。

在资源管理器地址栏中输入 `cmd` 打开命令行，运行以下命令：

```bash
.\viewers\bin\SIBR_gaussianViewer_app -m data/output
```

你将看到训练后的 3D Gaussian 模型渲染效果。建议将操作模式切换为 `trackball`，以获得更顺畅的交互体验。

## 8. 项目结构示意

```
task2_gaussian_splatting/
├── convert.py              # 数据预处理
├── train.py                # 模型训练
├── render.py               # 渲染图像
├── metrics.py              # 性能评估
├── data/
│   ├── input/              # 原始输入图像
│   └── splatting/          # COLMAP 输出及训练输入
└── output/
    └── gaussian_model/
        ├── renders/        # 渲染结果
        ├── model_weights/  # 模型权重
        └── results.json    # 指标评估结果
```

## 9. 方法简介

3D Gaussian Splatting 用各向异性的高斯分布替代体素表示，结合 GPU 加速光栅化实现高效渲染。

- 每个高斯包含位置、协方差、透明度与球谐系数颜色参数；
- 可实现 1080p 分辨率下的实时渲染（≥100fps）；
- 相比 NeRF，该方法训练更快、渲染更清晰，并无需网格重建；
- 该方法发表于 SIGGRAPH 2023，并被评为最佳论文之一。

## 10. 快速使用指南

```bash
git clone https://github.com/yourname/your-repo-name.git
cd your-repo-name/task2_gaussian_splatting

# 第一步：COLMAP 预处理
python convert.py --input data/input --output data/splatting

# 第二步：训练模型
python train.py --data data/splatting --output output/gaussian_model --iters 30000 -w

# 第三步：渲染图像
python render.py --model output/gaussian_model --iters 30000 --traj circular

# 第四步：评估指标
python metrics.py --model output/gaussian_model
```

## 11. 参考资料

- 官方实现仓库：https://github.com/graphdeco-inria/gaussian-splatting
- SIGGRAPH 2023 论文："3D Gaussian Splatting for Real-Time Radiance Field Rendering"


