---
name: lecture-tutor
description: Use when the user wants to generate a detailed explanation document from a lecture PDF, course slides, textbook PDF, or markdown notes. Triggers on requests like 详细讲解, 生成讲解文档, 把这个课件讲清楚, explain this lecture, 生成深度讲解, 帮我把课件变成讲义, or any request to produce a comprehensive teaching document from source material.
---

# Lecture Tutor

读取课件/教材 PDF 或 Markdown，自动生成一份完整的深度讲解文档。不是压缩总结，而是把课件中每一个定义、定理、公式展开讲清楚，用「理论 + 直观 + 定位」三维模式，产出一篇可以独立阅读的学习讲义。

**核心理念：** 输入一份浓缩的课件，输出一份自学友好的深度讲解文档。不是压缩总结，而是加水泡开做教材。

## Non-Negotiables

- **REQUIRED COMPANION SKILL:** use `pdf` for PDF intake and reading. This workflow is not optional.
- **NO OTHER SKILLS:** when this skill applies, use only `lecture-tutor` plus `pdf`. Do not invoke other skills.
- Default deliverables are `.tex` and compiled `.pdf`. The PDF is the required final artifact unless the user explicitly opts out.
- Default output folder: `DeepDive - <pdf-stem>` inside the same directory as the source PDF.
- All generated files (intermediate text, notes, LaTeX, images) must live inside that output folder only. Never scatter them beside the source PDF.
- Default writing style is Chinese-first, with important technical terms preserved in English in parentheses.
- Each knowledge point must include source page reference: `[文件名] 第X页 [节号/定理号]`.
- Do not skip any definition, theorem, lemma, or important example from the source material.
- A chat-only explanation is **not** a successful deliverable unless the user explicitly opts out of file creation.
- Check that `xelatex` is available before promising compiled PDF output.

## Skill Boundary

- Allowed skills: `lecture-tutor` and `pdf` only.
- Forbidden: all other skills.
- Task or subagent mechanisms may still be used when the platform supports them.

## Required Outcome

The default successful outcome is:
- a saved LaTeX source file named `DeepDive - <stem>.tex`
- a compiled PDF file named `DeepDive - <stem>.pdf`
- all generated files grouped in one folder named `DeepDive - <stem>` inside the same directory as the source PDF
- no intermediate files left outside that output folder
- a final response that reports the artifact paths and any blockers

The following are **not** successful completions:
- an inline chat explanation only
- a document that skips definitions, theorems, or examples
- claiming completion before LaTeX is written and PDF is compiled
- a generic chapter outline instead of full explanations

## When to Use

Use this skill when the user wants any of the following from a PDF or Markdown lecture:

- generate a detailed explanation document
- turn lecture slides into a self-study guide
- produce a comprehensive teaching document from course material
- 详细讲解这个课件 / 生成讲解文档
- 把这个课件变成讲义
- 把课件展开讲清楚
- 生成深度讲解
- 帮我把这些slides做成教学笔记

Do not use this skill for:
- homework solving
- essay writing

## Execution Flow

```
Intake (confirm source, output dir, scope)
    |
    v
Check Build Environment (xelatex)
    |
    v
Read PDF Globally (understand macro-structure)
    |
    v
Segment by chapter/section/topic
    |
    v
Dispatch segment readers (one per independent segment)
    |
    v
For each segment: extract ALL knowledge points
    |
    v
For each knowledge point: write three-dimensional explanation
    |
    v
Merge into complete teaching document draft
    |
    v
Dispatch coverage verifier (check no knowledge point was skipped)
    |
    v
Verifier approved?
    |-- NO -> fix gaps, re-verify
    |-- YES -> continue
    |
    v
Write LaTeX
    |
    v
Build PDF with xelatex
    |
    v
Final artifact check
    |
    v
Report result
```

## Step 1. Intake

- Confirm source PDF path or Markdown file path.
- Confirm output directory. Default: `DeepDive - <stem>` inside the same directory as the source file.
- Determine scope: whole document or specific chapters. Default: whole document.
- Determine output format: `.tex` + `.pdf`. Default: both.
- Record any style instructions from the user (e.g., application domain focus, language preference).
- Create the output folder as the first filesystem step.

## Step 2. Check Build Environment

- Verify `xelatex` exists before promising compiled PDF.
- If `xelatex` is missing, report the blocker. Do not silently fall back to `.tex` only.

## Step 3. Read the Source Globally

- Use `pdf` skill to read the entire document enough to understand macro-structure.
- Identify chapter boundaries, section titles, definition blocks, theorem blocks, example blocks.
- Build a segmentation map: which pages belong to which chapter/section.
- For very long documents (100+ pages), segmentation is mandatory.

## Step 4. Segment the Source

- Segment by natural chapter/section boundaries.
- Use the coarsest segmentation that preserves topic integrity.
- Each segment should be self-contained enough for an independent reader to process.

## Step 5. Segment Reader Phase

For each segment, the reader must extract:

1. **segment id** and **page range**
2. **all definitions** (verbatim, with page reference)
3. **all theorems, lemmas, corollaries** (statement + page reference)
4. **all proofs** (or proof sketches if the source provides them)
5. **all important examples** (with page reference)
6. **all formulas** that are standalone results
7. **key terminology** (Chinese + English)
8. **prerequisite concepts** from earlier segments
9. **logical dependencies**: which theorems depend on which definitions
10. **coverage concerns**: anything unclear or potentially missing

Segment readers are NOT allowed to:
- skip definitions or theorems
- collapse multiple knowledge points into one
- skip page references
- decide the final document structure

## Step 6. Generate Three-Dimensional Explanations

For every knowledge point extracted in Step 5, produce a three-dimensional explanation:

### Dimension 1: Definition & Theorem (定义与定理)

- Quote the exact mathematical definition/theorem from the source.
- Include the theorem statement precisely.
- Reference: `[文件名] 第X页 [定义/定理Y]`
- If there are equivalent definitions, list them.

### Dimension 2: Intuitive Understanding (直观理解)

- Plain language explanation of what the definition means.
- Geometric interpretation when applicable.
- Answer: "Why does this concept exist? What problem does it solve?"
- Address common misconceptions.
- Use concrete numerical examples when they help understanding.

### Dimension 3: Source Location (讲义定位)

- `[文件名] 第X页 [节号]`
- Cross-reference related concepts from earlier/later pages.

## Step 7. Build Narrative Flow

The document must read like a textbook, not a list of facts:

- Start each chapter/section with a brief overview: "This section covers X, Y, Z and why they matter."
- Show logical chains: "Definition A enables Theorem B, which we then use to prove Lemma C."
- When a concept depends on earlier material, briefly recap before continuing.
- End each chapter/section with a brief summary of what was covered.

## Step 8. Verifier Phase

After the full document draft is assembled, run a coverage verifier:

The verifier must check:
- every definition in the source PDF has a corresponding explanation in the draft
- every theorem in the source PDF has a corresponding explanation in the draft
- every important example in the source PDF is covered
- all three dimensions are present for each knowledge point
- page references are accurate
- bilingual terminology is consistent
- the narrative flows logically (no orphaned concepts)
- no section of the source was silently dropped

The verifier must return:
- `verdict`: `APPROVED`, `APPROVED_WITH_NOTES`, or `REJECTED`
- `coverage findings`
- `missing or weak areas`

If `REJECTED` or `APPROVED_WITH_NOTES`, fix gaps and re-verify.

## Step 9. LaTeX Output

Document structure:
```
\documentclass[12pt,a4paper]{article}
% Clean preamble with CTeX for Chinese support
% xelatex compilation

\begin{document}
\title{深度讲解: <source title>}
\maketitle
\tableofcontents

\section{<Chapter 1 Title>}
  \subsection{概述}
  \subsection{<Knowledge Point 1>}
    定义与定理 | 直观理解 | 讲义定位
  \subsection{<Knowledge Point 2>}
    ...
  \subsection{本节小结}

\section{<Chapter 2 Title>}
  ...

\end{document}
```

LaTeX rules:
- Single-column, readable layout.
- Use `amsmath` for all math.
- Use `ctex` or `xeCJK` for Chinese text.
- Use structured subsections for each knowledge point.
- Use a consistent visual template for the three dimensions (e.g., colored boxes or bold headers).
- Keep figure code reproducible with `tikz` if figures are needed.

## Step 10. Build and Final Check

- Compile with `xelatex`.
- Confirm both `.tex` and `.pdf` exist.
- Confirm compile succeeded with zero exit status.
- Check for: broken math, missing Chinese rendering, unreadable spacing, missing page references.

## Output Naming

- output folder: `DeepDive - <stem>` inside source PDF directory
- LaTeX source: `DeepDive - <stem>.tex`
- final PDF: `DeepDive - <stem>.pdf`
- intermediate files: keep inside the output folder with deterministic names

## Common Failure Modes

- skipping definitions or theorems because they seem "obvious"
- producing a generic outline instead of full explanations
- missing the intuitive understanding dimension
- losing page references during merge
- writing a chat-only explanation without generating the document
- scattering files outside the `DeepDive - <stem>` folder
- claiming completion before `.tex` and `.pdf` both exist
- compiling without verifying coverage first
