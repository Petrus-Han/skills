---
name: docx
description: "全面的文档创建、编辑和分析，支持修订跟踪、批注、格式保留和文本提取。当 Claude 需要处理专业文档（.docx 文件）时使用：(1) 创建新文档，(2) 修改或编辑内容，(3) 处理修订跟踪，(4) 添加批注，或任何其他文档任务"
license: 专有。完整条款见 LICENSE.txt
---

# DOCX 创建、编辑和分析

## 概述

用户可能会要求你创建、编辑或分析 .docx 文件的内容。.docx 文件本质上是一个包含 XML 文件和其他资源的 ZIP 归档，你可以读取或编辑。你有不同的工具和工作流程可用于不同的任务。

## 工作流程决策树

### 读取/分析内容
使用下面的"文本提取"或"原始 XML 访问"部分

### 创建新文档
使用"创建新 Word 文档"工作流程

### 编辑现有文档
- **你自己的文档 + 简单更改**
  使用"基本 OOXML 编辑"工作流程

- **他人的文档**
  使用**"红线标注工作流程"**（推荐默认）

- **法律、学术、商业或政府文档**
  使用**"红线标注工作流程"**（必需）

## 读取和分析内容

### 文本提取
如果你只需要读取文档的文本内容，你应该使用 pandoc 将文档转换为 markdown。Pandoc 对保留文档结构提供了出色的支持，并可以显示修订跟踪：

```bash
# 将文档转换为带有修订跟踪的 markdown
pandoc --track-changes=all path-to-file.docx -o output.md
# 选项：--track-changes=accept/reject/all
```

### 原始 XML 访问
你需要原始 XML 访问来处理：批注、复杂格式、文档结构、嵌入媒体和元数据。对于这些功能，你需要解包文档并读取其原始 XML 内容。

#### 解包文件
`python ooxml/scripts/unpack.py <office_file> <output_directory>`

#### 关键文件结构
* `word/document.xml` - 主文档内容
* `word/comments.xml` - document.xml 中引用的批注
* `word/media/` - 嵌入的图像和媒体文件
* 修订跟踪使用 `<w:ins>`（插入）和 `<w:del>`（删除）标签

## 创建新 Word 文档

从头创建新 Word 文档时，使用 **docx-js**，它允许你使用 JavaScript/TypeScript 创建 Word 文档。

### 工作流程
1. **必须 - 阅读完整文件**：完整阅读 [`docx-js.md`](docx-js.md)（约 500 行）。**阅读此文件时绝不设置任何范围限制。** 在继续创建文档之前阅读完整文件内容以了解详细语法、关键格式规则和最佳实践。
2. 使用 Document、Paragraph、TextRun 组件创建 JavaScript/TypeScript 文件（你可以假设所有依赖项已安装，但如果没有，请参阅下面的依赖项部分）
3. 使用 Packer.toBuffer() 导出为 .docx

## 编辑现有 Word 文档

编辑现有 Word 文档时，使用 **Document 库**（用于 OOXML 操作的 Python 库）。该库自动处理基础设施设置并提供文档操作方法。对于复杂场景，你可以通过该库直接访问底层 DOM。

### 工作流程
1. **必须 - 阅读完整文件**：完整阅读 [`ooxml.md`](ooxml.md)（约 600 行）。**阅读此文件时绝不设置任何范围限制。** 阅读完整文件内容以了解 Document 库 API 和直接编辑文档文件的 XML 模式。
2. 解包文档：`python ooxml/scripts/unpack.py <office_file> <output_directory>`
3. 使用 Document 库创建并运行 Python 脚本（参见 ooxml.md 中的"Document 库"部分）
4. 打包最终文档：`python ooxml/scripts/pack.py <input_directory> <office_file>`

Document 库为常见操作提供高级方法，并为复杂场景提供直接 DOM 访问。

## 文档审阅的红线标注工作流程

此工作流程允许你在实施到 OOXML 之前使用 markdown 规划全面的修订跟踪。**关键**：对于完整的修订跟踪，你必须系统地实施所有更改。

**批处理策略**：将相关更改分组为 3-10 个更改的批次。这使调试可管理同时保持效率。在进入下一批之前测试每个批次。

**原则：最小、精确的编辑**
实施修订跟踪时，只标记实际更改的文本。重复未更改的文本使编辑更难审阅并显得不专业。将替换分解为：[未更改文本] + [删除] + [插入] + [未更改文本]。通过从原始提取 `<w:r>` 元素并重用它来保留原始 run 的 RSID 用于未更改文本。

示例 - 在句子中将"30天"更改为"60天"：
```python
# 不好 - 替换整个句子
'<w:del><w:r><w:delText>期限为30天。</w:delText></w:r></w:del><w:ins><w:r><w:t>期限为60天。</w:t></w:r></w:ins>'

# 好 - 只标记更改的内容，保留原始 <w:r> 用于未更改文本
'<w:r w:rsidR="00AB12CD"><w:t>期限为</w:t></w:r><w:del><w:r><w:delText>30</w:delText></w:r></w:del><w:ins><w:r><w:t>60</w:t></w:r></w:ins><w:r w:rsidR="00AB12CD"><w:t>天。</w:t></w:r>'
```

### 修订跟踪工作流程

1. **获取 markdown 表示**：将文档转换为保留修订跟踪的 markdown：
   ```bash
   pandoc --track-changes=all path-to-file.docx -o current.md
   ```

2. **识别并分组更改**：审查文档并识别所有需要的更改，将它们组织成逻辑批次：

   **位置方法**（用于在 XML 中查找更改）：
   - 章节/标题编号（例如，"第 3.2 节"、"第四条"）
   - 如果有编号的段落标识符
   - 具有唯一周围文本的 Grep 模式
   - 文档结构（例如，"第一段"、"签名块"）
   - **不要使用 markdown 行号** - 它们不映射到 XML 结构

   **批次组织**（每批分组 3-10 个相关更改）：
   - 按章节："批次 1: 第 2 节修正"，"批次 2: 第 5 节更新"
   - 按类型："批次 1: 日期更正"，"批次 2: 当事人名称更改"
   - 按复杂性：从简单文本替换开始，然后处理复杂结构更改
   - 按顺序："批次 1: 第 1-3 页"，"批次 2: 第 4-6 页"

3. **阅读文档并解包**：
   - **必须 - 阅读完整文件**：完整阅读 [`ooxml.md`](ooxml.md)（约 600 行）。**阅读此文件时绝不设置任何范围限制。** 特别注意"Document 库"和"修订跟踪模式"部分。
   - **解包文档**：`python ooxml/scripts/unpack.py <file.docx> <dir>`
   - **记下建议的 RSID**：解包脚本将建议一个用于修订跟踪的 RSID。复制此 RSID 用于步骤 4b。

4. **分批实施更改**：逻辑分组更改（按章节、按类型或按邻近性）并在单个脚本中一起实施。这种方法：
   - 使调试更容易（较小的批次 = 更容易隔离错误）
   - 允许增量进度
   - 保持效率（3-10 个更改的批次大小效果良好）

   **建议的批次分组：**
   - 按文档章节（例如，"第 3 节更改"、"定义"、"终止条款"）
   - 按更改类型（例如，"日期更改"、"当事人名称更新"、"法律术语替换"）
   - 按邻近性（例如，"第 1-3 页的更改"、"文档前半部分的更改"）

   对于每批相关更改：

   **a. 将文本映射到 XML**：在 `word/document.xml` 中 grep 文本以验证文本如何在 `<w:r>` 元素之间分割。

   **b. 创建并运行脚本**：使用 `get_node` 查找节点，实施更改，然后 `doc.save()`。参见 ooxml.md 中的**"Document 库"**部分了解模式。

   **注意**：始终在编写脚本之前立即 grep `word/document.xml` 以获取当前行号并验证文本内容。每次脚本运行后行号都会改变。

5. **打包文档**：所有批次完成后，将解包的目录转换回 .docx：
   ```bash
   python ooxml/scripts/pack.py unpacked reviewed-document.docx
   ```

6. **最终验证**：对完整文档进行全面检查：
   - 将最终文档转换为 markdown：
     ```bash
     pandoc --track-changes=all reviewed-document.docx -o verification.md
     ```
   - 验证所有更改都已正确应用：
     ```bash
     grep "原始短语" verification.md  # 不应找到
     grep "替换短语" verification.md  # 应该找到
     ```
   - 检查没有引入意外更改


## 将文档转换为图像

要可视化分析 Word 文档，使用两步过程将其转换为图像：

1. **将 DOCX 转换为 PDF**：
   ```bash
   soffice --headless --convert-to pdf document.docx
   ```

2. **将 PDF 页面转换为 JPEG 图像**：
   ```bash
   pdftoppm -jpeg -r 150 document.pdf page
   ```
   这会创建 `page-1.jpg`、`page-2.jpg` 等文件。

选项：
- `-r 150`：设置分辨率为 150 DPI（根据质量/大小平衡调整）
- `-jpeg`：输出 JPEG 格式（如果首选 PNG 使用 `-png`）
- `-f N`：要转换的第一页（例如，`-f 2` 从第 2 页开始）
- `-l N`：要转换的最后一页（例如，`-l 5` 停止在第 5 页）
- `page`：输出文件的前缀

特定范围示例：
```bash
pdftoppm -jpeg -r 150 -f 2 -l 5 document.pdf page  # 只转换第 2-5 页
```

## 代码风格指南
**重要**：生成 DOCX 操作代码时：
- 写简洁的代码
- 避免冗长的变量名和冗余操作
- 避免不必要的打印语句

## 依赖项

必需的依赖项（如果不可用请安装）：

- **pandoc**：`sudo apt-get install pandoc`（用于文本提取）
- **docx**：`npm install -g docx`（用于创建新文档）
- **LibreOffice**：`sudo apt-get install libreoffice`（用于 PDF 转换）
- **Poppler**：`sudo apt-get install poppler-utils`（用于 pdftoppm 将 PDF 转换为图像）
- **defusedxml**：`pip install defusedxml`（用于安全 XML 解析）
