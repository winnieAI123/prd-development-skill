# PRD Development Skill

**PRD Development** 是一款专门为大型语言模型（如 Claude / Gemini / Cursor 等的 Agent 模式）打造的产品分析技能包（Agent Skill）。只需提供一个任意产品的官网 URL，AI 就能全自动进行联网检索、深度分析，并逆向输出一份结构化的商业级产品需求文档（PRD Word 格式）。

## ✨ 核心能力

本技能遵循**“宁可留白，绝不编造”**的铁律。所有分析均基于官网原始素材与外部多轮真实联网搜索的交叉验证，确保不产生幻觉。

- 🕸️ **全自动化调研 (`research <URL>`)**：自动爬取目标官网页面，并启动至少 3 轮递进式的联网扩展搜索（覆盖竞品、商业模式、技术栈等盲区）。
- 🧠 **结构化降维分析 (`analyze`)**：严格依照 PRD 模板梳理出产品名称、核心功能、定价模型、目标画像、用户评价及 SWOT 矩阵等 10 大核心维度，带来源索引并标注置信度。
- 📄 **专业文档输出 (`generate`)**：调用本地集成的 Python 排版引擎，将分析出的 JSON 数据流一键编排输出成标准排版的 `.docx` Word 文档。

## 📦 目录结构

```text
.
├── SKILL.md                          # 技能核心提示词指令集与工作流引擎入口
├── references/                       # 规则参考库
│   ├── prd-template.md               # 约束 AI 输出 PRD 的标准模板文档
│   └── research-guide.md             # 指导 AI 展开多轮深度调查的检索法则
└── scripts/                          # 本地自动化脚本
    └── generate_prd_docx.py          # 负责将提取内容渲染写入 Word 的 Python 程序
```

## 🚀 如何使用（AI 对话交互）

在宿主 AI 挂载并读取本技能目录后，用户只需输入三个核心流程指令：

1. **第一步 资料搜集**：
   发送 `research https://example.com` 
   *(AI 将自主展开地毯式搜索，直至关键维度收集率达标)*
2. **第二步 归纳分析**：
   发送 `analyze` 
   *(AI 将检索到的事实资料套入 PRD 维度模型，生成 Markdown 草稿供审阅)*
3. **第三步 一键导出**：
   发送 `generate` 
   *(触发底层 Python 脚本，桌面即可获取最终的 Word PRD)*

## ⚠️ 设计规范
所有调用过程中产生的数据文件和最终生成的 Word 文档均约束输出在宿主的固定 Download/Result 目录，确保工作区的持续整洁。
