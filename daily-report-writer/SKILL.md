---
name: daily-report-writer
description: 生成当前用户的工作日报：采集当天 git 提交、Meego（飞书项目）任务动态、飞书聊天记录，按固定三段式格式写成草稿，发飞书私信给本人审阅，确认后追加写入飞书日报文档。当用户提到写日报、生成日报、日报草稿、把日报写入文档，或由定时任务触发的每日日报时使用。
version: 2.0.0
---

# 日报生成工作流

## 个人配置（首次使用前填写）

首次触发本技能时，先与用户确认下表配置并**把确认后的值写回本表**，供后续会话与定时任务直接读取。

| 项 | 值 | 获取方式 |
| --- | --- | --- |
| 日报文档 docx token | `<你的日报文档 token>` | 新建或指定一篇飞书文档作为日报载体，取 URL 中的 token |
| 用户 open_id | `<你的 open_id>` | `lark-cli contact +search-user --query "<你的姓名>" --as user` |
| git 仓库列表 | `<本地仓库路径 1>`、`<本地仓库路径 2>`… | 需要统计提交记录的本地 git 仓库 |
| git 作者过滤词 | `<作者关键词>` | 与 `git log --pretty=%an` 中署名匹配的名字或邮箱前缀 |
| Meego 空间 project_key（可选） | `<MEEGO_PROJECT_KEY>` | 飞书项目（Meego）项目 URL 中的 key；团队不用 Meego 则留空并跳过该数据源 |
| Meego 用户写法（可选） | `<姓名><id:<user_key>>` | Meego 成员信息中的数字 user_key |
| lark-cli PATH（按需） | `<npm 全局 bin 目录>` | shell 找不到 lark-cli 时填写，`npm prefix -g` 定位 |

## 流程 A：生成草稿并发送审阅（定时任务默认流程）

### 1. 取当天时间范围

`date` 取当前本地时间（用户时区）。TODAY=`YYYY-MM-DD`，START=`${TODAY}T00:00:00+<时区偏移>`，NOW=当前 ISO 时间。

### 2. 采集三路素材（互相独立，可并行）

**git 提交**（配置表中的仓库逐一执行，`--no-merges` 去合并噪音）：

```bash
git -C "<仓库路径>" --no-pager log --all --no-merges --since="${TODAY} 00:00" --author="<作者关键词>" --pretty=format:"%s"
```

**Meego 任务动态**（可选，仅配置了 Meego 时执行；先查登录态）：

```bash
meegle auth status --format json
# authenticated=false → 执行 meegle auth login（会开浏览器，等用户授权）；仍失败则放弃此源并在草稿标注「Meego 未登录」
meegle workitem query --project-key <MEEGO_PROJECT_KEY> --mql "SELECT \`work_item_id\`, \`名称\`, \`状态\`, \`更新时间\` FROM \`<空间名>\`.\`Task\` WHERE \`__任务负责人\` = '<姓名><id:<user_key>>' AND \`更新时间\` >= '${TODAY}T00:00:00+<时区偏移>' ORDER BY \`更新时间\` DESC LIMIT 30" --format json
```

**飞书聊天**（识别当天跟谁聊、聊的是哪个项目/需求）：

```bash
lark-cli im +chat-list --as user --types=group,p2p --sort=active_time --page-size 20 --jq '.data.chats[] | {name, chat_id, chat_mode}'
```

取前 15 个活跃会话（跳过纯通知群：发布通知群、告警群等机器人播报类会话），逐个拉当天消息：

```bash
lark-cli im +chat-messages-list --as user --chat-id <id> --start "${TODAY}T00:00:00+<时区偏移>" --end "<NOW>" --order asc --page-all --page-limit 6 --no-reactions --jq '.data.messages[] | {t:.create_time, s:.sender.name, sid:.sender.id, c:.content}'
```

无当天消息的会话直接略过。对每个有消息的会话归纳：群名/对话人 → 关联项目或需求 → 本人（sender.id == 用户 open_id）参与了什么讨论、有无约定/阻塞。

### 3. 空日判定

三路素材全部为空（0 提交 + 0 任务动态 + 0 聊天消息）→ 大概率周末/节假日，**静默结束，不发消息**。

### 4. 成稿

严格按日报文档现有格式（默认三段式，无「总结/思考」；仅用户明确要求时才加第 4 段）。若日报文档里已有历史条目，先 `+fetch` 看一眼既往格式再动笔，保持同一语气：

```markdown
## **2026 - 8 - 13** 

1. **今日工作内容**

   - 条目（任务粒度、动宾短语，如「【XX项目】YY需求前端开发」「新增页面 3 个：（页面A、页面B、页面C）前端交互完成」）
2. **遇到的问题与阻塞**

   - 无（有则写明，如「等待后端对齐」）
3. **明日工作计划**

   - 条目（从进行中任务/今日约定推断；确实没有写「无」）
```

风格要求：日期标题样式沿用文档既有写法（如 `2026 - 8 - 13`，数字间带空格、无前导零）；条目简洁、与历史日报同一语气；git 提交语译成业务话术（`feat:xxx对接前端界面基础` → 「xxx对接前端界面搭建」），不直接粘 commit message；聊天中的评审会、联调、需求沟通也算工作内容。

### 5. 发送审阅

```bash
lark-cli im +messages-send --as bot --user-id <你的 open_id> --markdown "<草稿全文 + 末尾一行：以上为今日日报草稿，确认无误后在 QoderWork 说『写入日报』，或直接回复修改意见>"
```

**到此为止，不写文档。** 草稿同时存一份到工作区 `daily-report-draft-<TODAY>.md` 备用。

> bot 私信前提：所用飞书自建应用需开通 `im:message` 权限并发布过版本；不可用时改为在 QoderWork 会话内直接出示草稿，其余流程不变。

## 流程 B：确认后写入日报文档（用户说「写入日报/确认写入」时）

1. 拉最新文档确认防重与月份小节：

```bash
lark-cli docs +fetch --doc <你的日报文档 token> --doc-format markdown
```

- 已存在 `## **<今天日期>**` → 提示已写过，询问是否覆盖该天（用 str_replace），默认不重复追加。
- 检查当月一级标题（如 `# 八月日报`）是否存在；跨月时先在追加内容开头加 `# X月日报`（中文数字：一~十二月）。文档无此月份组织结构时，直接按既有结构追加。

2. 追加（stdin pipe，勿用 @绝对路径）：

```bash
cat daily-report-draft.md | lark-cli docs +update --as user --doc <你的日报文档 token> --command append --doc-format markdown --content -
```

3. 重新 `+fetch` 验证今天的日期标题已在文末。

## 易错点

- 新 shell 里 lark-cli 可能不在 PATH，必要时先 export（见个人配置表）。
- MQL 三坑：字段是小写 `work_item_id`（写「工作项ID」会报错）；人员条件必须 `姓名<id:user_key>` 完整写法；日期必须 ISO 8601 带时区（`2026-08-12 00:00:00` 这种会报 invalid datetime format）。
- chat-messages-list 不加 `--no-reactions` 会刷大量 reactions warning 且偶发报错。
- 用户偏好轻量：草稿控制在 3~6 条要点，不要罗列每条聊天记录；不要加「总结/思考」除非用户要求。
- 未获用户确认前绝不写日报文档。
- 若计划用 QoderWork 定时任务每天触发本技能，配置必须已写回本文件（定时任务没有会话上下文，只能读技能内的配置）。

## 验证

- 流程 A 成功 = 用户飞书私信收到草稿（bot 发送返回 ok:true）。
- 流程 B 成功 = `+fetch` 结果末尾包含今天的 `## **<日期>**` 小节且层级符合文档既有结构。
