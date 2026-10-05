# Asphalt Supplier Quote Normalizer（沥青供应商报价归一化工具）

**把供应商材料报价转化为规范化、有证据的价格数据集** —— 材料价、到货价、安装价与公开投标价严格区分。

[![Python 3.12](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![CLI](https://img.shields.io/badge/CLI-command--line-blue)](#使用)
[![Platform: Windows](https://img.shields.io/badge/Platform-Windows-0078D6)](https://www.microsoft.com/windows)

**[English](./README.md)** | 简体中文

---

## 解决什么问题

供应商价格表的单位、口径和有效期都不统一——一家"每吨"的价格和另一家的含义可能完全不同。本工具**读取供应商报价并提取**供应商、材料、混合料、单位、单价、最低起运量、附加费、生效/失效日期与地区，汇总为一个可对比的数据集。

它**硬性区分材料价、到货价、安装价与公开投标价**——绝不混并——并且只有拿到证据才映射到规范化类别。单位或口径不清楚时会被标记，而不是靠猜。

## 核心功能

- **供应商报价提取** ——供应商、材料、混合料、单位、单价、最低起运量、附加费、有效期、地区
- **严格价格类型区分** ——材料价 / 到货价 / 安装价 / 公开投标价绝不混并
- **证据化分类** ——只有有依据才归入规范化类别
- **有效期窗口** ——每条价格保留生效/失效日期
- **歧义标记** ——单位或口径不明 → 复核队列

## 安装

```bash
pip install -r requirements.txt
```

## 使用

```bash
python -m scripts.s07_supplier_quote_normalizer --project ./project --input ./fixtures/suppliers --region "City Rock"
```

加 `--help` 查看全部选项。本工具家族统一 CLI 约定：`--project <路径> --input <路径> --output <路径> --format json|csv|md|xlsx|pdf --config <路径> --verbose --dry-run`。

## 输出

- `supplier_prices.json` ——带依据与有效期窗口的规范化价格条目
- `supplier_prices.csv`
- `supplier_quote_summary.md`

## 工作原理

- **标准生命周期** ——`发现 → 摄取 → 提取 → 归一化 → 校验 → 输出 → 审计`
- **证据状态机** ——每条价格按 `提取 → 归一化 → 已校验 → 已验证` 推进；允许停在更早状态
- **绝不混并价格类型** ——材料价绝不用来对比到货价

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
