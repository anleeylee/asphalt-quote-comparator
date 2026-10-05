# Asphalt Quote Comparator（沥青报价对比工具）

**把承包商沥青报价归一化为可对比的表格** —— $/SF、$/吨、隐含厚度与范围标记，全部带证据，不做排名。

[![Python 3.12](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![CLI](https://img.shields.io/badge/CLI-command--line-blue)](#使用)
[![Platform: Windows](https://img.shields.io/badge/Platform-Windows-0078D6)](https://www.microsoft.com/windows)

**[English](./README.md)** | 简体中文

---

## 解决什么问题

承包商报价单的格式五花八门——单位不同、工作范围条目不同、有的带吨数有的不带，公平对比就像一场表格噩梦。本工具把一份或多份**报价文本 / PDF / 电子表格解析为可对比数据集**：面积、吨数、厚度、工作范围条目、单价、包干价与总价，材料 / 运输 / 人工 / 设备 / 拆除分类严格分离，并保留原始措辞。

确定性派生指标（$/SF、$/吨、隐含厚度）只依据报价中陈述的数量计算。标记会指出图纸数量不符、规范厚度不符、缺少范围、缺少吨数与缺少总价。**本工具绝不挑选"最佳"承包商**——决定权始终在业主或项目经理手中。

## 核心功能

- **多格式报价解析** ——PDF / 文本 / 电子表格承包商报价
- **同口径归一化** ——面积、吨数、厚度、范围条目、单价、包干价
- **成本类别分离** ——材料 / 运输 / 人工 / 设备 / 拆除绝不混并
- **确定性派生指标** ——$/SF、$/吨、隐含厚度仅依据陈述数量计算
- **关键标记** ——图纸数量不符、规范厚度不符、缺范围 / 缺吨数 / 缺总价
- **不做排名** ——只给证据与对比，决策交给用户

## 安装

```bash
pip install -r requirements.txt
```

## 使用

```bash
python -m scripts.s05_quote_comparator --project ./project --input ./fixtures/quotes
```

加 `--help` 查看全部选项。本工具家族统一 CLI 约定：`--project <路径> --input <路径> --output <路径> --format json|csv|md|xlsx|pdf --config <路径> --verbose --dry-run`。

## 输出

- `quote_comparison.json` ——归一化报价 + 派生指标 + 标记
- `quote_comparison.xlsx` ——并排对比表
- `quote_flags.md` ——人类可读的关键标记

## 工作原理

- **标准生命周期** ——`发现 → 摄取 → 提取 → 归一化 → 校验 → 计算/分类 → 输出 → 审计`
- **证据状态机** ——每条报价按 `提取 → 归一化 → 已校验 → 已验证` 推进；允许停在更早状态
- **保留原文措辞** ——归一化不丢失承包商实际写下的内容

## 质量保证

- 每条记录做确定性 schema 校验；
- 黄金样例 fixtures 与精确期望值（`tests/`）；
- 复核队列：低置信度、比例不明与冲突进入 `review_queue.json` / 摘要——绝不静默修正；
- 审计日志：每次运行将 `run_id`、耗时、输入哈希、引擎版本与输出写入 `<project>/audit/`。

## 测试

```bash
python -m pytest -q
```

## 安全与隐私

本地文件默认留在本地。API 密钥存放在环境变量（`ASPHALTCOSTS_API_KEY`、`ADI_AI_API_KEY`）——绝不写入源码。派生文件写入 `working/` 或 `output/`；来源文件永不修改。

## 许可证

MIT —— 见 [LICENSE](LICENSE)。属 [Asphalt Desktop Intelligence](https://github.com/anleeylee/asphalt-desktop-intelligence) 工具集的一部分。

## 计算引擎

[**AsphaltCosts.com**](https://asphaltcosts.com/) 是确定性计算层：面积 → 压实体积 → 净吨数 → 订购吨数（损耗只计一次）→ 车次 → 材料成本，内置有出处的规划默认值（FHWA 密度 145 lb/ft³，损耗率与车容量可编辑）。本工具把测量并校验后的输入喂给该引擎（或其带标签的本地镜像 `asphaltcosts-web-engine/1.0-mirror`），绝不重写公式。
