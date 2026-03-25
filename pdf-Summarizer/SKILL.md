---
name: pdf-Summarizer
description: Invoke for any PDF summarization task: when the user provides a PDF file path and wants its contents read, condensed, and saved — whether as a Word doc, structured notes, or organized report. Covers: "summarize this PDF", "extract key points from PDF", "convert PDF to Word summary", "把PDF整理成Word", "总结PDF内容", "PDF转摘要". The defining signal is a PDF path plus intent to digest/condense the content into an output document. Skip for: PDF-to-Excel table extraction, multi-PDF comparison, image/scan OCR, or converting non-PDF files into PDF.
---

# PDF Summarizer Skill

这个技能用于读取指定的 PDF 文件，提取文本，生成内容摘要，并将最终摘要保存为 `.docx` (Word) 文件。

## 执行步骤

当用户调用此技能并提供 PDF 路径时，请严格按照以下步骤操作：

### 第 1 步：提取 PDF 文本

使用 `Bash` 工具运行脚本，**直接将文本写入临时文件**（不要使用 `>` 重定向，避免 Windows GBK 编码崩溃）：

```
python "C:/Users/fyuanshuang/.claude/skills/pdf-summarizer/pdf_tool.py" extract "<用户提供的PDF路径>" "temp_pdf_content.txt"
```

### 第 2 步：阅读与摘要

使用 `Read` 工具读取 `temp_pdf_content.txt` 的内容，生成一份结构清晰、重点突出的摘要。

#### 摘要章节顺序（必须按此排列）

1. `# 文档标题`
2. `## 基本信息` — 作者、单位、日期等
3. `## 核心创新点 / 主要贡献` — **放在靠前位置**，让读者第一时间看到价值所在
4. `## 研究背景与问题` — 现状、痛点、研究意义
5. `## 研究内容 / 方法` — 各章节/模块的核心方案
6. `## 研究计划` — 若有时间表，用表格格式呈现（见下方格式说明）
7. `## 预期成果`

#### 格式元素说明

| 元素 | 写法 | 渲染效果 |
|------|------|----------|
| 一级标题 | `# 标题` | Word 大标题 |
| 二级标题 | `## 小节` | Word 二级标题 |
| 三级标题 | `### 子节` | Word 三级标题 |
| 无序列表 | `- 要点` | Word 项目符号 |
| 行内粗体 | `**关键词**` | 加粗文字 |
| 表格 | 见下方示例 | **真实 Word 表格**（不是纯文本） |

**表格写法示例**（用于研究计划等）：

```
| 时间 | 内容 |
|------|------|
| 2025.12–2026.2 | 学习前置知识 |
| 2026.3–2026.4 | 攻关核心算法 |
```

> 注意：`|---|---|` 分隔行会被自动过滤，只有内容行会出现在 Word 表格中。

### 第 3 步：保存为 Word 文档

1. 使用 `Write` 工具将摘要文本写入临时文件 `temp_summary.txt`（编码 UTF-8）。
2. 使用 `Bash` 工具将其转换为 `.docx`。输出路径默认与 PDF 同目录，文件名为 `<原文件名>_摘要.docx`；若用户指定了输出路径则使用用户指定的路径：

```
python "C:/Users/fyuanshuang/.claude/skills/pdf-summarizer/pdf_tool.py" save "temp_summary.txt" "<输出的docx路径>"
```

### 第 4 步：清理与汇报

- 删除临时文件 `temp_pdf_content.txt` 和 `temp_summary.txt`。
- 向用户报告任务完成，并告知生成的 Word 文档所在路径。

## 异常处理

- 若提示缺少模块，请提醒用户运行：`pip install pypdf python-docx`
- 若 PDF 路径含中文或空格，**路径必须用英文双引号包裹**
- 若 PDF 加密或无文字层（扫描件），脚本会报错，请告知用户
