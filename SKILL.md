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

### Depth Requirements (Critical — Do Not Violate)

These rules prevent the most common failure: producing a compressed summary instead of an expanded teaching document.

- **Target page ratio**: output PDF should be at least 50--70% of the source PDF page count. A 35-page lecture should produce at least 18--25 pages of explanation, not 13. If output is significantly below this ratio, the explanations are too compressed.
- **Every theorem must have a FULL proof**, not just a proof sketch. Write out every step of the derivation. If the source does not provide the proof, supply a complete proof yourself. Never write "by induction" without showing the base case and inductive step explicitly.
- **Every major concept must have a concrete numerical example** (2×2 or 3×3 matrix calculation, explicit vector computation, etc.). Abstract definitions without examples are not acceptable.
- **Intuitive understanding sections must be at least one full paragraph** (5--8 sentences). Two or three sentences are not enough. Cover: geometric meaning, physical analogy, "why was this invented", common misconceptions.
- **Every section/chapter must have a transition paragraph** explaining why this section follows from the previous one and what new question it answers.
- **Every section/chapter must end with a "本节小结"** listing the 3--5 key takeaways.
- **Do NOT compress to save space.** It is better to be slightly verbose than to leave the reader confused. Each concept should be explained as if the reader has never seen it before.

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
4. **all proofs** (complete proofs with every step — never just "proof sketch" or "by induction")
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
- Include the theorem statement precisely and completely.
- Reference: `[文件名] 第X页 [定义/定理Y]`
- If there are equivalent definitions or conditions, list them all.
- **Every theorem/proposition must be followed by its COMPLETE proof** (not just a sketch). Show every algebraic step. If the source omits the proof, supply one. Write "$\qed$" at the end.
- After each definition, provide a concrete example verifying it (e.g., "验证 $A$ 是 Hermite 矩阵" with actual matrix entries).

### Dimension 2: Intuitive Understanding (直观理解)

This is the most important dimension for learning. It must be substantial, not token.

- **Minimum length**: at least one full paragraph (5--8 sentences). Do NOT write only 2--3 sentences.
- Plain language explanation of what the definition means.
- Geometric interpretation when applicable (e.g., "自伴算子就是对称的拉伸", "幂零变换是逐层消灭信息").
- Answer: "Why does this concept exist? What problem does it solve?"
- Address common misconceptions explicitly (e.g., "$(kT)^* = kT^*$ is WRONG, the correct form is $\bar{k}T^*$").
- Include a concrete numerical example (e.g., a 2×2 or 3×3 matrix calculation showing the concept in action).
- Use physical or everyday analogies when they genuinely help.

### Dimension 3: Source Location (讲义定位)

- `[文件名] 第X页 [节号]`
- Cross-reference related concepts from earlier/later pages.

## Step 7. Build Narrative Flow

The document must read like a textbook, not a list of facts:

- Start each chapter/section with a **transition paragraph** (2--4 sentences) explaining: what question was left unresolved by the previous section, and how this section addresses it. Example: "在学习谱定理时，我们知道正规矩阵可以被酉对角化。但现实中大多数矩阵不是正规的，不能被酉对角化。那么，对于一般的矩阵，我们能做到什么程度？Schur 分解给出的答案是..."
- Show logical chains explicitly: "Definition A enables Theorem B, which we then use to prove Lemma C."
- When a concept depends on earlier material, briefly recap (2--3 sentences) before continuing. Never assume the reader remembers.
- End each chapter/section with a **本节小结** (numbered list of 3--5 key takeaways from this section).

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
\documentclass[12pt,a4paper]{ctexart}
\usepackage{amsmath,amssymb,amsthm}
\usepackage{geometry}
\usepackage{graphicx}

\geometry{margin=2.5cm}

% Only these packages. Do NOT use xcolor, tcolorbox, enumitem,
% hyperref, or other packages that may not be available in
% TeX Live basic installations. ctexart is the safe choice for
% Chinese support with xelatex.

\begin{document}
\title{深度讲解: <source title>}
\maketitle
\tableofcontents

\section{<Chapter 1 Title>}
  \subsection{概述}  % transition paragraph + what this section covers
  \subsection{<Knowledge Point 1>}
    \subsubsection*{定义与定理}  % precise statement + full proof + example
    \subsubsection*{直观理解}   % 5-8 sentences, analogy, misconceptions
    \subsubsection*{讲义定位}   % [文件名] 第X页
  \subsection{<Knowledge Point 2>}
    ...
  \subsection{本节小结}  % 3-5 numbered key takeaways

\section{<Chapter 2 Title>}
  ...

\section*{全讲总结}  % overall summary at the end
\end{document}
```

LaTeX rules:
- **Use `ctexart` as document class** — this is the safest way to get Chinese support without extra packages.
- **Only use packages from this whitelist**: `amsmath`, `amssymb`, `amsthm`, `geometry`, `graphicx`. Do NOT use `xcolor`, `tcolorbox`, `enumitem`, `hyperref`, or any other package that may not be installed.
- Single-column, readable layout.
- Use `\subsubsection*{}` for the three dimensions (定义与定理, 直观理解, 讲义定位) to keep the structure clean without extra packages.
- Keep figure code reproducible with `tikz` if figures are needed (only if tikz is available).

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
- writing "proof sketch" instead of a complete proof with every step
- writing only 2--3 sentences for intuitive understanding (minimum is one full paragraph)
- omitting concrete numerical examples for major concepts
- missing transition paragraphs between sections
- missing 本节小结 at the end of each section
- producing output significantly shorter than 50% of source page count
- losing page references during merge
- writing a chat-only explanation without generating the document
- scattering files outside the `DeepDive - <stem>` folder
- claiming completion before `.tex` and `.pdf` both exist
- using LaTeX packages not in the whitelist (causing compilation failure)
- compiling without verifying coverage first
