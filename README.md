# AEIRD
*Open data from Anthropic Economic Index report*

- 这份《Anthropic Economic Index》报告系统分析了 Claude.ai 平台和企业 API 的全球使用数据，旨在刻画人工智能在不同国家、地区、行业和任务类型中的早期应用格局及演变趋势。报告不仅追踪了过去数月 AI 在教育、科研、编程等领域的使用变化，还首次引入“AI 使用指数”（AUI）衡量各国及美国各州的 AI 采用程度，揭示了高收入经济体与新兴市场在使用规模、任务多样性和人机协作模式上的差异。同时，报告深入解析了企业通过 API 部署 AI 的方式、任务分布、自动化与协作化的比例，以及成本与上下文信息对复杂任务落地的影响。通过开放细分到任务层级的匿名化数据，这份报告为研究 AI 技术的扩散路径、经济影响及数字鸿沟提供了量化依据和全球视角。

- 本仓库包含了从 Anthropic Economic Index report 转载的开源数据文件。所有数据均来自该报告，便于用户查阅和分析相关经济指标。对于高校学生，可以考虑对这份数据进行简单的数据挖掘，快速形成一个简易但又与当下热点紧密结合的小项目。

- 另外，由于某些众所周知的原因以及Anthropic近期(2025-09)的抽风针对行为，中国没有被列入数据中。但是，个人认为这份报告可以提供一个相当有趣的研究视角，可供其他领域或项目借鉴（前提是你有数据）。

- 请注意，本仓库仅作为数据转载与整理，原始内容和解释权归 Anthropic Economic Index report 所有。

- 原文报告链接：[https://www.anthropic.com/research/anthropic-economic-index-september-2025-report](https://www.anthropic.com/research/anthropic-economic-index-september-2025-report)

- 若考虑在您的论文或项目中使用该数据，请按以下bibtex格式引用：
```
@online{appelmccrorytamkin2025geoapi,

author = {Ruth Appel and Peter McCrory and Alex Tamkin and Michael Stern and Miles McCain and Tyler Neylon],

title = {Anthropic Economic Index report: Uneven geographic and enterprise AI adoption},

date = {2025-09-15},

year = {2025},

url = {www.anthropic.com/research/anthropic-economic-index-september-2025-report},

}
```

**注意:** 本人不对以上内容的时效性做保证，如若Anthropic修改相关链接或引用格式等内容，请自行查阅原文，寻找或咨询相关要求。


## 仓库结构
```
AEIRD/
├── README.md                                       项目简介、数据来源与使用说明
├── aei_raw_1p_api_2025-08-04_to_2025-08-11.csv
├── aei_raw_claude_ai_2025-08-04_to_2025-08-11.csv
├── data_documentation.md                           原始英文版数据字典与字段解释
├── data_documentation_zh.md                        中文版数据字典与字段解释
└── report_print.pdf                                报告原文打印页面
```
