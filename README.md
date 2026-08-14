# QoderWork 飞书办公技能集合

四个面向飞书办公场景的 QoderWork 技能：云文档转换、日报自动生成、开发文档编写、周报自动生成。
配套展示文档（含实战案例与效果图）：https://gnl7hh3k42.feishu.cn/docx/HYL7dX8dJoirzcxgsdncb64nnBc

## 技能一览

| 技能目录 | 名称 | 一句话简介 | 触发词示例 |
|----------|------|-----------|-----------|
| `lark-feishu-doc-workflow` | 飞书文档自动转换与迭代 | 通过 lark-cli 创建和迭代更新飞书云文档，支持 callout、mermaid 白板、引用等原生 XML 排版 | 「把这个文档转成飞书云文档」「更新一下飞书文档」 |
| `daily-report-writer` | 日报自动生成 | 采集当天 git 提交、Meego 任务、飞书聊天，三段式草稿私信审阅确认后写入日报文档 | 「写日报」「生成日报」 |
| `anneng-dev-doc` | 安能前端开发文档编写 | 按团队模板骨架与写作风格，结合 PRD 与代码改动生成飞书开发文档 | 「写开发文档」「根据 PRD 写详设」 |
| `ai-week-report` | AI 智能周报 | 五大数据源采集 + 三项校准检查，生成六段结构周报，飞书私信交付（bot 发送 Markdown 全文） | 「生成周报」「写周报」 |

## 前置条件

| 依赖 | 说明 |
|------|------|
| [QoderWork](https://qoder.com) | 桌面端 AI 助手 |
| [lark-cli](https://www.npmjs.com/package/lark-cli) | 飞书 CLI 工具（`npm install -g lark-cli`），并完成用户态授权 |
| Node.js / Git Bash（Windows） | lark-cli 运行环境与 pipe 语法支持 |
| meegle CLI（仅日报技能） | Meego/飞书项目任务查询 |

## 安装方式

### 方式一：GitHub 安装（推荐）

1. 克隆本仓库：`git clone https://github.com/Wangjunrui8/lark-feishu-doc-workflow.git`
2. 把需要的技能文件夹整个复制到 `~/.qoderwork/skills/` 下（Windows 为 `C:\Users\<用户名>\.qoderwork\skills\`）
3. 新建一个 QoderWork 对话即生效，可同时安装多个技能

```bash
# 示例：一次装齐四个技能
cd lark-feishu-doc-workflow
cp -r lark-feishu-doc-workflow daily-report-writer anneng-dev-doc ai-week-report ~/.qoderwork/skills/
```

### 方式二：.skill 安装包安装

仓库根目录提供打包好的安装包（`lark-feishu-doc-workflow.skill`、`daily-report-writer.skill`、`anneng-dev-doc.skill`、`ai-week-report.skill`）。下载后在 QoderWork 中打开，点击「Save skill」按钮即可安装。

## 个性化说明

技能二 / 三 / 四内置了作者的个人常量（本人 open_id、多维表 token、日报文档 token、本地仓库路径、团队模板等）。复用到其他团队或个人时，请打开对应 `SKILL.md` 替换为自己的信息：技能二的常量集中在开头「固定常量」表格；技能三 / 四的常量内联在正文命令中（多维表 base/table/view token、文档 token、open_id、仓库路径等），全局搜索 `lark-cli` 命令逐处替换即可。

## 仓库结构

```
├── README.md                        本指南
├── *.skill                          四个技能的 QoderWork 安装包
├── lark-feishu-doc-workflow/        技能一源码（SKILL.md）
├── daily-report-writer/             技能二源码（SKILL.md）
├── anneng-dev-doc/                  技能三源码（SKILL.md + template.xml 团队模板）
└── ai-week-report/                  技能四源码（SKILL.md + references/ 数据源与周报模板）
```

## 使用示例

安装后在 QoderWork 中直接用自然语言触发：

- 「帮我把这个技术文档转成飞书云文档」
- 「写一下我今天的日报，写完发我确认」
- 「根据这个 PRD 写开发文档，填到今天建的模板文档里」
- 「生成周报」

## 许可证

MIT License
