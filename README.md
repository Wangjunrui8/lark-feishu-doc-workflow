 lark-feishu-doc-workflow

 QoderWork 技能：通过 lark-cli 创建和迭代更新飞书云文档。

 ## 功能简介

 本技能让 QoderWork 能够自动完成飞书云文档的创建与迭代更新工作流，包括：

 - **生成飞书 XML 内容** — 将 Markdown、Word 等格式转换为飞书文档 XML
 - - **创建云文档** — 通过 lark-cli 将内容上传到飞书云盘指定目录
   - - **验证文档结构** — 自动检查目录、章节是否完整
     - - **迭代更新** — 支持全文覆盖、末尾追加、块级精确插入三种更新模式
      
       - ## 前置条件
      
       - | 依赖 | 说明 |
       - |------|------|
       - | [QoderWork](https://qoder.com) | 桌面端 AI 助手 |
       - | [lark-cli](https://www.npmjs.com/package/lark-cli) | 飞书 CLI 工具（`npm install -g lark-cli`） |
       - | Node.js | lark-cli 运行环境 |
       - | Git Bash（Windows） | pipe 语法支持 |
      
       - ## 安装方式
      
       - ### 方式一：下载 .skill 文件安装

       1. 从本仓库下载 `lark-feishu-doc-workflow.skill` 文件
       2. 2. 在 QoderWork 中打开该文件，点击「Save skill」按钮即可安装
         
          3. ### 方式二：手动安装
         
          4. 1. 克隆或下载本仓库
             2. 2. 将 `SKILL.md` 文件复制到 `~/.qoderwork/skills/lark-feishu-doc-workflow/` 目录下
                3. 3. 重启 QoderWork 即可使用
                  
                   4. ## 使用示例
                  
                   5. 安装后，在 QoderWork 中直接使用自然语言即可触发：
                  
                   6. - 「帮我把这个技术文档转成飞书云文档，放到 XX 文件夹下」
                      - - 「把这个 Word 测试报告转成飞书文档」
                        - - 「飞书文档有更新，帮我同步一下」
                          - - 「帮我创建一份飞书项目报告」
                           
                            - ## 工作流
                           
                            - ```text
                              生成 XML 内容 → 创建文档 → 验证结构 → 迭代更新
                              ```

                              ## 文件说明

                              | 文件 | 说明 |
                              |------|------|
                              | `SKILL.md` | 技能定义文件（YAML frontmatter + 工作流指令） |
                              | `lark-feishu-doc-workflow.skill` | 打包好的技能安装包（可直接在 QoderWork 中安装） |
                              | `README.md` | 本说明文件 |

                              ## 许可证

                              MIT License
