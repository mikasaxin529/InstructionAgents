# InstructionAgents · 小学语文课件/教案生成器

> 一个通用的 Skill：输入课文名 + 年级 + 课型，自动生成 **.pptx 课件、HTML 互动课件、.docx 教案** 三份可上课的成品。

零外部 LLM API、零 LaTeX、零 pandoc——内容生成与渲染分离，脚本只负责把结构化 JSON 渲染成三种格式。

## ✨ 特性

- **一份输入 → 三份成品**：课件 PPT、互动 HTML、教案 Word
- **四种课型**：精读（分两课时）、识字写字、古诗词、口语交际/习作
- **1-6 年级学段可切换**，字号/页数/内容密度按学段自适应
- **内置 2022 新课标**四大核心素养，教学目标强制标注 competency
- **拼音注音**：HTML 用 ruby 原生注音 + 声调标色；PPT 用 Pillow 预渲染
- **笔顺动画**：HTML 用 HanziWriter；PPT 用田字格 PNG
- **课件结构**借鉴部编版真实课件的栏目设计（精读两课时栏目不同、低段页数更多等）

## 📐 架构

```
课文名 + 年级 + 课型
        │
        ▼  (用语文教学专业知识生成)
   课程 JSON（{meta, slides, lessonPlan, handout}）
        │
        ├── render_pptx.py ──► .pptx 课件（按课时分文件）
        ├── render_html.py ──► .html 互动课件（翻页/点读/笔顺）
        └── render_docx.py ──► .docx 教案（8模块+分层作业）
```

- **内容生成层**：读参考文档（schema/课型/学段/课标）→ 产出结构化 JSON
- **渲染层**：Python 脚本只负责把 JSON 渲染成三种格式，对内容来源无感知
- **共享契约层**：`common/` 提供 schema 校验、设计 token、字体、拼音处理

## 📂 目录结构

```
yuwen/
├── SKILL.md                      # Skill 入口：触发词 + 五步工作流
├── requirements.txt              # python-pptx / python-docx / Jinja2 / Pillow
├── references/                   # 内容生成契约（生成 JSON 时必读）
│   ├── schema.md                 # JSON schema 中文契约
│   ├── lesson-types.md           # 四课型栏目序列（借鉴真实课件）
│   ├── stages.md                 # 学段约束（页数/识字量/字号阶梯）
│   ├── curriculum.md             # 2022 新课标四素养摘编
│   └── examples/                 # 两个完整示例
│       ├── zuojing-guantian.json # 《坐井观天》精读 27页(2课时)
│       └── jingyesi.json         # 《静夜思》古诗词 9页
├── scripts/
│   ├── render_all.py             # 总入口：校验+依赖+三渲染器
│   ├── check_deps.py             # 依赖自检 + 中文安装提示
│   ├── render_pptx.py            # JSON → PPTX 布局引擎
│   ├── render_html.py            # JSON → 互动 HTML（Jinja2）
│   ├── render_docx.py            # JSON → 教案 DOCX
│   └── common/                   # 共享契约层
│       ├── schema.py             # 校验 + 缺省补齐 + 枚举
│       ├── design_tokens.py      # 暖色板 / 字号阶梯 / EMU 辅助
│       ├── fonts.py              # 中文字体探测 fallback
│       └── pinyin.py             # 声调拆分 / 标色 / ruby 生成
└── assets/
    ├── css/yuwen.css             # 暖色大字卡片样式
    ├── js/interactivity.js       # 翻页/点读/答案显隐/计时器/HanziWriter
    └── templates/slides.html.j2  # Jinja2 HTML 模板
```

## 🚀 使用

### 手动运行渲染器

```bash
# 安装依赖
pip install -r yuwen/requirements.txt

# 用示例 JSON 生成三份成品
python yuwen/scripts/render_all.py yuwen/references/examples/zuojing-guantian.json \
  --out ~/Desktop/语文课件/坐井观天-精读/

# 单独渲染某种格式
python yuwen/scripts/render_pptx.py input.json -o output.pptx
python yuwen/scripts/render_html.py input.json -o output.html
python yuwen/scripts/render_docx.py input.json -o output.docx
```

### 依赖

| 包 | 用途 |
|----|------|
| python-pptx | 生成 .pptx 课件 |
| python-docx | 生成 .docx 教案 |
| Jinja2 | 渲染 HTML 互动课件 |
| Pillow | 预渲染识字卡/田字格/注音行 PNG |

Python ≥ 3.11。无 OpenAI Key、无 LaTeX、无 pandoc。

## 📦 示例成品

`语文课件/坐井观天-精读/` 下是《坐井观天》精读课（2 课时）的完整成品：

| 文件 | 说明 |
|------|------|
| 坐井观天.pptx / .p2.pptx | 第 1/2 课时 PPT 课件 |
| 坐井观天.html / .p2.html | 第 1/2 课时 互动 HTML（浏览器全屏上课） |
| 坐井观天-教案.docx | 完整教案（8 模块 + 分层作业） |

## 📐 技术细节

- **布局引擎**（render_pptx）：classify_slide 布局分类、高度估算、垂直居中、maxY 截断防溢出
- **拼音注音**：声调标色（一声红/二声橙/三声绿/四声蓝/轻声灰），HTML 用 `<ruby>`，PPT 用 Pillow 预渲染 PNG
- **课时分文件**：精读课 2 课时自动拆分输出（`.pptx` + `.p2.pptx`）
- **退出码约定**：0 成功 / 1 异常 / 2 前置缺失（schema 校验失败或依赖缺失）

## 📄 License

MIT
