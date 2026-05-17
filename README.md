# Lecture Tutor Skill

> 把课件/教材 PDF 变成深度讲解文档的 Claude Code Agent Skill

[![版本](https://img.shields.io/badge/version-1.1.0-blue)](https://github.com/Epiphanythu/lecture-tutor-skill/blob/main/SKILL.md)
[![许可](https://img.shields.io/badge/license-MIT-green)](LICENSE)

---

## 它解决什么问题

大学课件通常是高度浓缩的——一个定义一页，一个定理一页，推导过程压缩到几乎没有。对学生来说，光看课件很难真正理解每个知识点。做作业时遇到不会的题，也不知道该回去看课件的哪一页。

`lecture-tutor` 解决的就是这个问题：**输入一份课件/作业 PDF，自动输出一份完整的深度讲解文档**。

### 核心特色

- **不是压缩总结，而是加水泡开做教材** — 输出长度达到源文档的 50-70%，每个知识点都充分展开
- **三维讲解模式** — 每个知识点从「定义与定理 → 直观理解 → 讲义定位」三个维度展开，既严谨又易懂
- **完整证明，拒绝省略** — 每个定理都有逐步推导的完整证明，不接受"证明略"
- **具体数值例子** — 每个重要概念配 2×2 或 3×3 矩阵等实际计算，不只是抽象定义
- **讲义定位回查** — 标注 `[文件名] 第X页 [节号]`，方便回查原课件，建立知识点与源文档的精确对应
- **Markdown + PDF 双格式输出** — Markdown 方便编辑和阅读，PDF 方便打印和分享

## 致谢

本项目的创建受到了 [summarize-slides-skill](https://github.com/Li-Baichuan-James/summarize-slides-skill) 项目的启发，在项目结构、安装流程和 companion skill 机制上参考了该项目的设计。

## 安装

### 前置要求

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI 工具
- `pdf` skill（安装过程会自动检测和配置）
- LaTeX 环境（安装过程会自动检测和安装）

### 安装命令

复制给 Claude Code：

```
Fetch and follow instructions from:
https://raw.githubusercontent.com/Epiphanythu/lecture-tutor-skill/main/INSTALL.md
```

安装脚本会自动完成：
1. 下载 `lecture-tutor` skill 到 `~/.claude/skills/lecture-tutor/`
2. 检测 `pdf` companion skill 是否可用
3. 检测并安装 LaTeX 环境（macOS 安装 basictex + 中文支持，Linux 安装 texlive-xetex）
4. 验证所有组件就绪

### 支持的平台

| 平台 | 状态 |
|------|------|
| macOS (Intel / Apple Silicon) | 完全支持 |
| Linux (Debian/Ubuntu) | 完全支持 |
| Linux (RHEL/CentOS/Fedora) | 完全支持 |
| Windows | 未测试 |

## 更新

查看当前版本：打开 `~/.claude/skills/lecture-tutor/SKILL.md`，文件头部的 `version` 字段即为当前版本号。

检查是否有新版本：访问 [SKILL.md](https://github.com/Epiphanythu/lecture-tutor-skill/blob/main/SKILL.md) 查看最新版本号。

更新方式：重新运行安装命令即可覆盖旧文件。

```
Fetch and follow instructions from:
https://raw.githubusercontent.com/Epiphanythu/lecture-tutor-skill/main/INSTALL.md
```

## 快速开始

### 讲解课件

```
详细讲解这个课件 /path/to/lecture.pdf
```

### 讲解作业（推荐同时提供课件）

```
讲解这份作业 /path/to/作业.pdf，参考课件 /path/to/课件.pdf
```

这样才能在「讲义定位」环节准确标注对应课件的章节和页码。如果只提供题目文档而不提供课件，讲义定位将无法生成准确的章节引用。

## 输入输出

### 输入

- PDF 或 Markdown 格式的课件、教材、作业
- **推荐同时提供课件/教材 PDF 和作业/题目 PDF**，以便讲义定位环节能准确引用课件的章节页码
- 如果只提供题目文档（无课件），讲义定位将只标注题目来源页码，无法关联到课件章节

### 输出

所有输出文件位于源文件同级的 `DeepDive - <pdf-stem>` 文件夹内：

| 文件 | 说明 |
|------|------|
| `DeepDive - <stem>.md` | Markdown 格式，便于阅读和编辑 |
| `DeepDive - <stem>.tex` | LaTeX 源文件 |
| `DeepDive - <stem>.pdf` | 编译后的 PDF，排版精美 |

- 如果 `xelatex` 不可用，仍会交付 Markdown 文件，并提示 PDF 生成受阻
- 编译在 `/tmp` 中进行，输出文件夹不会残留中间文件
- 默认文风为中文主写，重要术语保留英文括注

## 讲解模式

每个知识点按三维模式展开：

### 1. 定义与定理
精确引用课件中的数学定义和定理原文，给出完整证明（逐步推导，不接受"证明略"），标注来源页码。

### 2. 直观理解
用通俗语言解释含义、几何直觉、为什么需要这个概念、常见误区，并配具体数值例子（如 2×2 或 3×3 矩阵计算）。

### 3. 讲义定位
标注 `[文件名] 第X页 [节号]`，方便回查原文，交叉引用相关概念。

## 质量保证

为确保输出质量，skill 内置以下硬性规则：

- **读全部页面**：必须读取源文档的所有页面，不得遗漏
- **完整证明**：每个定理都有完整证明，不接受"证明略"或"由归纳法显然"
- **数值例子**：每个重要概念配具体计算
- **直观理解**：至少一段话（5-8 句），涵盖几何意义、物理类比、常见误区
- **Markdown 和 PDF 同等详细**：LaTeX 版本不得压缩内容
- **章节过渡**：每个章节有过渡段落和本节小结

## 适用场景

- 把浓缩课件变成可自学的详细讲义
- 深入理解每个定义和定理的来龙去脉
- 补全课件中省略的推导和解释
- 生成带有页码引用的教学笔记
- 考前复习，逐题讲解作业

## 技术架构

### 与 `pdf` skill 的关系

- `lecture-tutor` 必须与 `pdf` 配合使用
- `pdf` 负责 PDF 读取、提取、OCR 与页面内容确认
- `lecture-tutor` 负责全局通读、分段提取、三维讲解生成、Markdown/LaTeX 输出

### 工作流程

```
读取源文档（全部页面）
    ↓
分段提取知识点
    ↓
三维模式展开讲解
    ↓
覆盖率验证
    ↓
输出 Markdown + LaTeX/PDF
```

## 仓库内容

| 文件 | 说明 |
|------|------|
| `SKILL.md` | 技能的权威定义，包含触发条件、工作流、边界与交付要求 |
| `INSTALL.md` | 安装入口与环境检查说明 |
| `LICENSE` | MIT 许可文件 |
| `README.md` | 本文件 |

## 许可

MIT License
