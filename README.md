# SysGPT Dataset and Benchmark
This repository contains the full dataset and evaluation benchmark introduced in
our OSDI'25 paper:

"Principles and Methodologies for Serial Performance Optimization (OSDI' 25)"

## Overview

Large language models (LLMs) hold promise as assistants for system performance
optimization, yet their evaluation in this domain remains underexplored. This
repository provides:

- A curated dataset of performance optimization problems and observations, derived from 10 years of SOSP/OSDI papers
- A taxonomy-grounded benchmark to assess LLMs' ability to suggest concrete, actionable system optimizations
- Scripts to evaluate models on their ability to recover real-world optimization strategies

## Contents

```
.
├── dataset/
│ ├── dataset.xlsx  # Full training + test data (see below)
│ ├── example_3     # Few-shot prompt examples (N = 3)
│ ├── example_5     # Few-shot prompt examples (N = 5)
│ └── example_10    # Few-shot prompt examples (N = 10)
│
├── eval.py         # Evaluation script (e.g., precision/recall)
├── run_test.sh     # Script to reproduce Figure 7
└── README.md

```

### `dataset.xlsx`

- Sheet 1: Training dataset distilled from 10 years of OSDI/SOSP papers (2013–2022).
- Sheet 2: Test dataset of 96 papers published in 2024 (OSDI/SOSP).
- Each entry includes a problem statement, system observations, and labeled methodologies.


## Citation
If you use this dataset or benchmark, please cite:

    ```
    @inproceedings{park:sysgpt,
      title        = {{Principles and Methodologies for Serial Performance Optimization}},
      author       = {Sujin Park and Mingyu Guan and Xiang Cheng and Taesoo Kim},
      booktitle    = {Proceedings of the 19th USENIX Symposium on Operating Systems Design and Implementation (OSDI)},
      month        = jul,
      year         = 2025,
    }
    ```

## 运行方法

### 1. 环境准备

在运行评估脚本之前，需要确保安装了所需的依赖包：

```bash
pip install openai openpyxl
```

### 2. 配置参数

在 `eval.py` 文件中，需要修改以下配置：

- `API_KEY`: 替换为您的 OpenAI API 密钥
- `MODEL_KEY`: 替换为您的微调模型 ID

### 3. 单一测试运行

可以直接运行 `eval.py` 脚本进行单一测试，需要提供两个命令行参数：

- 第一个参数：生成温度（控制输出随机性）
- 第二个参数：每个问题的尝试次数（取最优结果）

示例：

```bash
python3 eval.py 0.0 1
```

上述命令将使用温度 0.0 和尝试次数 1 运行评估。

### 4. 批量测试运行

使用 `run_test.sh` 脚本可以自动化运行不同参数组合的测试：

```bash
chmod +x run_test.sh  # 赋予执行权限
./run_test.sh
```

该脚本会运行以下参数组合：
- 生成温度：0.0, 0.1, 0.2, 0.3, 0.4, 0.5, 0.7
- 尝试次数：1, 3, 5, 10

总计 7 × 4 = 28 次测试。

### 5. 结果输出

- 测试结果会实时显示在终端上
- 同时会保存到 `./auto_log` 目录下的日志文件中
- 日志文件名格式：`temp[温度]-best[尝试次数].log`

### 6. 结果解读

测试结果包含以下指标：

- **精确率（Precision）**：模型预测正确的方法论占所有预测方法论的比例
- **召回率（Recall）**：模型预测正确的方法论占真实方法论的比例
- **F1 分数**：精确率和召回率的调和平均值，综合反映模型性能

示例输出：

```
=== sysgpt ===
avg_precision 0.85
avg_recall 0.72
avg_f1 0.78

=== few-shot ===
avg_precision 0.65
avg_recall 0.58
avg_f1 0.61
```

### 7. 目录结构说明

运行测试后，项目目录结构将变为：

```
.
├── auto_log/            # 测试日志目录
│   ├── temp0.0-best1.log
│   ├── temp0.0-best3.log
│   └── ...
├── dataset/
│   ├── dataset.xlsx
│   ├── example_3
│   ├── example_5
│   └── example_10
├── eval.py
├── run_test.sh
└── README.md
```

### 8. 注意事项

- 运行测试需要消耗 OpenAI API 额度，请确保您的账户有足够的余额
- 批量测试可能需要较长时间完成，建议在后台运行
- 测试过程中如果遇到 API 错误，脚本会自动重试
- 日志文件会占用一定磁盘空间，建议定期清理

## 评估指标说明

本项目使用以下指标评估模型性能：

1. **方法论提取**：从模型生成的文本中提取性能优化方法论标签
2. **向量转换**：将方法论标签转换为二进制向量表示
3. **指标计算**：
   - 真阳性（TP）：真实为 1 且预测为 1
   - 假阳性（FP）：真实为 0 但预测为 1
   - 假阴性（FN）：真实为 1 但预测为 0
   - 精确率 = TP / (TP + FP)
   - 召回率 = TP / (TP + FN)
   - F1 分数 = 2 × (精确率 × 召回率) / (精确率 + 召回率)

这些指标能够全面评估模型在系统性能优化方法论预测任务上的表现。
