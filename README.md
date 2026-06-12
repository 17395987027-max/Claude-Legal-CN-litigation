# 诉讼律师插件（中国化版本）

面向中国诉讼律师的 Claude Code 插件。管理案件组合 — 收案、大事记、证据保全、索赔函、庭前准备、案件状态汇总。已针对中国大陆法律体系本地化。

**本插件基于 [anthropics/claude-for-legal](https://github.com/anthropics/claude-for-legal) 官方 `litigation-legal` 插件改造。** 每个输出均为供执业律师审阅的草稿，非法律结论。

## 中国化改造

| 改造项 | 原来（美国） | 现在（中国） |
|--------|------------|------------|
| 法律研究 | CourtListener/Trellis/Westlaw | 北大法宝（可用）+ 威科先行（需自建 MCP） |
| 证据制度 | eDiscovery (Everlaw/Relativity) | 本地文件系统 / 中国证据交换 |
| 证据保全 | US legal hold (FRCP) | 《民事诉讼法》第81条 |
| 庭前取证 | deposition (FRCP 30) | 庭前准备 / 证据交换 / 质证 |
| 调解保密 | FRE 408 | 中国调解保密规则 |
| 保密标头 | Attorney Work Product | 律师法保密义务 |
| 会计准则 | ASC 450 / 10-Q / 10-K | 中国企业会计准则 |
| 默认法域 | 美国普通法 | 中国大陆成文法 |
| 已删除 | docket-watcher / Slack / Gmail | — |

## 前提条件

- Claude Code 已安装
- 北大法宝 MCP Token（可选，但强烈推荐）：在 https://mcp.pkulaw.com/console/apps 注册获取
- 威科先行 MCP Server（可选）：需自行基于威科先行 API 构建，接口规范见 `.mcp.json`

无法律研究 MCP 时插件仍可运行，但所有法条/案例引用将标注 `[模型知识 — 待核实]`。

## 谁适用

| 角色 | 主要用途 |
|------|---------|
| **执业律师** | 全部功能 — 收案、案件管理、大事记、庭前准备 |
| **法务负责人** | 案件组合概览、风险分布、状态汇总 |
| **律所合伙人** | 案件简报、外部律师管理 |

## 首次使用：初始化

```
/litigation-legal:cold-start-interview
```

初始化访谈会学习你的诉讼实践并写入配置 profile（存储于 `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md`，不受插件更新影响）。

三个核心模块：
- **风险校准** — 风险偏好、重要性阈值、和解授权
- **案件版图** — 业务背景、争议模式、常见对手、外部律师库
- **文档风格** — 案件备忘录格式、外部律师指令风格、保密惯例

## 命令

| 命令 | 功能 |
|------|------|
| `/litigation-legal:cold-start-interview` | 初始化 — 写入插件配置 |
| `/litigation-legal:matter-intake` | 统一收案 — 创建案件工作区 + 日志条目 |
| `/litigation-legal:portfolio-status` | 案件组合概览 — 风险分布、即将到期、陈旧案件 |
| `/litigation-legal:matter-briefing [slug]` | 单个案件深度简报 — 当前态势、变化、截止日 |
| `/litigation-legal:matter-update [slug]` | 追加案件事件到历史记录 |
| `/litigation-legal:matter-close [slug]` | 归档案件（保留不删除） |
| `/litigation-legal:demand-intake [title]` | 索赔函起草前信息收集 |
| `/litigation-legal:demand-draft [slug]` | 起草索赔函 — 保密审查 → 输出 .docx |
| `/litigation-legal:demand-received [path]` | 分诊收到的索赔函 — 选项分析、案件交叉检查 |
| `/litigation-legal:subpoena-triage [path]` | 传票/协助调查通知书分诊 — 分类、范围分析 |
| `/litigation-legal:legal-hold [slug] [--issue/--refresh/--release/--status]` | 证据保全 — 签发/刷新/解除/状态报告 |
| `/litigation-legal:chronology [slug]` | 构建/更新大事记 — 从文档提取并标注重要性 |
| `/litigation-legal:oc-status` | 起草每周外部律师状态请求邮件（Markdown） |
| `/litigation-legal:claim-chart` | 权利要求/要素对照表 — 逐项映射证据 |

## Skills 列表（19 个）

| Skill | 用途 |
|-------|------|
| **cold-start-interview** | 初始化配置 — 风险校准、案件版图、文档风格 |
| **matter-intake** | 统一收案 — 识别、冲突检查、风险分类、重要性 |
| **portfolio-status** | 全组合汇总 — 风险分布、截止日、异常标记 |
| **matter-briefing** | 案件深度简报 — 态势、变化、开放问题、风险重评估 |
| **matter-update** | 追加案件事件、刷新日志 |
| **matter-close** | 归档案件 — 记录结果、最终敞口、经验教训 |
| **matter-workspace** | 多客户案件工作区管理 |
| **demand-intake** | 索赔函上下文收集 — 当事人、事实、依据、筹码 |
| **demand-draft** | 起草索赔函 — 保密审查 → .docx → 发送后清单 |
| **demand-received** | 分诊收到的索赔函 — 实质评估、选项分析 |
| **subpoena-triage** | 传票/协助调查通知书分诊 — 分类、异议框架 |
| **legal-hold** | 证据保全 — 签发/刷新/解除，基于《民事诉讼法》第81条 |
| **chronology** | 大事记 — 从文档源提取日期事件，按案件理论标注重要性 |
| **deposition-prep** | 庭前准备提纲 — 证据交换、证人出庭、质证准备 |
| **brief-section-drafter** | 按内部格式起草代理词章节 |
| **claim-chart** | 权利要求/要素对照表 — 逐格引用来源、差距检测 |
| **privilege-log-review** | 特权日志初审 — 基于《民事诉讼证据规定》 |
| **oc-status** | 每周外部律师状态请求邮件草稿 |
| **customize** | 定向调整 plugin 配置 |

## 法律研究 MCP

| 工具 | 数据规模 | MCP 状态 | 获取方式 |
|------|---------|---------|---------|
| **北大法宝** | 500万+法规 / 1.6亿+案例 | ✅ 可用 | https://mcp.pkulaw.com/console/apps |
| **威科先行** | 法规+裁判文书+实务指南 | ⚠️ 需自建 | 基于 API 构建 MCP Server（见 `.mcp.json`） |

**强烈建议至少配置一个法律研究 MCP。** 无 MCP 时所有法条/案例引用标注 `[模型知识 — 待核实]`，需手动核实。配置后引用自动标注来源（`[北大法宝]` 或 `[威科先行]`）。

## 数据组织

```
├── CLAUDE.md                            # 插件配置模板
├── matters/
│   ├── _log.yaml                        # 案件台账
│   └── [案件简称]/
│       ├── matter.md                    # 案件详情 + 案件理论
│       ├── history.md                   # 仅追加的事件日志
│       ├── chronology.md                # 大事记
│       └── legal-hold-v[N].docx         # 证据保全通知
├── demand-letters/                      # 发出索赔函
│   └── [slug]/
│       ├── intake.md
│       ├── draft-v1.docx
│       └── checklist.md
├── inbound/                             # 收到的索赔函/传票
│   └── [slug]/
│       ├── incoming.[ext]
│       ├── triage.md
│       └── response-v1.docx
└── oc-status/                           # 每周外部律师状态请求
    └── [YYYY-MM-DD]/
        ├── _summary.md
        └── [slug].md
```

## 引用标记惯例

三个标记出现在输出中，不是免责声明而是行动项：

- `[CITE: 需引用的具体法条]` — 法律依据占位符，律师填写后发出
- `[待核实: 具体事实]` — 尚未核实的 factual assertion
- `[审核: 具体判断]` — 需执业律师判断的事项

带有未解决标记的草稿不可视为终稿。

## 输出标头

内部文件自动添加：
- 执业律师：`保密 · 律师工作成果 — 受委托律师指导编制`
- 非律师：`研究笔记 — 非法律意见 — 依赖前请咨询执业律师`

对外发送文件不添加此标头。

## 其它说明

- `_log.yaml` 是案件组合状态的唯一可信源
- 案件历史仅追加，发现错误以新条目更正
- 重启 Claude Code 后插件生效

## 许可

Apache-2.0，基于 [anthropics/claude-for-legal](https://github.com/anthropics/claude-for-legal) 改造。

## 安装

```bash
git clone <你的仓库地址>
cd <仓库目录>
# 在 Claude Code 中:
/plugin marketplace add .
/plugin install litigation-legal@<marketplace名>
# 重启 Claude Code
/litigation-legal:cold-start-interview
```
