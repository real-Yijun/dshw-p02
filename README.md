## P01：金融数据获取、管理与初步分析

本项目围绕“可复现的金融数据研究流程”展开，覆盖数据下载、存储管理、数据清洗、描述统计、可视化和回归分析六个环节。目标不仅是跑通代码，更强调研究口径统一、结果可追溯、过程可复现。

## 项目简介

- 研究对象：A 股 10 只股票（覆盖银行、汽车、房地产、白酒、能源、通讯、物流）
- 样本区间：2020-01-01 至 2026-04-07（日频）
- 研究主线：
	1. 下载并管理多源金融数据
	2. 完成标准化清洗与多表合并
	3. 进行收益率统计、相关性分析与可视化
	4. 估计 CAPM 并完成宏观敏感性回归
- 技术栈：Python, pandas, numpy, akshare, matplotlib, seaborn, scipy, statsmodels, pyarrow

## 项目结构

```text
dshw-p01/
├── readme.md
├── report.html
├── requirements.txt
├── .gitignore
├── 01_download.ipynb
├── 02_clean.ipynb
├── 03_analysis.ipynb
├── data/
│   ├── stock/
│   ├── index/
│   ├── macro/
│   ├── finance/
│   ├── clean/
│   └── combined/
├── output/
└── download_log.txt
```

## 股票列表

| 序号 | 代码 | 名称 | 行业 | 选股理由 |
|---:|:---:|:---|:---:|:---|
| 1 | 600036 | 招商银行 | 银行 | 零售金融与风险管理能力较强，作为股份制银行代表具有研究价值。 |
| 2 | 601398 | 工商银行 | 银行 | 国有大行样本，规模大、经营稳健，可与股份行形成风格对照。 |
| 3 | 002594 | 比亚迪 | 汽车 | 新能源整车与动力电池协同优势明显，具备较强行业代表性。 |
| 4 | 601238 | 广汽集团 | 汽车 | 传统车企向新能源与智能化转型路径清晰，适合做同业比较。 |
| 5 | 000002 | 万科A | 房地产 | 房地产龙头样本之一，能反映地产周期变化对股价的影响。 |
| 6 | 600519 | 贵州茅台 | 白酒 | 高端白酒龙头，盈利与现金流稳定，适合作为防御型消费样本。 |
| 7 | 601857 | 中国石油 | 能源 | 上游能源龙头，受国际油价与宏观周期影响较为显著。 |
| 8 | 600938 | 中国海油 | 能源 | 油气开发核心公司，盈利弹性较高，可与中国石油同业对比。 |
| 9 | 000063 | 中兴通讯 | 通讯 | 通讯设备核心公司，受数字基建与技术周期驱动较明显。 |
| 10 | 002352 | 顺丰控股 | 物流 | 物流行业龙头，经营表现与消费和供应链景气度联系紧密。 |

## 数据来源与口径

- 股票行情：akshare 日度后复权数据，主要接口为 `stock_zh_a_daily(adjust='hfq')`
- 市场指数：
	- 必选：沪深300（000300）
	- 自选：中证500（000905）
- 宏观指标（月频）：
	- 必选：CPI 同比
	- 自选：M2 同比
- 财务指标（近5年，长格式）：
	- ROE（净资产收益率）
	- 净利润率
	- 资产负债率

说明：下载阶段采用“主接口 + 备用接口 + 重试 + 随机延时 + 日志记录”机制，降低网络波动导致的失败率。

## 存储方式与选择理由

- 基础方式 A：CSV
	- 原始数据：`data/stock/`、`data/index/`、`data/macro/`、`data/finance/`
	- 合并数据：`data/combined/combined_data.csv`
- 进阶方式 B：Parquet
	- 清洗数据并存：`data/clean/stock_clean.csv` + `data/clean/stock_clean.parquet`

选择 Parquet 的理由：在保持与 CSV 并存的前提下，Parquet 能提供更好的类型约束、列式读取能力和在中大规模数据下更优的 I/O 表现，适合后续研究扩展。

CSV 的优点：
- 通用性强，跨工具兼容成本低
- 可读性好，便于检查与教学场景展示
- 管理简单，适合课程作业和小规模项目

CSV 的局限：
- 类型信息弱，Schema 不稳定
- 大数据量下读写速度和压缩效率有限
- 多表复杂查询依赖内存计算，扩展性较弱

## 关键产出

### Notebook
- `01_download.ipynb`：数据下载与日志管理
- `02_clean.ipynb`：缺失值处理、类型统一、去重、离群标注、宽长表转换、多表合并、Parquet 对比
- `03_analysis.ipynb`：描述统计、可视化、CAPM 回归、宏观敏感性回归（选做）

### 输出文件
- `output/desc_stats.csv`：10 只股票日收益率的描述统计结果表（年化均值、波动率、偏度、峰度、最大回撤）。
- `output/fig1_normalized_price.png`：10 只股票与沪深300的归一化收盘价走势对比图（基期=1）。
- `output/fig2_return_distribution.png`：10 只股票日对数收益率分布图（含正态拟合、均值与标准差标注）。
- `output/fig3_corr_heatmap.png`：10 只股票日收益率相关系数热力图（按行业排序并标注数值）。
- `output/fig4_macro_vs_hs300.png`：宏观指标（CPI同比）与沪深300月度收益率关系散点图（含线性拟合与 Pearson 相关系数）。
- `output/fig5_roe_compare.png`（选做）：10 只股票近 5 年 ROE 对比图，用于观察行业盈利能力差异与趋势。
- `output/capm_results.csv`：CAPM 回归汇总表，包含每只股票的 alpha、beta、显著性、置信区间和 R²。
- `output/fig6_capm_beta.png`：CAPM Beta 系数点图（含 95% 置信区间、行业分组着色及 beta=1 参考线）。
- `output/macro_gamma_results.csv`（选做）：月度宏观回归结果表，给出各股票对 CPI 同比的敏感性系数 gamma 及显著性。
- `output/fig7_macro_gamma.png`（选做）：宏观敏感性 gamma 点图（含 95% 置信区间、行业分组着色及 gamma=0 参考线）。

## 核心结论摘要

- CAPM 结果显示，样本中存在 `beta > 1` 的股票（如比亚迪、中兴通讯等），市场敏感性更高。
- Alpha 显著性在个股间存在分化，说明部分收益可能未被单因子市场模型完全解释。
- 不同股票的 `R^2` 差异明显，反映市场因子解释力存在行业和个股层面的异质性。
- 月度宏观回归（CPI 同比）显示，不同行业对通胀冲击的敏感性方向和强度并不一致。

## 如何运行

1. 创建虚拟环境并激活
	 - macOS/Linux：`python3 -m venv .venv && source .venv/bin/activate`
	 - Windows (PowerShell)：`python -m venv .venv; .venv\\Scripts\\Activate.ps1`
2. 安装依赖：`pip install -r requirements.txt`
3. 运行 `01_download.ipynb` 下载原始数据
4. 运行 `02_clean.ipynb` 完成清洗和存储
5. 运行 `03_analysis.ipynb` 生成统计、图表与回归结果
6. 打开 `report.html` 阅读完整报告

## 复现与版本说明

- 不提交本地虚拟环境目录 `.venv/`，请按上述步骤重建。
- 原始数据目录通常不提交仓库，可通过 `01_download.ipynb` 重新获取。
- 下载日志在 `download_log.txt`，用于复查每次请求状态。
- 若需重做分析报告，可执行：

```bash
jupyter nbconvert --to html 03_analysis.ipynb --output report.html
```

## GitHub 仓库

https://github.com/real-Yijun/dshw-p02
