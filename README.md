# MLIP_demo

**AI4S 公开课实战：训练一个 toy MLIP，并判断它什么时候不可信**

本仓库提供一个 15-20 分钟课堂实战 notebook：`MLIP_demo.ipynb`。实战使用三维水分子 toy system 和解析 toy potential 生成“伪 DFT”能量与力标签，不依赖外部数据文件。

## 题目描述

给定一个三维水分子 toy 势能面，本实战首先训练一个轻量神经网络势函数，并通过自动微分从能量得到原子力。随后，学生将检查模型是否满足能量旋转不变性和力旋转等变性，比较模型在 ID 与 OOD 构型上的误差，并利用多模型 committee disagreement / oracle selection 思路选择下一轮最值得补充标注的构型。通过这个 demo，学生将理解：可靠的 MLIP 不只是一个低测试误差的模型，而是物理约束、数据覆盖与主动学习闭环共同作用的结果。

## 学习目标

1. 训练一个轻量神经网络势函数，学习构型到能量和力的映射。
2. 检查能量旋转不变性与力旋转等变性，理解物理对称性为什么是 sanity check。
3. 比较 ID / OOD 误差，并通过一次数据回流模拟主动学习闭环。

## 项目结构

```text
MLIP_demo/
├── README.md                 # 实战说明与运行方式
├── environment.yml           # Conda 环境配置
├── requirements.txt          # pip 依赖
├── MLIP_demo.ipynb           # 主 notebook：数据生成、训练、OOD 诊断与主动学习
├── assets/
│   └── 3Dmol-min.js          # 本地 3D 分子渲染库
└── outputs/
    ├── figures/              # 训练曲线、误差对比、主动学习结果图
    ├── config_space_molecule_viewer.html
    │                         # 构型空间交互图：点击点查看分子结构
    └── high_error_ood_molecule_viewer.html
                              # 高误差 OOD 构型交互图
```

## 环境依赖

推荐使用 Python 3.10 或更高版本。默认使用 CPU 即可运行，不需要 GPU。

主要依赖包括：

- `numpy`
- `pandas`
- `matplotlib`
- `torch`
- `plotly`
- `ipywidgets`
- `anywidget`
- `py3Dmol`
- `jupyterlab` 或 `notebook`

## 安装方式一：Conda / Miniconda 推荐

macOS / Linux / Windows 都可以使用这一方式。Windows 用户建议在 **Anaconda Prompt** 或 **Miniconda Prompt** 中运行。

```bash
conda env create -f environment.yml
conda activate mlip-demo
```

如果创建环境时下载较慢，可以使用 `mamba`：

```bash
mamba env create -f environment.yml
conda activate mlip-demo
```

## 安装方式二：venv + pip

macOS / Linux：

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
```

Windows PowerShell：

```powershell
py -3 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
```

如果 PowerShell 不允许激活虚拟环境，可以先执行：

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

然后重新运行：

```powershell
.\.venv\Scripts\Activate.ps1
```

Windows CMD：

```bat
py -3 -m venv .venv
.venv\Scripts\activate.bat
python -m pip install --upgrade pip
pip install -r requirements.txt
```

说明：本地虚拟环境目录 `.venv/`、`.venv_mlip_test/` 等已经写入 `.gitignore`，不应该上传到 GitHub。

## 运行方式

启动 JupyterLab：

```bash
jupyter lab MLIP_demo.ipynb
```

或者启动 classic notebook：

```bash
jupyter notebook MLIP_demo.ipynb
```

如果使用 VS Code：

1. 打开仓库目录 `MLIP_demo/`。
2. 打开 `MLIP_demo.ipynb`。
3. 选择刚创建的 Python 环境，例如 `mlip-demo` 或 `.venv`。
4. 点击 `Run All` 从头运行 notebook。

所有数据都会在 notebook 中自动生成，不依赖外部数据文件。运行过程中会训练两个 toy MLIP，并进行一次主动学习式数据回流：从 OOD 区域选择高误差构型加入训练集，然后从头训练新的 `InvariantFeatureNet`。

## OOD 构型说明

notebook 中的 ID 数据是接近平衡水分子的构型。OOD 数据用于模拟训练数据覆盖之外的情况：

- `OOD_highT`：键长和角度波动更大，模拟更高温度下的构型扰动。
- `OOD_stretch`：一个 O-H 键被明显拉伸，模拟键拉伸或反应路径上的构型。
- `OOD_compress`：一个 O-H 键被明显压缩，模拟短程排斥区域。
- `OOD_angle`：H-O-H 夹角远离平衡角，模拟角度畸变区域。

## 结果预览

Notebook 运行过程中会把关键静态图保存到 `outputs/figures/`，并生成两个可直接用浏览器打开的交互式 HTML 页面。这样即使不运行 notebook，也可以先查看主要结果。

- `outputs/config_space_molecule_viewer.html`：完整构型空间交互图。点击 3D 散点中的任意构型，右侧会显示对应三维水分子的球棍结构。
- `outputs/high_error_ood_molecule_viewer.html`：高误差 OOD 构型交互图。红色点是 force RMSE 最高的 OOD 构型，点击后可以查看对应分子结构。

如果 3D 交互图第一次打开没有立刻显示分子结构，可以等待几秒，或点击任意一个数据点触发右侧分子渲染。

训练 loss 用来确认两个势函数是否正常收敛。这里的 loss 同时包含能量和力。

![Training loss](outputs/figures/01_training_loss.png)

ID/OOD RMSE 对比展示模型在训练分布附近和训练分布之外的误差差异。对数坐标用于突出数量级变化。

![ID/OOD RMSE by model](outputs/figures/02_id_ood_rmse_by_model.png)

坐标变换误差用于检查模型是否能正确处理平移和旋转后的同一个分子构型。

![Coordinate transform error](outputs/figures/03_coordinate_transform_error.png)

对称性检查进一步比较能量不变性和力旋转等变性，帮助发现模型表示是否违反基本物理约束。

![Symmetry error check](outputs/figures/04_symmetry_error_check.png)

高误差 OOD 构型图把模型最不可靠的点放回构型空间中，便于观察它们是否落在训练数据覆盖不足的区域。

![High-error OOD locations](outputs/figures/05_high_error_ood_locations.png)

主动学习回流图比较补充 OOD 标注前后的 force RMSE，用来说明数据回流对可靠性的影响。

![Active-learning update](outputs/figures/06_active_learning_update.png)

## 交付文件

- `MLIP_demo.ipynb`
- `README.md`
- `requirements.txt`
- `environment.yml`
- `assets/3Dmol-min.js`
- `outputs/figures/*.png`
- `outputs/config_space_molecule_viewer.html`
- `outputs/high_error_ood_molecule_viewer.html`
