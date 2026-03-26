具体位置：直接依赖 claude -p：scripts/run_eval.py:71、scripts/improve_description.py:26
依赖 Claude Code 的项目结构与注入机制（写入 .claude/commands 让其出现在 available_skills）：scripts/run_eval.py:23、scripts/run_eval.py:30、scripts/run_eval.py:53
触发判定硬编码 Claude Code 的 stream-json 事件格式 + tool 名（只认 Skill/Read）：scripts/run_eval.py:73、scripts/run_eval.py:75、scripts/run_eval.py:133、scripts/run_eval.py:137、scripts/run_eval.py:164、scripts/run_eval.py:166
依赖 CLAUDECODE 环境变量语义（为允许嵌套 claude -p）：scripts/run_eval.py:83、scripts/improve_description.py:33
skill 规范硬编码（frontmatter、1024 字符 description 上限等，未必适配别的平台）：scripts/quick_validate.py:42、scripts/quick_validate.py:83、scripts/improve_description.py:132、scripts/improve_description.py:163
.skill 打包格式假设：scripts/package_skill.py:87

命令行工具硬编码 (CLI Invocation)
系统完全依赖本地安装的 claude 命令行工具来执行模型调用和测试：

在 scripts/run_eval.py 中，执行测试的命令被硬编码为 ["claude", "-p", query, "--output-format", "stream-json", ...]。

在 scripts/improve_description.py 中，调用模型优化描述的命令被硬编码为 ["claude", "-p", "--output-format", "text"]。

2. 特定的认证方式与环境变量 (Authentication & Env Vars)
该工具没有使用常规的 API Key 认证，而是直接劫持和依赖本地环境：

scripts/improve_description.py 明确指出它依赖当前会话的 Claude Code 认证，不需要单独的 ANTHROPIC_API_KEY。

为了防止嵌套调用冲突，代码中硬编码了移除名为 CLAUDECODE 的环境变量。

3. Claude 专属的项目目录结构 (Directory Structure)
为了让 Claude 能够识别并触发正在测试的技能，代码依赖于特定的文件路径注册机制：

scripts/run_eval.py 中的 find_project_root 函数硬编码了向上寻找 .claude/ 隐藏目录的逻辑。

随后，它将生成的临时技能 Markdown 文件硬编码写入到 .claude/commands/ 目录下。在其他平台（如 OpenAI 或通用开发环境）中，这个机制是无效的。

4. 模型特定的响应流与事件解析 (Stream/Event Parsing)
不同大模型厂商（OpenAI, Gemini, Anthropic）的 Tool Calling（工具调用）数据结构完全不同。该代码针对 Claude 的输出结构进行了硬编码：

在 scripts/run_eval.py 中，代码专门解析 Claude 特有的 JSON 事件流结构，如查找 type 为 stream_event 或 content_block_start 的字段。

工具调用的识别硬编码了 Claude 默认提供的工具名称（如判断工具名是否为 "Skill" 或 "Read"）。

5. 提示词中的系统限制与平台背景 (System Prompts)
提示词（Prompt）为 Claude 进行了深度定制：

scripts/improve_description.py 的提示词中写死了一句话：“You are optimizing a skill description for a Claude Code skill...”。

提示词中还硬编码了 Claude 对技能描述长度的硬性限制：“There is a hard limit of 1024 characters”。

SKILL.md 的说明文档也明确指出了 Claude 的 available_skills 触发机制。

6. 专属的打包格式 (Packaging)
scripts/package_skill.py 将输出强制打包为 .skill 后缀的文件压缩包，这是 Claude Code 识别外部技能的特定格式。
