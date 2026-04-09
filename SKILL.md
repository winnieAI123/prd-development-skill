---
name: prd-development
description: >-
  输入产品官网 URL，自动研究、分析并生成专业 PRD Word 文档。Use when someone provides a product URL and wants a comprehensive PRD, competitive analysis, or product teardown. Triggers on product URLs, "PRD", "产品分析", "逆向分析", "product teardown", "产品调研".
---

# PRD Development — 产品逆向 PRD 生成器

输入一个产品官网 URL，通过系统化的信息收集和专业分析，生成一份结构化的 PRD Word 文档。

## 最高优先级规则（铁律）

> **宁可留白，绝不编造。**

1. **信息必须有出处**。每一条事实性描述都必须能追溯到具体的 URL 或搜索结果。
2. **不编造不存在的功能**。如果官网没提到、搜索也找不到，就标注"未找到公开信息"。
3. **区分事实和推测**。事实标注来源 URL；推测必须明确标注 `[推测]`，并说明推测依据。
4. **研究不充分不生成**。`generate` 命令会检查 `research_state.md`，如果关键信息缺失（产品名称、核心功能、目标用户三者任一为空），拒绝生成并提示先补充研究。
5. **多轮搜索优于单次搜索**。不要只搜一次就下结论。先搜大方向，根据结果再细化搜索关键词，反复 2-3 轮直到信息充足。

## 文件输出规范

- **生成的 PRD 文档（.docx）**：统一输出到 `/Users/winnie/clauderesult/` 目录下
- **中间状态文件（research_state.md）**：存放在当前工作目录
- **临时文件（测试、调试用）**：统一放在 `/Users/winnie/clauderesult/`
- **禁止**在用户的项目目录、桌面或其他位置随意创建文件
- 文件命名格式：`{产品名}_prd_{日期}.docx`，例如 `notion_prd_20260327.docx`

---

## 命令注册表

执行命令时，先读取对应的 reference 文件，再按流程执行。

| 命令 | 功能 | 参考文件 |
|------|------|----------|
| `research <URL>` | 爬取官网 + 联网搜索，收集产品信息 | `references/research-guide.md` |
| `analyze` | 结构化分析已收集的信息，生成 PRD 草稿 | `references/prd-template.md` |
| `generate` | 基于分析结果，生成 PRD Word 文档 | 调用 `scripts/generate_prd_docx.py` |
| `help` | 显示命令列表和使用说明 | — |

### 命令执行流程

```
research <URL>  ──→  analyze  ──→  generate
   │                    │              │
   ▼                    ▼              ▼
收集并保存原始信息    结构化分析     生成 .docx
(research_state.md)  (更新 state) (最终文档)
```

---

## 命令详细规范

### `research <URL>`

**目的**：收集目标产品的所有公开可得信息。

**执行前**：读取 `references/research-guide.md`。

**执行流程**：

1. **Phase 1：官网爬取**
   - 使用 `read_url_content` 抓取官网核心页面：
     - 首页（`/`）
     - 关于页面（`/about`, `/about-us`, `/company`）
     - 产品/功能页面（`/features`, `/product`, `/solutions`）
     - 定价页面（`/pricing`, `/plans`）
   - 如果 `read_url_content` 返回内容为空或不完整（JS 重渲染页面），使用 `browser_subagent` 重新访问
   - 提取关键信息：产品名称、Slogan、核心功能列表、定价模型、团队信息

2. **Phase 2：联网搜索扩展**（至少 3 轮搜索）
   - **第 1 轮**：基础搜索
     - `search_web("{产品名} 是什么 产品介绍")`
     - `search_web("{产品名} review 评测")`
     - `search_web("{产品名} pricing 定价")`
   - **第 2 轮**：根据第 1 轮结果细化
     - `search_web("{产品名} vs {竞品A} {竞品B}")` （竞品从第 1 轮结果中识别）
     - `search_web("{产品名} funding 融资 估值")`
     - `search_web("{产品名} target users 用户群体")`
   - **第 3 轮**：补充缺失维度
     - 检查已收集信息，对空白维度进行针对性搜索
     - 如果商业模式不清晰：`search_web("{产品名} business model revenue")`
     - 如果技术栈不明：`search_web("{产品名} tech stack built with")`

3. **Phase 3：保存研究数据**
   - 将所有收集到的信息写入 `research_state.md`
   - 每条信息标注来源 URL
   - 对信息标注置信度：✅ 已确认 / ⚠️ 推测 / ❌ 未找到

**输出**：`research_state.md` 文件 + 研究摘要报告给用户

**完成标准**：以下至少 6 项有 ✅ 标记才算研究充分：
- 产品名称与描述
- 核心功能列表
- 目标用户画像
- 定价模型
- 竞品信息
- 商业模式
- 用户评价/市场反馈
- 团队/公司背景

如果不足 6 项，自动提示用户："当前研究信息不足（X/8），建议先补充以下维度：[列出空白维度]。你可以提供额外信息，或输入 `research <URL>` 补充搜索。"

---

### `analyze`

**目的**：将收集到的原始信息结构化为 PRD 各章节。

**执行前**：
- 读取 `references/prd-template.md`
- 读取 `research_state.md`（如果不存在，提示先执行 `research`）

**执行流程**：

1. 检查 `research_state.md` 完整性（见上方完成标准）
2. 按 PRD 模板 10 大维度逐一分析：
   - 对每个维度，仅使用 `research_state.md` 中已标注来源的信息
   - 信息不足的维度标注"信息不足，建议补充研究"
   - 推测性结论加 `[推测]` 前缀
3. 生成完整的 PRD Markdown 草稿
4. 更新 `research_state.md`，追加分析结果

**输出**：PRD Markdown 草稿展示给用户审阅

**关键规范**：
- 不凭空生成 User Stories 或 Success Metrics — 这些基于你研究到的真实产品行为
- 竞品对比表只列入在搜索中确认存在的竞品
- SWOT 分析中的每一项都要有信息支撑

---

### `generate`

**目的**：将分析结果输出为专业排版的 Word 文档。

**执行前**：
- 检查 `research_state.md` 中是否包含分析结果（如果没有，提示先执行 `analyze`）
- 验证关键字段不为空（产品名称、核心功能、目标用户）

**执行流程**：

1. 从 `research_state.md` 提取结构化 PRD 数据
2. 转换为 JSON 格式
3. 调用 `scripts/generate_prd_docx.py` 生成 Word 文档
   ```bash
   echo '<JSON 数据>' | python <skill-path>/scripts/generate_prd_docx.py --output <输出路径>
   ```
4. 告知用户文件路径

**输出**：`.docx` 文件

**如果关键字段为空**：拒绝生成，提示：
"以下关键信息缺失，无法生成有效的 PRD 文档：[列出缺失的字段]。请先通过 `research` 或手动补充信息后再生成。"

---

### `help`

显示以下内容：

```
PRD Development — 产品逆向 PRD 生成器

命令列表：
  research <URL>    爬取官网 + 联网搜索，收集产品信息
  analyze           结构化分析已收集的信息
  generate          生成 PRD Word 文档
  help              显示此帮助信息

使用流程：
  1. research https://example.com  — 自动收集产品信息
  2. analyze                       — 分析并生成 PRD 草稿
  3. generate                      — 输出 Word 文档

原则：宁可留白，绝不编造。
```

---

## 状态管理：research_state.md

在当前工作目录下创建和维护 `research_state.md`。

### 格式规范

```markdown
# 产品研究状态 — [产品名称]
最后更新：[日期时间]

## 基础信息
- 产品名称：[名称] | 来源：[URL]
- 官网：[URL]
- Slogan：[口号] | 来源：[URL]
- 所属行业：[行业] | 来源：[URL]
- 创立时间：[时间] | 来源：[URL]
- 团队/公司：[信息] | 来源：[URL]

## 研究完成度
| 维度 | 状态 | 来源数量 |
|------|------|----------|
| 产品名称与描述 | ✅/⚠️/❌ | N |
| 核心功能列表 | ✅/⚠️/❌ | N |
| 目标用户画像 | ✅/⚠️/❌ | N |
| 定价模型 | ✅/⚠️/❌ | N |
| 竞品信息 | ✅/⚠️/❌ | N |
| 商业模式 | ✅/⚠️/❌ | N |
| 用户评价/市场反馈 | ✅/⚠️/❌ | N |
| 团队/公司背景 | ✅/⚠️/❌ | N |

## 原始数据

### 官网爬取数据
[按页面分组的提取内容，每条标注来源 URL]

### 搜索结果数据
[按搜索轮次分组的结果，每条标注来源]

## PRD 分析结果
[analyze 命令执行后追加此节]
```

### 写入时机

| 操作 | 写入内容 |
|------|----------|
| `research` 完成 | 基础信息 + 研究完成度 + 原始数据 |
| `analyze` 完成 | 更新完成度 + 追加 PRD 分析结果 |
| 用户手动补充信息 | 更新对应维度的数据和完成度 |

---

## 信息质量控制机制

### 三级置信度标注

| 级别 | 标记 | 含义 | 使用场景 |
|------|------|------|----------|
| 已确认 | ✅ | 有明确来源 URL 支撑 | 官网明确说明的信息 |
| 推测 | ⚠️ | 有间接证据但不确定 | 从描述推断的功能、可能的用户群 |
| 未找到 | ❌ | 搜索后仍无信息 | 搜索无果的维度 |

### 反编造检查清单（analyze 和 generate 阶段执行）

- [ ] 产品功能列表中的每一项是否都能在官网或搜索结果中找到？
- [ ] 定价信息是否来自官网定价页或可靠第三方？
- [ ] 竞品是否在搜索结果中被多次提及为同类产品？
- [ ] 用户评价是否来自真实的评测/评论？
- [ ] SWOT 的每一条是否有对应的事实依据？
- [ ] 所有的 `[推测]` 标注是否都说明了推测依据？

---

## 文件路由

执行命令前，读取以下参考文件：

- **`research`**：读取 `references/research-guide.md`
- **`analyze`**：读取 `references/prd-template.md`
- **`generate`**：读取 `scripts/generate_prd_docx.py` 的使用说明（脚本顶部注释）

---

## 使用示例

### 场景 1：分析一个 SaaS 产品

```
用户：research https://notion.so
Agent：[爬取官网 → 3 轮搜索 → 保存 research_state.md]
      研究完成，7/8 维度已覆盖。技术栈信息不足。

用户：analyze
Agent：[读取 research_state.md → 生成 PRD 草稿]
      PRD 草稿已生成，请审阅。技术栈部分标注为"信息不足"。

用户：generate
Agent：[生成 Word 文档]
      PRD 已生成：./notion_prd_20260327.docx
```

### 场景 2：信息不足时

```
用户：research https://some-obscure-tool.com
Agent：[爬取 + 搜索]
      研究完成，仅 3/8 维度有信息。
      ⚠️ 信息不足，建议补充：定价、竞品、用户评价、商业模式、团队背景。
      你可以手动提供信息，或尝试不同搜索关键词。

用户：generate
Agent：❌ 关键信息缺失（核心功能不完整、目标用户未知），无法生成有效 PRD。
      请先通过 research 或手动补充信息。
```
