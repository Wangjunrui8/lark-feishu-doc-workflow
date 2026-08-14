# 数据源映射

## 目录

1. [大前端项目信息共享表（多维表格）](#1-大前端项目信息共享表)
2. [飞书任务](#2-飞书任务)
3. [飞书消息](#3-飞书消息)
4. [本周创建的文档](#4-本周创建的文档)
5. [大前端项目信息收集表（多维表格）](#5-大前端项目信息收集表)

---

## 1. 大前端项目信息共享表

多维表格，记录大前端团队所有项目的需求与任务跟踪信息。

### 定位信息

- **wiki URL**: `https://gnl7hh3k42.feishu.cn/wiki/Z63Ewst6SiAtDwkJZqIcdCMvnBg?table=tbl6a70HjOPmq5AS&view=vewu9HlQU8`
- **base_token**: `HZpJbvuLPaEFBXskUi1cyouonLh`
- **table_id**: `tbl6a70HjOPmq5AS`
- **view_id**: `vewu9HlQU8`

### 已知字段

| 字段名 | 字段 ID | 类型 | 用途 |
|--------|---------|------|------|
| 需求名称 | fldaxqIJ1m | text | 需求标题 |
| 状态 | fldJjwpiC3 | select | 当前状态（需求沟通/待排期/已排期/开发中/测试中/待发布/已发布/废弃/暂停/临时） |
| 优先级 | fld08ljyli | select | P0 / P1 / P2 |
| 业务子域 | fldCBfoWVq | select | 需求所属业务域（用于按子域分组展示） |
| 进度 | fldXO40LQ3 | number(progress) | 完成百分比（0~1） |
| 需求Owner | fldRJZTWeG | user(单选) | 需求负责人（第一层筛选条件） |
| 任务执行人 | fldp8FSfnw | user(多选) | 负责开发的人员（第二层筛选条件） |
| 上线时间 | flddOQSEwM | datetime | 已发布需求的上线日期 |
| 最新进展记录 | fldq2mc2Al | text | 最新进展描述 |
| 是否延期 | fld90fZ6tK | formula | 自动计算的延期状态 |
| 任务等级 | fld8yZKqGb | select | 技术难度（1简单/2中等/3困难/4极其困难） |
| 产品 | fldLyzNdB9 | user | 产品负责人 |
| 提测时间 | fldR1ZTLIl | datetime | 计划提测日期 |

> 实际执行时应先 `lark-cli base +field-list` 获取最新字段结构，以上仅为初始参考。

### 提取命令

```bash
# 1. 获取字段结构（首次执行时）
lark-cli base +field-list \
  --base-token HZpJbvuLPaEFBXskUi1cyouonLh \
  --table-id tbl6a70HjOPmq5AS \
  --as user

# 2. 按当前用户筛选"我负责的"记录
lark-cli base +record-list \
  --base-token HZpJbvuLPaEFBXskUi1cyouonLh \
  --table-id tbl6a70HjOPmq5AS \
  --view-id vewu9HlQU8 \
  --as user \
  --format json

# 3. 如记录较多，可按筛选条件拉取
#    例如：筛选"任务执行人"=当前用户 且 "状态"=进行中
#    先配筛选条件到视图，再用 +record-list --view-id
```

### 数据提取逻辑

- 按以下两层规则筛选当前登录用户的条目，按**业务子域**分组
- 提取每条记录的：需求名称、需求状态、进度、任务等级

**第一层 — 需求 Owner（按状态过滤）**：
- 「需求 Owner」为当前登录用户的记录，纳入以下状态：
  - **需求沟通** — 需求阶段
  - **待排期** — 尚未启动
  - **已排期** — 已排期待开发
  - **开发中** — 正在开发
  - **测试中** — 进入测试阶段
  - **待发布** — 测试通过，等待发布上线
  - **暂停** — 已暂停，标注暂停原因（如有）
  - **废弃** — 已废弃，标注废弃原因（如有）
  - **临时** — 临时性需求
  - **已发布** — 仅保留「上线时间」落在**本周**范围内的记录，非本周上线的不纳入周报

**第二层 — 任务执行人（按状态过滤）**：
- 「任务执行人」为当前登录用户的记录，仅保留以下状态：
  - **待排期** — 尚未启动，用于下周计划参考
  - **开发中** — 正在开发，标注进度百分比
  - **测试中** — 进入测试阶段，标注进度百分比
  - **待发布** — 测试通过，等待发布上线
  - **暂停** — 已暂停，标注暂停原因（如有）
  - **废弃** — 已废弃，标注废弃原因（如有）
  - **临时** — 临时性需求
  - **已发布** — 仅保留「上线时间」落在**本周**范围内的记录

不满足以上任一条件的记录一律丢弃。

---

## 2. 飞书任务

用户个人的待办和已完成任务。

### 提取命令

```bash
# 本周已完成任务
lark-cli task +get-my-tasks \
  --complete=true \
  --format json

# 本周未完成任务（待办/进行中）
lark-cli task +get-my-tasks \
  --complete=false \
  --format json

# 按关键词搜索特定任务
lark-cli task +search --query "关键词" --format json
```

### 数据提取逻辑

- **本周工作**：`complete=true` 中本周完成的任务
- **下周计划**：`complete=false` 中截止时间为下周的任务
- 按任务的 `updated_at` 或 `completed_at` 过滤本周范围

---

## 3. 飞书消息

本周在单聊和群聊中的关键沟通、讨论、决策。

### 提取命令

```bash
# 搜索本周与我相关的关键消息
lark-cli im +messages-search \
  --query "关键词" \
  --start "YYYY-MM-DDT00:00:00+08:00" \
  --end "YYYY-MM-DDT23:59:59+08:00" \
  --as user \
  --format json

# 搜索@我的消息
lark-cli im +messages-search \
  --is-at-me \
  --start "YYYY-MM-DDT00:00:00+08:00" \
  --end "YYYY-MM-DDT23:59:59+08:00" \
  --as user \
  --format json

# 搜索特定群聊中的消息
lark-cli im +messages-search \
  --query "关键词" \
  --chat-id oc_xxx \
  --as user \
  --format json
```

### 数据提取逻辑

- 优先搜索@我的消息，这些通常是分配给我的工作或需要我关注的事项
- 搜索关键词建议：项目名称、需求关键词、"评审"、"上线"、"bug"、"修复"等
- 按消息所属的群聊/单聊归类
- 提炼关键讨论结论和决策，归入**本周工作**

---

## 4. 本周创建的文档

主动识别本周由当前用户创建的飞书云文档，经过内容过滤、任务关联和价值提取后，分别归入「本周工作」和「建议」章节。

### 提取命令

```bash
# 搜索本周创建的云文档（--created-by-me 限定我创建，时间窗用 created-since/until）
lark-cli drive +search \
  --query "" \
  --doc-types docx \
  --created-by-me \
  --created-since "YYYY-MM-DDT00:00:00+08:00" \
  --created-until "YYYY-MM-DDT23:59:59+08:00" \
  --as user \
  --format json
```

> 旗标易错点：是 `--doc-types`（非 `--docs-types`）、`--created-since/--created-until`（非 `--start-time/--end-time`），写错会报 unknown flag。
>
> ⚠️ 空查询陷阱：`--query ""`（空关键词）+ 时间窗可能**静默返回 total=0**（实测本周明明有文档也搜不到）。结果为空时不要直接判定"本周无文档"，必须用业务关键词（项目名、"日报"、"方案"、"文档"等）补搜 1~2 次并合并去重；仍为空才可标注「本周未检索到新建文档」。

### 数据处理流程

对搜索到的每篇文档，执行以下三步处理：

**Step 1 — 读取内容并过滤空文档**：
- 使用 `lark-cli docs +fetch --doc-id <token>` 读取文档正文
- **内容不能为空**：跳过纯模板、空白页或无实质内容的文档，只保留有实质内容的文档

**Step 2 — 关联到具体任务**：
- 将每篇文档的标题和关键内容与数据源 1（大前端项目信息共享表）中的记录（需求名称、业务子域）进行匹配
- **能关联到已有任务的文档**：归入对应业务子域的「本周工作」章节，作为该任务的补充说明（如"相关文档：《XX方案》—— 要点摘要"）
- **无法关联到已有任务的文档**：归入「本周工作」下的**「文档沉淀」**分组，作为独立产出展示

**Step 3 — 提取价值要素**：
- 对有实质内容的文档，提取以下关键要素：
  - 文档标题和主题
  - 核心结论、决策、方案要点
  - 重要的技术设计或架构变更
  - 创新性思路或方法论沉淀
- 这些价值要素同时沉淀到**「四、建议」**章节，作为"能力提升建议"和"解决问题带来的价值"两个子板块的数据支撑



---

## 5. 大前端项目信息收集表

> ⚠️ 收集表与数据源 1（信息共享表）实为**同一张多维表格**（base `HZpJbvuLPaEFBXskUi1cyouonLh` / table `tbl6a70HjOPmq5AS`）下的不同视图：收集表对应「财金/国际域」视图 `vewcTeBYqU`（带业务域过滤），数据源 1 对应「全部数据」视图 `vewu9HlQU8`（无过滤）。字段结构完全一致。旧的独立收集表 token（`QW4Yb41d6aG28msz7fNc356hn6e` 等）已失效，勿再使用。

### 定位信息

- **wiki URL**: `https://gnl7hh3k42.feishu.cn/wiki/Z63Ewst6SiAtDwkJZqIcdCMvnBg?table=tbl6a70HjOPmq5AS&view=vewcTeBYqU`
- **base_token**: `HZpJbvuLPaEFBXskUi1cyouonLh`
- **table_id**: `tbl6a70HjOPmq5AS`
- **view_id**: `vewcTeBYqU`（财金/国际域视图）

### 已知字段

与数据源 1 完全一致（同一张表），直接复用数据源 1 的字段列表（需求名称 fldaxqIJ1m、状态 fldJjwpiC3、优先级 fld08ljyli、业务子域 fldCBfoWVq、进度 fldXO40LQ3、需求Owner fldRJZTWeG、任务执行人 fldp8FSfnw、上线时间 flddOQSEwM 等），无需重复 `+field-list`。

### 提取命令

```bash
# 拉取记录（使用「财金/国际域」视图）
lark-cli base +record-list \
  --base-token HZpJbvuLPaEFBXskUi1cyouonLh \
  --table-id tbl6a70HjOPmq5AS \
  --view-id vewcTeBYqU \
  --as user \
  --format json
```

### 数据提取逻辑

筛选规则（单层）：
- 仅保留「需求 Owner」为当前登录用户的记录，**不论需求状态，全部纳入周报**
- **例外**：状态为「已发布」的记录，仅保留「上线时间」落在**本周**范围内的记录，非本周上线的不纳入周报
- 不满足此条件的记录一律丢弃

筛选后按**业务子域**分组。

### 与数据源 1 的合并处理

两个视图属于同一张表，筛选后的记录会大量重叠，需**合并去重**再归入周报：
- 以「需求名称」为去重键，同名记录只保留一份（字段完全一致，无需比较优先级）
- 仅出现在某一视图中的需求，正常按业务子域分组纳入周报
- 合并后的记录统一参与 Step 3 分组整理
