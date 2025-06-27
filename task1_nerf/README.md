# 基于 Nerfstudio 的三维重建与模型对比分析

[![Framework](https://img.shields.io/badge/Framework-Nerfstudio-blue.svg)](https://github.com/nerfstudio-project/nerfstudio)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

本项目旨在实现一个完整的、从真实世界照片到高质量三维重建与新视图合成的端到端工作流。我们利用强大的 `nerfstudio` 框架，对二种代表性的神经渲染技术进行了深入的实践与性能对比。

---

## 项目简介

本项目以一个猫形毛绒玩具作为重建目标，通过智能手机采集多角度照片，利用 COLMAP 进行相机位姿估计，最终在一个统一的框架下，训练并评估了以下二种模型：

1.  **Vanilla-NeRF**: 经典的神经辐射场，作为性能基准。
2.  **Nerfacto**: `nerfstudio` 官方推荐的、融合多种加速技术的现代混合式 NeRF 模型。


通过本项目的实践，我们深入分析了这二种技术在**训练效率**、**收敛速度**和**渲染质量**上的核心差异。

## 环境准备

本项目依赖于 Conda 进行环境管理。请确保您的服务器已正确安装 NVIDIA 驱动、CUDA 以及 Conda。

- **硬件**: 一块支持 CUDA 的 NVIDIA GPU。
- **软件**: Conda, Git, FFmpeg。

## 快速开始：一步步指南

请按照以下步骤复现本项目的完整流程。

### 1. 克隆仓库与环境设置

首先，克隆本 GitHub 仓库到您的服务器。

```bash
# 克隆官方 nerfstudio 项目
git clone [https://github.com/nerfstudio-project/nerfstudio.git](https://github.com/nerfstudio-project/nerfstudio.git)

# 进入项目目录
cd nerfstudio
```

然后，创建并激活 Conda 虚拟环境。`nerfstudio` 官方推荐使用 `pip` 进行安装。

```bash
# 创建一个新的 conda 环境
conda create --name nerfstudio -y python=3.8

# 激活环境
conda activate nerfstudio

# 升级 pip 并安装核心依赖
python -m pip install --upgrade pip
pip install torch torchvision --index-url [https://download.pytorch.org/whl/cu118](https://download.pytorch.org/whl/cu118)
pip install nerfstudio
```
*注意：在我们的实际操作中，遇到了大量环境依赖问题。请参考的报官方告文档，其中详细记录了如 `tiny-cuda-nn` 编译、`setuptools` 版本冲突、`Qt` 图形库缺失等问题的解决方案。*

### 2. 准备数据

将您自己拍摄的照片（例如 `XXX.jpg`, `YYY.jpg` ...）放入一个新建的文件夹中。

```bash
# 假设我们将数据放在 data/my-object/images 目录下
mkdir -p data/my-object/images
# 然后将您的所有照片上传或复制到这个文件夹内
```
在本实验中，我们使用了158张照片，存放在 `data/input` 目录。

### 3. 数据预处理 (COLMAP)

使用 `ns-process-data` 命令来处理您的图片，它会自动调用 COLMAP 计算相机位姿。

```bash
# --data 指向您的图片文件夹
# --output-dir 是处理后数据的输出目录，后续训练将使用这个目录
ns-process-data images --data data/my-object/images --output-dir data/my-object-processed
```
* **提示**: 此过程非常耗时（5-20分钟），并且可能会遇到各种环境问题。如遇 `Killed` 错误，请尝试缩小输入图片的尺寸；如遇 `Aborted` 或 `Qt` 相关错误，请参考官方文档中的环境修复步骤。

### 4. 模型训练

数据处理完成后，`data/my-object-processed` 文件夹里会生成一个关键的 `transforms.json` 文件。现在，我们可以用它来训练不同的模型。

* **训练 Nerfacto (推荐的快速模型)**
  ```bash
  ns-train nerfacto --data data/my-object-processed --vis viewer+tensorboard
  ```

* **训练 Vanilla-NeRF (用于对比，非常慢)**
  ```bash
  ns-train vanilla-nerf --data data/my-object-processed --vis viewer+tensorboard
  ```

### 5. 实时监控与远程访问

训练启动后，您可以通过网页查看器（端口 `7007`）和 TensorBoard（端口 `6006`）来监控进度。由于服务在远程服务器上，您需要**在您自己的电脑上**建立 SSH 端口转发通道。

**a) 访问 Viewer (实时渲染)**

1.  **在本地电脑终端中**，运行（将 `[服务器IP]` 和 `[SSH端口]` 替换为您的信息）：
    ```bash
    ssh -CNgL 7007:127.0.0.1:7007 root@[服务器IP] -p [SSH端口]
    ```
2.  输入服务器密码，保持此终端开启。
3.  在本地浏览器访问 `http://localhost:7007/`。

**b) 访问 TensorBoard (训练曲线)**

1.  **在服务器上新的终端中**，启动 TensorBoard 服务：
    ```bash
    # 激活环境
    conda activate nerfstudio
    # --logdir 指向训练开始时生成的日志目录
    tensorboard --logdir outputs/my-object-processed/nerfacto/xxxxxxxx --port 6006
    ```
2.  **在本地电脑另一个终端中**，建立新的 SSH 通道：
    ```bash
    ssh -CNgL 6006:127.0.0.1:6006 root@[服务器IP] -p [SSH端口]
    ```
3.  输入服务器密码，保持此终端开启。
4.  在本地浏览器访问 `http://localhost:6006/`。

### 6. 渲染与导出

训练完成后，您可以生成最终成果。

* **渲染视频**: 在网页查看器的 `Render` 选项卡中创建好相机轨迹后，复制其生成的 `ns-render` 命令，并在服务器终端中运行。
  ```bash
  # 示例命令，请以查看器生成的为准
  ns-render camera-path --load-config outputs/my-object-processed/nerfacto/xxxxxxxx/config.yml --camera-path-filename path/to/camera-path.json --output-path renders/my-video.mp4
  ```

* **导出点云**: 在 `Export` 选项卡中，复制 `ns-export` 命令并在终端运行。
  ```bash
  # 示例命令
  ns-export pointcloud --load-config outputs/my-object-processed/nerfacto/xxxxxxxx/config.yml --output-dir exports/pcd/
  ```

## 项目成果


* **模型权重与最终渲染视频**: [– 链接：https://pan.baidu.com/s/1LqTAJ9mqZsnp_A8q4InW-g
– 提取码：w9jw
]

## 致谢

本项目基于优秀的开源框架 `nerfstudio` 实现，数据处理核心依赖于 `COLMAP`。感谢所有为这些项目做出贡献的研究者和开发者。
