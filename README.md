# Lecture Tutor Skill

> 输入浓缩课件，输出自学讲义 — 精确到页码和章节的知识点定位，配合完整证明与直觉解读

[![版本](https://img.shields.io/badge/version-1.1.0-blue)](https://github.com/Epiphanythu/lecture-tutor-skill/blob/main/SKILL.md)
[![许可](https://img.shields.io/badge/license-MIT-green)](LICENSE)

---

## 核心特色

- **不是压缩总结，而是加水泡开做教材** — 输出长度达到源文档的 50-70%，每个知识点充分展开
- **三维讲解模式** — 从「定义与定理 → 直观理解 → 讲义定位」三个维度展开
- **完整证明，拒绝省略** — 每个定理逐步推导，不接受"证明略"
- **讲义定位回查（最大特色）** — 每个知识点精确标注 `[文件名] 第X页 [章节/定理号]`，讲解作业时还能关联到课件对应章节，告诉你"这道题考的是课件第几页"
- **Markdown + PDF 双格式输出** — Markdown 方便编辑，PDF 方便打印

## 快速开始

复制给 Claude Code 完成安装：

```
Fetch and follow instructions from:
https://raw.githubusercontent.com/Epiphanythu/lecture-tutor-skill/main/INSTALL.md
```

然后直接使用：

```
详细讲解这个课件 /path/to/lecture.pdf
```

讲解作业时推荐同时提供课件，以便讲义定位准确引用课件的章节页码：

```
讲解这份作业 /path/to/作业.pdf，参考课件 /path/to/课件.pdf
```

## 输出

所有文件位于源文件同级的 `DeepDive - <pdf-stem>` 文件夹内：

| 文件 | 说明 |
|------|------|
| `DeepDive - <stem>.md` | Markdown，便于阅读和编辑 |
| `DeepDive - <stem>.tex` | LaTeX 源文件 |
| `DeepDive - <stem>.pdf` | 编译后的 PDF |

- 如果 `xelatex` 不可用，仍会交付 Markdown，并提示 PDF 生成受阻
- 默认文风为中文主写，重要术语保留英文括注

## 适用场景

- 把浓缩课件变成可自学的详细讲义
- 补全课件中省略的推导和解释
- 考前复习，逐题讲解作业
- 生成带有页码引用的教学笔记

## 支持平台

macOS / Linux（Windows 未测试）

## 仓库内容

| 文件 | 说明 |
|------|------|
| `SKILL.md` | 技能定义，包含触发条件、工作流、交付要求 |
| `INSTALL.md` | 安装入口与环境检查说明 |
| `LICENSE` | MIT 许可文件 |

## 致谢

本项目的创建受到了 [summarize-slides-skill](https://github.com/Li-Baichuan-James/summarize-slides-skill) 项目的启发。

## 许可

MIT License
