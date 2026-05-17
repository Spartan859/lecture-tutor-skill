# Lecture Tutor Skill

> 把课件/教材 PDF 变成深度讲解文档的 Agent Skill

---

## 它解决什么问题

大学课件通常是高度浓缩的——一个定义一页，一个定理一页，推导过程压缩到几乎没有。对学生来说，光看课件很难真正理解每个知识点。

`lecture-tutor` 就是解决这个问题的：**输入一份课件 PDF，自动输出一份完整的深度讲解文档**。每个知识点都按「定义与定理 → 直观理解 → 讲义定位」三维模式展开，生成可以独立阅读的自学讲义。

## 与 `summarize-slides` 的区别

| | `summarize-slides` | `lecture-tutor` |
|---|---|---|
| 目标 | 压缩成考前小抄 | 展开成教学讲义 |
| 输出长度 | ~10 页 | 完整详细文档 |
| 覆盖度 | 只保留考点 | 每个定义、定理、例子 |
| 深度 | 要点列表 | 完整三维讲解 |
| 使用场景 | 考前复习 | 从零深入理解 |

## 安装

复制给 Claude Code / 其他支持 skills 的 LLM agent：

```
Fetch and follow instructions from:
https://raw.githubusercontent.com/<your-username>/lecture-tutor-skill/main/INSTALL.md
```

## 快速开始

1. 让 Agent 按 `INSTALL.md` 完成安装与环境检查
2. 然后直接说：

```
详细讲解这个课件 /path/to/lecture.pdf
```

3. 也可以指定范围：

```
只详细讲解第 3 章到第 5 章。
```

## 默认输入输出

- 默认输入为 PDF 或 Markdown 格式的课件
- 默认交付物是 `.tex` + 编译后的 `.pdf`
- 默认输出目录为源文件同级的 `DeepDive - <pdf-stem>`
- 默认文风为中文主写，重要术语保留英文括注
- 每个知识点附带页码引用，便于回查原课件

## 讲解模式

每个知识点按三维模式展开：

### 1. 定义与定理
精确引用课件中的数学定义和定理原文，标注来源页码。

### 2. 直观理解
用通俗语言解释含义、几何直觉、为什么需要这个概念、常见误区。

### 3. 讲义定位
标注 `[文件名] 第X页 [节号]`，方便回查原文。

## 它适合做什么

- 把浓缩课件变成可自学的详细讲义
- 深入理解每个定义和定理的来龙去脉
- 补全课件中省略的推导和解释
- 生成带有页码引用的教学笔记

## 与 `pdf` skill 的关系

- `lecture-tutor` 必须与 `pdf` 配合使用
- `pdf` 负责 PDF 读取、提取、OCR 与页面内容确认
- `lecture-tutor` 负责全局通读、分段提取、三维讲解生成、LaTeX 输出与 PDF 编译

## 仓库内容

- `SKILL.md`：技能的权威说明，包含触发条件、工作流、边界与交付要求
- `INSTALL.md`：安装入口与环境检查说明
- `LICENSE`：MIT 许可文件
- `README.md`：使用说明

## 许可

MIT License
