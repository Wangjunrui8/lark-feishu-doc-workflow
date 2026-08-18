---
name: anneng-dev-doc
description: 按团队开发文档模板骨架与写作风格，结合 PRD 与实际代码改动，编写飞书前端开发文档（开发文档/技术文档/详设）。用 lark-cli 读取 PRD 与范例文档、生成 DocxXML 并写入目标 wiki 文档。当用户提到写开发文档、技术文档、详设文档、提测文档、根据 PRD 写文档、填充开发文档模板时使用。
version: 2.0.0
---

# 前端开发文档编写（团队模板风格）

把 PRD + 实际代码改动，写成团队风格的飞书开发文档，写入用户给的目标 wiki 文档（通常是当天新建的空模板《YYYY-MM-DD-需求名》）。

## 个人配置（首次使用前填写）

首次触发本技能时，先与用户确认下表配置并**把确认后的值写回本表**，供后续会话直接读取。

| 项 | 值 | 说明 |
| --- | --- | --- |
| 用户姓名 / open_id | `<你的姓名>` / `<你的 open_id>` | 基础信息与工作量表中 @ 自己用；open_id 可用 `lark-cli contact +search-user --query "<你的姓名>" --as user` 查 |
| 团队开发文档目录页 | `<目录页 wiki URL>` | 存放历史开发文档的 wiki 目录页，用于步骤 3 学习团队写作风格 |
| 技术栈一句话 | `<框架/状态管理/路由/UI 库及版本>` | 2.1 技术选型的固定描述，如「Vue 2.7 / Vuex / Vue Router 3 / Element UI」 |
| 开发分支命名约定 | `feature/xxx-日期-<后缀>` | 发布准备 5.1 分支小节的写法 |
| 飞书项目空间名 | `<project.feishu.cn 空间名>` | 基础信息表「飞书需求URL」的链接前缀 |

## 工作流（按序执行，用 TodoWrite 跟踪）

1. **拉目标文档**：`+fetch` 目标 wiki，确认是空模板还是已有内容；**已有内容必须先备份**（保存 fetch 结果到工作区）。
2. **拉 PRD**：`+fetch --doc-format markdown`，记录 revision、需求点编号（4.1/5.2 这类）、校验规则表、历史数据兼容等章节。
3. **学风格**：`+fetch` 配置表中的团队开发文档目录页——返回的是 `<sub-page-list>`，从里面挑 1-2 篇**同业务域且已填写完整**的文档再 `+fetch` 其 doc-id 学习。不要照抄目录页本身。
4. **收集技术事实**：优先用当前会话已有的代码分析；缺什么补什么——`git status`/`git diff`、`git branch --show-current`（发布准备要写分支）、关键文件路径与接口路径。工作量数字与用户已有的工时拆分保持一致。
5. **生成 XML**：以 [template.xml](template.xml) 为骨架填充。写 `--content` 前按 CLI 要求执行 `lark-cli skills read lark-doc references/lark-doc-xml.md` 获取最新 XML 规则。
6. **写入**：`+update --command overwrite`（空模板场景）；**文档里已有用户手改内容或重要图片时改用块级操作**（见避坑）。
7. **验证**：`+fetch --scope outline` 核对章节齐全，把文档链接发给用户，并列出留给用户补充的"待补充"项。

## lark-cli 执行要点（Windows + cmd 环境）

- 每条命令用 `bash -c` 包裹；shell 找不到 lark-cli 时先 export PATH（`npm prefix -g` 定位 npm 全局 bin 目录）：
  `bash -c 'export PATH="<npm全局bin目录>:$PATH" && lark-cli ...'`
- 内容一律 stdin pipe：`cat "工作区/doc.xml" | lark-cli docs +update --as user --doc "<ID>" --command overwrite --content -`；**禁止** `--content @绝对路径`。
- 所有命令带 `--as user`。大输出先重定向到工作区文件再 Read。
- fetch 返回的 `document_id` 可直接作为 `--doc`（wiki URL 也行，document_id 更稳）。

## 团队风格要点（从范例文档提炼）

- 保留模板骨架：标题 `YYYY-MM-DD-需求名`、开头两条引言 blockquote、五个一级章节；章节内用 h3/h4。
- **基础信息表**：产品/后端/前端/UI/测试参与人、飞书需求URL（project.feishu.cn 链接，`<a type="url-preview">`）、PRD文档URL（`<cite type="doc">`）、后端技术设计/UI设计稿/测试用例 URL。@人必须先用 `lark-cli contact +search-user --query "名字" --as user` 查 open_id 再写 `<cite type="user" user-id="ou_xxx">`；**查不到就写"待补充"，绝不编造**。用户本人按配置表的姓名与 open_id 填写。
- **2.1 技术选型**：一句话写清配置表中的技术栈（框架/状态管理/路由/UI 库），并说明是否引入新依赖。
- **2.2 技术概设**：总体思路一段（加法原则/配置驱动等）+ mermaid 结构图（`<whiteboard type="mermaid">`，注意 mermaid 里的 `>` 也要转义成 `&gt;` 或改用 `---`）；随后**每个需求点一个 h3 小节**：页面位置 → 调整前/后或字段规则表 → 涉及文件（`<b>` 加粗完整路径）→ 关键代码 `<pre lang="JavaScript" caption="文件：说明"><code>...</code></pre>`（短片段即可）→ 易踩坑点用黄色 callout 标注。
- **风险与待确认**：`<ol><li seq="auto">` 列出（持久化链路、命名约定、依赖后端配置等），这是多数团队 CR 习惯。
- **工作量表**：thead 六列（系统/模块/需求点描述/开发人员/工作量人天/风险与依赖），系统列 rowspan 合并，开发人员用 cite；表前写开发分支与总人日。
- **影响面评估**：保留模板引言 blockquote；按 页面 → 关联中心/模块 → 机制/组件 → 历史/在途数据 分层写 `<ul>`；文中原有 `<sheet sheet-id token>` 用原 token 重新嵌入可保留。
- **发布准备**：5.1 分支（迭代名 + 配置表的分支命名约定）、5.2 菜单（无则写无）、5.3 配置变更表（后端配置项、责任方）。
- 措辞克制、直接给结论；不写空话套话；中文标点。

## 避坑清单

- **overwrite 会清空图片与评论**：模板里的配图多为 internal-api-drive-stream authcode 临时链接，过期后无法重新嵌入；写入前告知用户会丢图，或用块级操作（block_replace/block_insert_after）绕开图片所在章节。sheet 块可用 `<sheet sheet-id="..." token="...">` 原样重嵌（会复制出新实例，空模板场景无影响）。
- XML 转义：只转义**文本内容**里的 `&`→`&amp;`、`<`→`&lt;`、`>`→`&gt;`，标签本身不转义；代码块里的 `&&`、`=>` 要处理。
- 表格用 thead/th + tbody/td；表头格加 `background-color="light-gray"`；合并单元格只在起始格写 rowspan/colspan。
- 目录页 fetch 回来是 `<sub-page-list>`，不是正文；要用子页 doc-id 二次 fetch。
- 认证失败（401）提示用户执行 `lark-cli auth login`，不要自动处理。
- overwrite 成功后必须 `+fetch --scope outline` 验证；块 ID 在 overwrite 后全部失效，后续块级操作要重新 fetch。

## 验证标准

- outline 五个一级章节齐全、需求点小节数量与 PRD 对应。
- 基础信息表可 @ 的人已 @，其余标"待补充"。
- 工作量合计与用户既定工时一致。
- 返回文档 URL 给用户，并说明哪些字段留待补充。
