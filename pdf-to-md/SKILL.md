---
name: pdf-to-md
description: '将 PDF 课件或教材转换为同名的 Markdown 文件。触发条件：用户提供或当前工作目录下存在一个 PDF 文件，要求将其内容完整改写为 Markdown，包含所有文本、公式和图表信息。'
version: "1.0.0"
license: MIT
---

# PDF 课件/教材 → Markdown 转换器

## 触发条件
当用户要求将某个 PDF（课件 PPT、教材章节等）内容转换为同名 Markdown 文件时使用此技能。

---

## 工作流程

### Step 1：定位 PDF 文件

在工作目录下查找 PDF 文件：

```bash
cd /d <工作目录> && dir /s /b *.pdf
```

确认 PDF 路径后，设：
- `PDF_PATH` = PDF 文件绝对路径
- `BASE_NAME` = PDF 文件名（不含扩展名），即输出 Markdown 的文件名
- `WORK_DIR` = PDF 所在目录
- `OUTPUT_MD` = `WORK_DIR\BASE_NAME.md`

---

### Step 2：渲染 PDF 页面为 PNG 图像

使用 `pdftoppm`（随 MiKTeX 或 poppler 提供）将每一页渲染为高分辨率 PNG：

```bash
cd /d <WORK_DIR> && mkdir -p pdf_images
pdftoppm -png -r 150 "<PDF_PATH>" "pdf_images/page"
```

如果 `pdftoppm` 不可用，尝试安装（Windows PowerShell）：
```powershell
# 使用 Chocolatey 安装 poppler
choco install poppler -y
```

---

### Step 3：提取 PDF 文本（备用参考）

```bash
pdftotext -layout "<PDF_PATH>" "pdf_text_raw.txt"
```

读取 `pdf_text_raw.txt` 作为全文内容参考。注意此文本仅作辅助，关键信息（图表、公式、排版结构）需通过图像读取确认。

---

### Step 4：逐页读取图像内容

对每一页渲染出的 PNG 图像，调用 `read_image` 工具读取并理解其视觉内容：

```
read_image(source = "<WORK_DIR>/pdf_images/page-XX.png")
```

对每页重点关注：
- **标题层级**：确定 H1/H2/H3 结构
- **正文内容**：逐条提取文字、列表、表格
- **数学公式**：识别公式并将手写体/排版体转换为 LaTeX（如 `log₂x` → `\log_2 x`，分数→`\frac{a}{b}`，求和→`\sum_{i=1}^{n}`）
- **图表/示意图**：识别图表类型（流程图、框图、数据图、照片等），在 Markdown 中以图片引用形式保留
- **表格**：转为 Markdown 表格语法
- **页脚/来源标注**：酌情保留或省略

将每页内容按层级记录到草稿中。

---

### Step 5：编写生成脚本

将收集到的所有内容写入一个 Python 生成脚本，脚本逻辑如下：

```python
# -*- coding: utf-8 -*-
import os

work_dir    = r"<绝对工作目录>"
img_dir     = os.path.join(work_dir, "pdf_images")
output_md   = os.path.join(work_dir, "<BASE_NAME>.md")

def img(tag, page):
    """生成 Markdown 图片引用"""
    return f"![{tag}](pdf_images/page-{page:02d}.png)"

def fig(tag, page):
    """生成带标题的图片块"""
    return f"\n\n{img(tag, page)}\n\n**图 {tag}**\n"

lines = []

# ── 封面信息（如有）──
lines.append(f"# {BASE_NAME}（Markdown）\n")
lines.append("> 来源：<PDF_FILENAME>\n")

# ── 逐页追加内容（从 Step 4 的草稿中填入）──
# 每页示例：
lines.append(fig("通信系统一般模型", page_num))
lines.append("正文内容...\n")
lines.append("$$公式 LaTeX 行内模式$$\n")

# ── 写出文件 ──
with open(output_md, "w", encoding="utf-8") as f:
    f.write("".join(lines))

print(f"Done: {output_md}")
```

**重要规则：**
- 所有文本内容必须来源于 PDF，不得推理拓展或自行查找资料补充
- 图表以渲染后的 PNG 引用形式嵌入，不擅自生成新图片
- 公式统一使用 LaTeX 语法（行内 `$$...$$` 或 `\(...\)`）
- 多级标题对应 PPT 中的层级结构（大标题→H1，节标题→H2，小节→H3）
- 表格用标准 Markdown 语法输出

---

### Step 6：验证输出

检查生成结果：

```bash
python -c "import os; p=r'<OUTPUT_MD>'; print(os.path.getsize(p), 'bytes')"
```

确认：
- 文件大小 > 0
- 内容覆盖所有页面（无遗漏）
- 图片引用路径正确（`pdf_images/page-XX.png` 均存在）
- 无乱码

---

### Step 7：注册为工件

调用 `agnes_artifacts__present_artifacts` 注册生成的 Markdown 文件：

```json
{
  "title": "<BASE_NAME>（Markdown）",
  "kind": "document",
  "files": [{"path": "<OUTPUT_MD>", "role": "primary"}]
}
```

---

## 停止条件

- 成功生成并注册 Markdown 文件后停止
- 若 `pdftoppm` 渲染失败且无可用的替代工具，报告错误并告知用户手动转换方案，不继续执行

## 注意事项

- 不要使用 `python -c "..."` 单行命令直接操作大量文本，始终通过 `.py` 脚本文件执行
- 工作目录下的中间文件（`pdf_images/`、`pdf_text_raw.txt`、生成脚本）可在任务完成后保留，便于后续复查
- 若原始 PDF 本身是扫描版（无文本层），则完全依赖 `read_image` 逐页识别，此时需更仔细地提取文字和公式