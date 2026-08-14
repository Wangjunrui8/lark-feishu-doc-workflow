---
name: daily-report-writer
description: 生成王均睿的工作日报：采集当天 git 提交、Meego 任务动态、飞书聊天记录，按固定三段式格式写成草稿，发飞书私信给本人审阅，确认后追加写入飞书日报文档。当用户提到写日报、生成日报、日报草稿、把日报写入文档，或由定时任务触发的每日日报时使用。
version: 1.0.0
---

# 日报生成工作流（王均睿）

## 固定常量

| 项 | 值 |
| --- | --- |
| 日报文档 docx token | `RCu2dMKtioZKuDxQNuscCd3VnIB`（wiki 节点 `RqhRwa7IQim5LFkcngdcs22Mnjd`，标题「王均睿-日报」） |
| 用户 open_id | `ou_532219c8f688a56f453bd32430ba8c17`（王均睿） |
| bot→用户私信 chat_id | `oc_ee8527676291ea4bd35c89bb70d52156`（已验证可达） |
| Meego 空间 | project_key `678f09205dee7d1ce129d92c`（chintannengtech 数智能源研发中心） |
| Meego 用户写法 | `王均睿<id:7657357438227598296>`（user_key `7657357438227598296`） |
| git 仓库 | `E:/CHITeng/finance-middleware`、`E:/CHITeng/ltc-new`、`E:/CHITeng/anneng-boostrap-ui`，作者过滤 `junrui` |
| lark-cli PATH | 每条命令前 `export PATH="/c/nvm4w/nodejs:$PATH"` |

## 流程 A：生成草稿并发送审阅（定时任务默认流程）

### 1. 取当天时间范围

`date` 取当前本地时间（Asia/Shanghai）。TODAY=`YYYY-MM-DD`，START=`${TODAY}T00:00:00+08:00`，NOW=当前 ISO 时间。

### 2. 采集三路素材（互相独立，可并行）

**git 提交**（三个仓库逐一执行，`--no-merges` 去合并噪音）：

```bash
git -C "E:/CHITeng/<repo>" --no-pager log --all --no-merges --since="${TODAY} 00:00" --author="junrui" --pretty=format:"%s"
```

**Meego 任务动态**（先查登录态）：

```bash
meegle auth status --format json
# authenticated=false → 执行 meegle auth login（会开浏览器，等用户授权）；仍失败则放弃此源并在草稿标注「Meego 未登录」
meegle workitem query --project-key 678f09205dee7d1ce129d92c --mql "SELECT \`work_item_id\`, \`名称\`, \`状态\`, \`更新时间\` FROM \`chintannengtech\`.\`Task\` WHERE \`__任务负责人\` = '王均睿<id:7657357438227598296>' AND \`更新时间\` >= '${TODAY}T00:00:00+08:00' ORDER BY \`更新时间\` DESC LIMIT 30" --format json
```

**飞书聊天**（识别当天跟谁聊、聊的是哪个项目/需求）：

```bash
lark-cli im +chat-list --as user --types=group,p2p --sort=active_time --page-size 20 --jq '.data.chats[] | {name, chat_id, chat_mode}'
```

取前 15 个活跃会话（跳过纯通知群：「数智化研发中心发布通知群」「大前端应用告警」），逐个拉当天消息：

```bash
lark-cli im +chat-messages-list --as user --chat-id <id> --start "${TODAY}T00:00:00+08:00" --end "<NOW>" --order asc --page-all --page-limit 6 --no-reactions --jq '.data.messages[] | {t:.create_time, s:.sender.name, sid:.sender.id, c:.content}'
```

无当天消息的会话直接略过。对每个有消息的会话归纳：群名/对话人 → 关联项目或需求（如 并网申请商务字段、户用应付入账对接、电站管理/完工登记低代码迁移、竞价联系人建档）→ 本人（sender.id == 用户 open_id）参与了什么讨论、有无约定/阻塞。

### 3. 空日判定

三路素材全部为空（0 提交 + 0 任务动态 + 0 聊天消息）→ 大概率周末/节假日，**静默结束，不发消息**。

### 4. 成稿

严格按文档现有格式（三段式，2026 年 8 月起的最新风格，无「总结/思考」；仅用户明确要求时才加第 4 段）：

```markdown
## **2026 - 8 - 13** 

1. **今日工作内容**

   - 条目（任务粒度、动宾短语，如「【财务中台】户用应付入账对接前端先行开发」「新增页面 3 个：（抵扣账单、任务列表、任务详情）前端交互完成」）
2. **遇到的问题与阻塞**

   - 无（有则写明，如「等待后端对齐」）
3. **明日工作计划**

   - 条目（从进行中任务/今日约定推断；确实没有写「无」）
```

风格要求：日期带空格 `2026 - 8 - 13`（无前导零）；条目简洁、与历史日报同一语气；git 提交语译成业务话术（`feat:户用应付入账对接前端界面基础` → 「户用应付入账对接前端界面搭建」），不直接粘 commit message；聊天中的评审会、联调、需求沟通也算工作内容。

### 5. 发送审阅

```bash
lark-cli im +messages-send --as bot --user-id ou_532219c8f688a56f453bd32430ba8c17 --markdown "<草稿全文 + 末尾一行：以上为今日日报草稿，确认无误后在 QoderWork 说『写入日报』，或直接回复修改意见>"
```

**到此为止，不写文档。** 草稿同时存一份到工作区 `daily-report-draft-<TODAY>.md` 备用。

## 流程 B：确认后写入日报文档（用户说「写入日报/确认写入」时）

1. 拉最新文档确认防重与月份小节：

```bash
lark-cli docs +fetch --doc RCu2dMKtioZKuDxQNuscCd3VnIB --doc-format markdown
```

- 已存在 `## **<今天日期>**` → 提示已写过，询问是否覆盖该天（用 str_replace），默认不重复追加。
- 检查当月一级标题（如 `# 八月日报`）是否存在；跨月时先在追加内容开头加 `# X月日报`（中文数字：一~十二月）。

2. 追加（stdin pipe，勿用 @绝对路径）：

```bash
cat daily-report-draft.md | lark-cli docs +update --as user --doc RCu2dMKtioZKuDxQNuscCd3VnIB --command append --doc-format markdown --content -
```

3. 重新 `+fetch` 验证今天的日期标题已在文末。

## 易错点

- 新 shell 里 lark-cli 不在 PATH，必须先 export（见常量表）。
- MQL 三坑：字段是小写 `work_item_id`（写「工作项ID」会报错）；人员条件必须 `姓名<id:user_key>` 完整写法；日期必须 ISO 8601 带时区（`2026-08-12 00:00:00` 这种会报 invalid datetime format）。
- chat-messages-list 不加 `--no-reactions` 会刷大量 reactions warning 且偶发报错。
- 用户偏好轻量：草稿控制在 3~6 条要点，不要罗列每条聊天记录；不要加「总结/思考」除非用户要求。
- 未获用户确认前绝不写日报文档。

## 验证

- 流程 A 成功 = 用户飞书私信收到草稿（bot 发送返回 ok:true）。
- 流程 B 成功 = `+fetch` 结果末尾包含今天的 `## **<日期>**` 小节且层级在当月 H1 之下。
