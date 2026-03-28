# Dex Retargeting — 项目摘要

## 项目概览
Dex Retargeting（Python 包名：`dex_retargeting`）提供多种手部重定向（retargeting）优化器，用于将**人手运动/姿态**映射为**机器人灵巧手关节运动**，覆盖：
- 从人手视频进行在线/离线重定向（teleoperation / post-process）
- 从手-物体姿态数据集进行离线重定向

主要入口与使用说明见：[README.md](README.md)。

## 仓库结构（导航）
- [src/](src/)：核心库代码
  - [src/dex_retargeting/](src/dex_retargeting/)：主要 Python 包
    - [src/dex_retargeting/retargeting_config.py](src/dex_retargeting/retargeting_config.py)：`RetargetingConfig`（从 YAML 配置构建重定向流水线）
    - [src/dex_retargeting/seq_retarget.py](src/dex_retargeting/seq_retarget.py)：`SeqRetargeting`（序列/流式重定向封装，含关节限位、滤波、warm start）
    - [src/dex_retargeting/optimizer.py](src/dex_retargeting/optimizer.py)：优化器基类 `Optimizer` 及 `PositionOptimizer` / `VectorOptimizer` / `DexPilotOptimizer`
    - [src/dex_retargeting/robot_wrapper.py](src/dex_retargeting/robot_wrapper.py)：`RobotWrapper`（基于 Pinocchio 的运动学封装）
    - [src/dex_retargeting/kinematics_adaptor.py](src/dex_retargeting/kinematics_adaptor.py)：运动学/仿真关节（mimic joint）适配
    - [src/dex_retargeting/configs/](src/dex_retargeting/configs/)：内置机器人重定向配置（YAML）
      - [src/dex_retargeting/configs/offline/](src/dex_retargeting/configs/offline/)
      - [src/dex_retargeting/configs/teleop/](src/dex_retargeting/configs/teleop/)
- [example/](example/)：示例与教程
  - [example/vector_retargeting/](example/vector_retargeting/)：从视频/摄像头进行重定向（教程：[example/vector_retargeting/README.md](example/vector_retargeting/README.md)）
  - [example/position_retargeting/](example/position_retargeting/)：从 DexYCB 等数据集进行重定向（教程：[example/position_retargeting/README.md](example/position_retargeting/README.md)）
  - [example/profiling/](example/profiling/)：性能剖析脚本
- [tests/](tests/)：单元测试
  - [tests/test_optimizer.py](tests/test_optimizer.py)
  - [tests/test_retargeting_config.py](tests/test_retargeting_config.py)
- [.github/workflows/](.github/workflows/)：CI 配置（例如：[.github/workflows/test.yml](.github/workflows/test.yml)）
- [pyproject.toml](pyproject.toml)：依赖与打包配置
- [LICENSE](LICENSE)：MIT License

## 核心模块与功能（从代码出发）
### 典型数据流
1. **加载配置**：`RetargetingConfig.load_from_file(...)` 读取 YAML（见 [src/dex_retargeting/retargeting_config.py](src/dex_retargeting/retargeting_config.py)）。
2. **构建机器人模型**：内部使用 URDF + Pinocchio，经 `RobotWrapper` 提供正运动学与雅可比等能力（见 [src/dex_retargeting/robot_wrapper.py](src/dex_retargeting/robot_wrapper.py)）。
3. **选择优化器**：根据 `type` 选择 `PositionOptimizer` / `VectorOptimizer` / `DexPilotOptimizer`（见 [src/dex_retargeting/optimizer.py](src/dex_retargeting/optimizer.py)）。
4. **序列/实时重定向**：通过 `SeqRetargeting.retarget(...)` 输入每帧参考量，输出机器人关节角；可选关节限位、低通滤波与 warm start（见 [src/dex_retargeting/seq_retarget.py](src/dex_retargeting/seq_retarget.py)）。

### 内置配置
内置了多种机器人手的 YAML 配置，分别面向离线与 teleop 任务：
- 离线： [src/dex_retargeting/configs/offline/](src/dex_retargeting/configs/offline/)
- Teleop： [src/dex_retargeting/configs/teleop/](src/dex_retargeting/configs/teleop/)

## Quickstart（从现有文档提取）
### 安装
PyPI 安装（见 [README.md](README.md)）：

```bash
pip install dex_retargeting
```

运行示例时的额外依赖（见 [README.md](README.md)）：

```bash
git clone https://github.com/dexsuite/dex-retargeting
cd dex-retargeting
pip install -e ".[example]"
```

注意：核心代码在导入时会检查 `torch` 是否可用（见 [src/dex_retargeting/__init__.py](src/dex_retargeting/__init__.py)）。如果未安装，可按其提示安装（CPU 版也可用）：

```bash
pip install torch --index-url https://download.pytorch.org/whl/cpu
```

### 示例：从人手视频重定向到机器人手（vector / DexPilot）
教程与更多参数见：[example/vector_retargeting/README.md](example/vector_retargeting/README.md)。

示例命令（原文档）：

```bash
cd example/vector_retargeting
python3 detect_from_video.py \
  --robot-name allegro \
  --video-path data/human_hand_video.mp4 \
  --retargeting-type dexpilot \
  --hand-type right \
  --output-path data/allegro_joints.pkl
```

渲染生成机器人视频（原文档）：

```bash
python3 render_robot_hand.py \
  --pickle-path data/allegro_joints.pkl \
  --output-video-path data/allegro.mp4 \
  --headless
```

### 示例：从手-物体数据集进行重定向（DexYCB）
完整流程（包含 DexYCB 数据准备、manopth/MANO 配置、可视化与多机器人对比）见：
- [example/position_retargeting/README.md](example/position_retargeting/README.md)

### 运行测试（基于仓库配置）
仓库使用 `pytest`（见 [pyproject.toml](pyproject.toml) 的 `tool.pytest.ini_options`）。开发依赖 extras 在 [pyproject.toml](pyproject.toml) 的 `project.optional-dependencies.dev` 中定义。

一个常见方式是安装 dev extras 后运行：

```bash
pip install -e ".[dev]"
pytest
```

## 备注与常见坑（来自 README）
- **关节顺序（Joint Order）**：不同 URDF 解析器/仿真器可能产生不同关节顺序；建议使用**关节名**建立映射后再将结果送入下游系统。说明与示例见 [README.md](README.md)。
- **依赖与版本**：项目要求 `Python >= 3.7, < 3.13`，并在 [pyproject.toml](pyproject.toml) 中声明 `numpy>=2.0.0` 等依赖。若你的环境依赖 `numpy<2`，请参考 [README.md](README.md) 的版本说明。
