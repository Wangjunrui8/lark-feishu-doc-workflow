---
name: lark-feishu-doc-workflow
description: >-
  Create and iteratively update Feishu cloud documents via lark-cli. Generates Feishu-native
  XML content (callouts, mermaid whiteboards, citations, tables), uploads to document library
  with --parent-position my_library, and supports overwrite/append/block-level updates.
  Use when the user asks to create a Feishu document, update an existing Feishu doc, write
  content to Feishu, or iterate on a Feishu cloud document (飞书云文档).
version: 1.0.0
---

# 飞书云文档创建与迭代更新工作流

## 关键约束

1. **强制 stdin pipe** — 禁止 `--content @绝对路径`（Windows 下必定失败），一律用 `cat file | lark-cli ... --content -`
2. **PATH 前置** — 每条命令前加 `export PATH="/g/program files/nodejs/node_global:$PATH"`
3. **Shell 要求** — pipe 语法需要 Git Bash；若在 cmd.exe 下执行，改用 `bash -c "cat file | lark-cli ..."`
4. **认证** — 所有命令加 `--as user` 使用用户态身份

## 步骤一：生成本地 XML 内容

用 Write 工具将飞书文档 XML 写入工作区临时文件（如 `doc-content.xml`）。

### XML 格式速查

```xml
<title>文档标题</title>

<h1>一级标题</h1>
<h2>二级标题</h2>
<h3>三级标题</h3>

<p>普通段落，支持 <b>加粗</b>、<i>斜体</i>、<a href="https://example.com">链接</a></p>

<hr/>

<callout emoji="📝" background-color="light-blue" border-color="blue">
  <p>高亮提示框内容</p>
</callout>

<whiteboard type="mermaid">
flowchart LR
    A["节点A"] --> B["节点B"]
    B --> C["节点C"]
</whiteboard>

<cite type="doc" doc-id="目标文档ID"></cite>
<cite type="user" user-id="ou_xxx" user-name="用户名"></cite>

<table>
  <tr><th>列1</th><th>列2</th></tr>
  <tr><td>值1</td><td>值2</td></tr>
</table>
```

callout 可用颜色：`light-blue` / `light-green` / `light-red` / `light-yellow` / `light-purple` / `light-grey`。

### 失败策略

- **XML 标签未闭合 / 嵌套错误** → 写入前检查标签配对；优先使用上述模板结构，避免手动拼接
- **正文含 `&` `<` `>`** → 转义为 `&amp;` `&lt;` `&gt;`
- **写入路径不存在** → 始终使用 QoderWork 工作区临时目录

## 步骤二：创建文档

```bash
export PATH="/g/program files/nodejs/node_global:$PATH"
cat "工作区路径/doc-content.xml" | lark-cli docs +create --as user --parent-position my_library --content -
```

创建成功后，从输出中提取 `doc_id` 和文档 URL。

若不需要放入文档库，省略 `--parent-position my_library` 即可创建到默认位置。

### 失败策略

- **lark-cli 命令找不到** → 确认 PATH 设置正确；仍失败则提示用户检查 lark-cli 安装
- **cmd.exe 不支持 pipe** → 切换为 `bash -c "cat file | lark-cli ..."`
- **认证过期（401 / token expired）** → 提示用户在终端执行 `lark-cli auth login`，不要自动处理认证
- **`my_library` 不存在或无权限** → 先运行 `lark-cli docs +list-folders` 确认可用知识库；不可用时退回到默认位置创建，并告知用户实际位置
- **网络超时 / API 限流** → 等 3-5 秒后重试一次；连续失败则提示用户检查网络
- **创建成功但未返回 doc_id** → 用 `lark-cli docs +list` 按标题搜索获取 doc_id

## 步骤三：验证文档结构

```bash
export PATH="/g/program files/nodejs/node_global:$PATH"
lark-cli docs +fetch --as user --doc "DOC_ID_OR_URL" --scope outline
```

对比返回的目录结构与预期章节，确认无缺失或多余。

可用的 scope 值：

| scope | 用途 |
|-------|------|
| `outline` | 查看目录结构和块 ID（结构验证首选） |
| `full` | 获取完整 XML 内容（迭代前拉取最新内容） |
| `simple` | 获取文档标题等基本信息 |

### 失败策略

- **使用了不存在的 scope 值** → 仅使用 `outline` / `full` / `simple` 三个已验证值
- **返回内容与预期不符** → 列出缺失/多余章节；差异较大时重新 overwrite 而非局部修补
- **doc_id 无效** → 检查格式（支持短 ID 如 `HnkndIO6oogAPZxefsAcmtnxnkf` 和完整 URL）；文档不存在则重新创建

## 步骤四：迭代更新

根据需求选择更新方式：

### 全文覆盖（overwrite）

最常用。修改本地 XML 后整体替换文档内容：

```bash
export PATH="/g/program files/nodejs/node_global:$PATH"
cat "工作区路径/updated-content.xml" | lark-cli docs +update --as user --doc "DOC_ID" --command overwrite --content -
```

### 末尾追加（append）

在文档末尾添加新内容：

```bash
cat "工作区路径/new-section.xml" | lark-cli docs +update --as user --doc "DOC_ID" --command append --content -
```

### 块级操作（精确插入/替换）

需要插入到中间位置时使用。先从 `+fetch --scope outline` 获取目标块 ID：

```bash
lark-cli docs +update --as user --doc "DOC_ID" --command block_insert_after --block-id "TARGET_BLOCK_ID" --content "<p>插入的内容</p>"
```

更新后**必须**重新 `+fetch --scope outline` 验证结构。

### 失败策略

- **overwrite 丢失用户手动修改** → 更新前先 `+fetch --scope full` 拉取最新内容并备份到本地；将用户修改合并后再 overwrite
- **append 位置不对** → append 只能追加到末尾；需在中间插入改用 `block_insert_after`
- **块 ID 失效** → 文档更新后块 ID 可能变化，每次操作前重新 fetch outline 获取最新 ID，不要缓存
- **更新后验证失败** → 若本地有上版 XML 备份，重新 overwrite 恢复；否则告知用户现状并协商修复

## 完整端到端示例

```
任务：创建一份飞书文档，包含标题、两个章节、一个 mermaid 图和一个 callout。
```

**1) 生成 XML：**

```xml
<title>项目迭代报告</title>

<h1>一、背景</h1>
<p>本报告总结了 Q3 迭代成果。</p>

<h1>二、核心改进</h1>
<h2>2.1 架构优化</h2>

<whiteboard type="mermaid">
flowchart LR
    A["旧架构"] --> B["迁移工具"]
    B --> C["新架构"]
</whiteboard>

<callout emoji="✅" background-color="light-green" border-color="green">
  <p>本次迭代共完成 <b>12 项</b> 优化，覆盖率提升至 95%。</p>
</callout>
```

**2) 创建文档：**

```bash
export PATH="/g/program files/nodejs/node_global:$PATH"
cat "workspace/doc.xml" | lark-cli docs +create --as user --parent-position my_library --content -
# → 获取 doc_id: Abc123xyz
```

**3) 验证结构：**

```bash
lark-cli docs +fetch --as user --doc "Abc123xyz" --scope outline
# → 确认：一、背景 / 二、核心改进 / 2.1 架构优化 ✓
```

**4) 迭代更新（新增第三章）：**

拉取最新内容 → 本地修改 XML 添加第三章 → overwrite：

```bash
lark-cli docs +fetch --as user --doc "Abc123xyz" --scope full
# → 备份当前内容，合并新章节后写入 updated.xml
cat "workspace/updated.xml" | lark-cli docs +update --as user --doc "Abc123xyz" --command overwrite --content -
```

**5) 再次验证：**

```bash
lark-cli docs +fetch --as user --doc "Abc123xyz" --scope outline
# → 确认第三章已出现 ✓
```

## Pitfalls 汇总

| 坑点 | 原因 | 正确做法 |
|------|------|----------|
| `--content @C:\path\file.xml` 失败 | Windows 绝对路径不被 lark-cli 正确解析 | 用 `cat file \| lark-cli ... --content -` |
| lark-cli: command not found | Node.js 全局 bin 不在 PATH 中 | 命令前加 `export PATH="/g/program files/nodejs/node_global:$PATH"` |
| pipe 在 cmd.exe 下无输出 | cmd.exe pipe 行为与 bash 不同 | 用 `bash -c "..."` 包裹整条命令 |
| overwrite 后用户修改丢失 | overwrite 替换全部内容 | 先 fetch full 备份，合并后再更新 |
| block_insert_after 找不到块 ID | 文档更新后 ID 变化 | 每次操作前重新 fetch outline |
| callout/whiteboard 不渲染 | XML 标签格式错误 | 严格按速查模板书写，注意属性名用连字符 |
| 认证失败 401 | token 过期 | 提示用户执行 `lark-cli auth login` |
