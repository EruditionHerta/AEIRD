# 数据文档

本文档描述第三期 Anthropic Economic Index（AEI）报告所使用的数据来源与变量。

## Claude.ai 使用数据

### 概述
核心数据集包含按地理和分析维度（分面，facet）聚合的 Claude AI 使用指标。

- 源文件:
  - `aei_raw_claude_ai_2025-08-04_to_2025-08-11.csv`（富化前数据，位于 data/intermediate/）
  - `aei_enriched_claude_ai_2025-08-04_to_2025-08-11.csv`（富化后数据，位于 data/output/）

- 关于数据来源的说明：AEI 原始文件包含原始计数与百分比。派生指标（指数、等级、按人计算、自动化/增强百分比）在[...]期间计算。

### 数据架构
每一行表示某一特定地理与分面组合下的一个指标值：

| 列名 | 类型 | 说明 |
|------|------|------|
| `geo_id` | string | 地理标识符（国家为 ISO-2 国家代码、美国州为州代码，或为 "GLOBAL"，富化数据中为 ISO-3 国家代码） |
| `geography` | string | 地理层级："country"、"state_us" 或 "global" |
| `date_start` | date | 数据采集期开始日期 |
| `date_end` | date | 数据采集期结束日期 |
| `platform_and_product` | string | "Claude AI (Free and Pro)" |
| `facet` | string | 分析维度（见下文“分面”） |
| `level` | integer | 分面层级（0-2） |
| `variable` | string | 指标名称（见下文“变量”） |
| `cluster_name` | string | 分面内的具体实体（任务、模式等）。交叉分面使用 "base::category" 格式 |
| `value` | float | 数值型指标值 |

### 分面
- country：国家层级聚合
- state_us：美国州层级聚合
- onet_task：O*NET 职业任务
- collaboration：人机协作模式
- request：请求复杂度层级（0 = 最高粒度，1 = 中等粒度，2 = 最低粒度）
- onet_task::collaboration：任务与协作模式的交叉
- request::collaboration：请求类别与协作模式的交叉

### 核心变量

变量遵循 `{prefix}_{suffix}` 模式，含义如下：

- AEI 处理产生：`*_count`、`*_pct`
- 富化步骤产生：`*_per_capita`、`*_per_capita_index`、`*_pct_index`、`*_tier`、`automation_pct`、`augmentation_pct`、`soc_pct`

#### 使用度量
- `usage_count`：该地理区域的会话/交互总数
- `usage_pct`：占总使用量的百分比（相对于父级地理：国家以全球为基准、美国各州以美国为基准）
- `usage_per_capita`：使用次数除以工作年龄人口
- `usage_per_capita_index`：集中度指数，用于衡量相对于人口份额该地理区域使用是否超/低预期（1.0 = 与人口占比相称，>1.0 = 代表性过高，<1.0 = 代表性不足[...]）
- `usage_tier`：使用采用等级（0 = 无/低采用，1-4 = 在具有足够使用量的地理中按采用程度四分位）

#### 内容分面指标
- O*NET 任务指标:
  - `onet_task_count`：使用该 O*NET 任务的会话数量
  - `onet_task_pct`：该任务在该地理总量中的占比
  - `onet_task_pct_index`：相对基线（国家用全球、州用美国）的专业化指数
  - `onet_task_collaboration_count`：同时包含该任务与某协作模式的会话数量（交叉）
  - `onet_task_collaboration_pct`：在该任务基数内该协作模式所占百分比（对每个任务内求和为 100%）

#### 职业指标
- `soc_pct`：与某 SOC 大类职业组（如管理类、计算机与数学类）相关的、已分类 O*NET 任务所占百分比

- 请求指标:
  - `request_count`：处于该请求类别层级的会话数量
  - `request_pct`：该类别在该地理总量中的占比
  - `request_pct_index`：相对基线的专业化指数
  - `request_collaboration_count`：同时属于该请求类别与某协作模式的会话数量（交叉）
  - `request_collaboration_pct`：在该请求基数内该协作模式所占百分比（对每个请求内求和为 100%）

- 协作模式指标:
  - `collaboration_count`：该协作模式的会话数量
  - `collaboration_pct`：该协作模式在该地理总量中的占比
  - `collaboration_pct_index`：相对基线的专业化指数
  - `automation_pct`：可分类的协作中偏自动化的占比（指令型、反馈循环模式）
  - `augmentation_pct`：可分类的协作中偏增强的占比（验证、任务迭代、学习模式）

#### 人口与经济指标
- `working_age_pop`：工作年龄人口（15–64 岁，采用世界银行定义）
- `gdp_per_working_age_capita`：总 GDP 除以工作年龄人口（美元）

#### 特殊取值
- `not_classified`：表示为保护隐私被过滤或无法分类的数据
- `none`：表示该属性缺失（如无协作、未选择任务）

### 数据处理说明
- 最小观测量：每个国家至少 200 次会话、每个美国州至少 100 次（在富化步骤中应用，非原始预处理）
- 人口基数：工作年龄人口（15–64 岁）
- `not_classified`：
  - 对常规分面：捕获被过滤/未分类的会话
  - 对交叉分面：每个基础聚类有其各自的 `not_classified`（如 "task1::not_classified"）
- 交叉百分比：相对于基础聚类总量计算，确保每个基础聚类内百分比合计为 100%
- 百分比指数计算：
  - 从指数计算中排除 `not_classified` 与 `none` 类别，因为其不具意义
- 国家代码：原始数据为 ISO-2（如 "US"），富化后国家为 ISO-3（如 "USA"、"GBR"、"FRA"）
- 变量定义：见上文“核心变量”部分

## 1P API 使用数据

### 概述
基于 1P API 流量样本并采用隐私保护方法分析的第一方 API 使用指标，涵盖多个维度。

- 源文件：`aei_raw_1p_api_2025-08-04_to_2025-08-11.csv`（位于 data/intermediate/）

### 数据架构
每一行表示全局层级下某一特定分面组合的一个指标值：

| 列名 | 类型 | 说明 |
|------|------|------|
| `geo_id` | string | 地理标识符（API 数据始终为 "GLOBAL"） |
| `geography` | string | 地理层级（API 数据始终为 "global"） |
| `date_start` | date | 数据采集期开始日期 |
| `date_end` | date | 数据采集期结束日期 |
| `platform_and_product` | string | "1P API" |
| `facet` | string | 分析维度（见下文“分面”） |
| `level` | integer | 分面层级（0-2） |
| `variable` | string | 指标名称（见下文“变量”） |
| `cluster_name` | string | 分面内具体实体。对交叉分面：格式为 "base::category"；对均值指标：使用 "base::index"/"base::count" |
| `value` | float | 数值型指标值 |

### 分面
- onet_task：O*NET 职业任务
- collaboration：人机协作模式
- request：请求类别（自底向上分类法的 0–2 层级）
- onet_task::collaboration：任务与协作模式的交叉
- onet_task::prompt_tokens：按任务的平均提示 tokens（归一化，均值 = 1.0）
- onet_task::completion_tokens：按任务的平均完成 tokens（归一化，均值 = 1.0）
- onet_task::cost：按任务的平均成本（归一化，均值 = 1.0）
- request::collaboration：请求类别与协作模式的交叉

### 核心变量

#### 使用度量
- `collaboration_count`：具有该协作模式的 1P API 记录数
- `collaboration_pct`：该协作模式在总量中的占比

#### 内容分面指标
- O*NET 任务指标:
  - `onet_task_count`：使用该 O*NET 任务的 1P API 记录数
  - `onet_task_pct`：该任务在总量中的占比
  - `onet_task_collaboration_count`：同时具备该任务与协作模式的记录数
  - `onet_task_collaboration_pct`：该任务基数内该协作模式的占比

- 均值型交叉指标（API 数据特有）:
  - `prompt_tokens_index`：重新定基的平均提示 tokens（1.0 = 所有任务的总体平均）
  - `prompt_tokens_count`：该指标的记录数
  - `completion_tokens_index`：重新定基的平均完成 tokens（1.0 = 所有任务的总体平均）
  - `completion_tokens_count`：该指标的记录数
  - `cost_index`：重新定基的平均成本（1.0 = 所有任务的总体平均）
  - `cost_count`：该指标的记录数

- 请求指标:
  - `request_count`：处于该请求类别的 1P API 记录数
  - `request_pct`：该类别在总量中的占比
  - `request_collaboration_count`：同时属于该请求类别与协作模式的记录数
  - `request_collaboration_pct`：该请求基数内该协作模式的占比

## 外部数据来源

我们使用外部经济与人口数据来富化 Claude 使用数据。

### ISO 国家代码

- ISO 3166 国家代码

用于表示国家和地区的国际标准代码，用于将基于 IP 的地理定位数据映射到标准化的国家标识符。

- 标准：ISO 3166-1
- 来源：GeoNames 地理数据库
- URL：https://download.geonames.org/export/dump/countryInfo.txt
- 许可：Creative Commons Attribution 4.0 License（https://creativecommons.org/licenses/by/4.0/）
- 下载日期：2025 年 9 月 2 日
- 输出文件：
  - `geonames_countryInfo.txt`（原始 GeoNames 数据，位于 data/input/）
  - `iso_country_codes.csv`（处理后的国家代码，包含部分修改，位于 data/intermediate/）
- 关键字段：
  - `iso_alpha_2`：两字母国家代码（如 "US"、"GB"、"FR"）
  - `iso_alpha_3`：三字母国家代码（如 "USA"、"GBR"、"FRA"）
  - `country_name`：GeoNames 中的国家名称
- 用途：将基于 IP 的国家识别映射到标准化的 ISO 代码，以实现一致的地理聚合

### 美国州代码

- 州 FIPS 代码与 USPS 缩写

涵盖所有美国各州、领地及哥伦比亚特区的官方州与领地代码，包括 FIPS 代码与两字母 USPS 缩写。

- 系列：State FIPS Codes
- 来源：美国人口普查局（U.S. Census Bureau），地理司（Geography Division）
- URL：https://www2.census.gov/geo/docs/reference/state.txt
- 许可：公共领域（美国政府作品）
- 下载日期：2025 年 9 月 2 日
- 输出文件：
  - `census_state_codes.txt`（原始管道分隔文本文件，位于 data/input/）
- 用途：将州名称映射为两字母缩写（例如 "California" → "CA"）

### 人口数据

#### 美国各州人口

- 州特征估计——年龄与性别——平民人口

按单岁年龄、性别、种族和西语裔来源给出的各州及哥伦比亚特区的年度平民人口估计。

- 系列：SC-EST2024-AGESEX-CIV
- 来源：美国人口普查局，人口司
- URL：https://www2.census.gov/programs-surveys/popest/datasets/2020-2024/state/asrh/sc-est2024-agesex-civ.csv
- 许可：公共领域（美国政府作品）
- 下载日期：2025 年 9 月 2 日
- 输出文件：
  - `sc-est2024-agesex-civ.csv`（原始普查数据，位于 data/input/）
  - `working_age_pop_2024_us_state.csv`（处理后数据，按州汇总 15–64 岁人口，位于 data/intermediate/）
- 文档：https://www2.census.gov/programs-surveys/popest/technical-documentation/file-layouts/2020-2024/SC-EST2024-AGESEX-CIV.pdf

#### 各国人口

- 15–64 岁人口，总量

15 至 64 岁之间的总人口。人口基于实际居住定义，统计所有居民，不论法律身份或国籍。

- 系列：SP.POP.1564.TO
- 来源：联合国《世界人口展望》，发布方：联合国人口司；世界银行工作人员估计
- URL：https://api.worldbank.org/v2/country/all/indicator/SP.POP.1564.TO?format=json&date=2024&per_page=1000
- 许可：Creative Commons Attribution 4.0 License（https://creativecommons.org/licenses/by/4.0/）
- 下载日期：2025 年 9 月 2 日
- 输出文件：
  - `working_age_pop_2024_country_raw.csv`（世界银行原始数据，位于 data/input/）
  - `working_age_pop_2024_country.csv`（处理后的国家级数据，包含台湾，位于 data/intermediate/）

#### 台湾人口

- 按单岁人口

台湾（中华民国）的按单岁人口预测。该数据补充了不含台湾的世界银行国家数据。

- 系列：按单岁人口（中位方案，总人口）
- 来源：国家发展委员会，人口推计（台湾）
- URL：https://pop-proj.ndc.gov.tw/main_en/Custom_Detail_Statistics_Search.aspx?n=175&_Query=258170a1-1394-49fe-8d21-dc80562b72fb&page=1&PageSize=10&ToggleType=
- 许可：政府资料开放授权条款（台湾）
- 更新日期：2025.06.17
- 下载日期：2025 年 9 月 2 日
- 参考年份：2024
- 脚本中的变量名：`df_taiwan`（原始数据），并入 `df_working_age_pop_country`
- 输出文件：
  - `Population by single age _20250802235608.csv`（原始数据，位于 data/input/，已预先筛至 15–64 岁）
  - 已合并至 `working_age_pop_2024_country.csv`（处理后的国家级数据，位于 data/intermediate/）

## GDP 数据

### 各国 GDP

- 国内生产总值，现价（十亿美元）

各国与地区的按当期市场价格计的国内生产总值总量。

- 系列：NGDPD
- 来源：国际货币基金组织（IMF），世界经济展望数据库
- URL：https://www.imf.org/external/datamapper/api/v1/NGDPD
- 许可：IMF 数据条款与条件
- 参考年份：2024
- 下载日期：2025 年 9 月 2 日
- 输出文件：
  - `imf_gdp_raw_2024.json`（API 原始响应，位于 data/input/）
  - `gdp_2024_country.csv`（处理后的国家 GDP 数据，位于 data/intermediate/）

### 美国各州 GDP

- SASUMMARY 州年度汇总统计：个人收入、GDP、消费者支出、价格指数与就业

各州按百万现价美元计的国内生产总值。

- 系列：SASUMMARY（各州国内生产总值）
- 来源：美国经济分析局（BEA）
- URL：https://apps.bea.gov/itable/?ReqID=70&step=1
- 许可：公共领域（美国政府作品）
- 下载日期：2025 年 9 月 2 日
- 参考年份：2024
- 输出文件：
  - `bea_us_state_gdp_2024.csv`（原始数据，位于 data/input/，从 BEA 手动下载）
  - `gdp_2024_us_state.csv`（处理后的各州 GDP 数据，位于 data/intermediate/）
- 引用：U.S. Bureau of Economic Analysis, "SASUMMARY State annual summary statistics: personal income, GDP, consumer spending, price indexes, and employment"（访问于 2025 年 9 月 2 日）

## SOC 与 O*NET 数据

### O*NET 任务陈述

- O*NET 任务陈述数据集

包含与 O*NET-SOC 分类相关的职业任务陈述，为每个职业提供详细的工作活动。

- 数据库版本：O*NET Database 20.1
- 来源：美国劳工部 O*NET 资源中心
- URL：https://www.onetcenter.org/dl_files/database/db_20_1_excel/Task%20Statements.xlsx
- 许可：公共领域（美国政府作品）
- 下载日期：2025 年 9 月 2 日
- 输出文件：
  - `onet_task_statements_raw.xlsx`（原始 Excel 文件，位于 data/input/）
  - `onet_task_statements.csv`（处理后数据，包含 soc_major_group，位于 data/intermediate/）
- 关键字段：
  - `O*NET-SOC Code`：完整职业代码（如 "11-1011.00"）
  - `Title`：职业标题
  - `Task ID`：任务唯一标识
  - `Task`：工作任务描述
  - `Task Type`：核心或补充
  - `soc_major_group`：SOC 大类的两位代码（如 "11" 代表管理类）
- 备注：
  - 从 O*NET-SOC 代码中提取 SOC 大类代码用于聚合
  - 用于将 Claude 使用模式映射到职业类别

### SOC 结构

- 标准职业分类（SOC）结构

职业的分层分类体系，提供标准化的职业标题与代码。

- SOC 版本：2019
- 来源：O*NET 资源中心（SOC 分类体系）
- URL：https://www.onetcenter.org/taxonomy/2019/structure/?fmt=csv
- 许可：公共领域（美国政府作品）
- 下载日期：2025 年 9 月 2 日
- 脚本中的变量名：`df_soc`（SOC 结构数据框）
- 输出文件：
  - `soc_structure_raw.csv`（原始数据，位于 data/input/）
  - `soc_structure.csv`（处理后的 SOC 结构，位于 data/intermediate/）
- 关键字段：
  - `Major Group`：SOC 大类代码（如 "11-0000"）
  - `Minor Group`：SOC 小类代码
  - `Broad Occupation`：宽职业代码
  - `Detailed Occupation`：细职业代码
  - `soc_major_group`：两位大类代码（如 "11"）
  - `SOC or O*NET-SOC 2019 Title`：职业（组）标题
- 备注：
  - 提供职业分类的层级结构

### 企业趋势与展望调查

核心问题，全国。

- 来源：美国人口普查局
- URL：https://www.census.gov/hfp/btos/downloads/National.xlsx
- 许可：公共领域（美国政府作品）
- 下载日期：2025 年 9 月 5 日
- 参考期：截至 2023 年 9 月与 2025 年 8 月
- 输入文件：`BTOS_National.xlsx`
