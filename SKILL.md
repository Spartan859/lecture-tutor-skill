---
version: "1.2.0"
name: lecture-tutor
description: Use when the user wants to generate a detailed explanation document from a lecture PDF, course slides, or textbook PDF. Triggers on requests like 详细讲解, 生成讲解文档, 把这个课件讲清楚, explain this lecture, 生成深度讲解, 帮我把课件变成讲义, or any request to produce a comprehensive teaching document from source material.
---

# Lecture Tutor

读取课件/教材 PDF，自动生成一份完整的深度讲解文档（TeX + PDF）。不是压缩总结，而是把课件中每一个定义、定理、公式展开讲清楚，用「理论 + 直观 + 定位」三维模式，产出一篇可以独立阅读的学习讲义。

**核心理念：** 输入一份浓缩的课件，输出一份自学友好的深度讲解文档。不是压缩总结，而是加水泡开做教材。

## Non-Negotiables

- **REQUIRED COMPANION SKILL:** use `pdf` for PDF intake and reading.
- **NO OTHER SKILLS:** when this skill applies, use only `lecture-tutor` plus `pdf`.
- Default deliverables are a **LaTeX source** file (`.tex`) **and** a compiled **PDF** file (`.pdf`). Both are always generated.
- If `xelatex` is unavailable, report the blocker and stop — there is no fallback format.
- Default output folder: `DeepDive - <pdf-stem>` inside the same directory as the source PDF.
- All generated files must live inside that output folder only.
- Default writing style is Chinese-first, with important technical terms preserved in English in parentheses.
- Each knowledge point must include source page reference: `[文件名] 第X页 [节号/定理号]`.
- Do not skip any definition, theorem, lemma, or important example from the source material.
- **Read ALL pages of the source PDF.** Never stop after the first page.
- **Compile in `/tmp`**, only copy `.tex` and `.pdf` back to output folder. No intermediate files in the output folder.
- A chat-only explanation is **not** a successful deliverable unless the user explicitly opts out of file creation.

## Depth Requirements

- **Target ratio**: output at least 50--70% of source page count. A 35-page lecture → at least 18--25 pages.
- **Every theorem must have a FULL proof** with every algebraic step. No "proof sketch". If source omits the proof, supply one.
- **Every major concept must have a concrete numerical example** with actual calculations.
- **Intuitive understanding**: at least one full paragraph (5+ sentences) covering meaning, analogy, why it exists, and common misconceptions.
- **Every section needs a transition paragraph** (why this section follows the previous).
- **Every section ends with a 本节小结** (3--5 key takeaways).
- Do NOT compress to save space. Explain as if the reader has never seen it before.

## Skill Boundary

- Allowed skills: `lecture-tutor` and `pdf` only.
- Forbidden: all other skills.
- Task or subagent mechanisms may still be used when the platform supports them.

## Required Outcome

The default successful outcome is:
- a saved LaTeX source named `DeepDive - <stem>.tex`
- a compiled PDF named `DeepDive - <stem>.pdf`
- all generated files grouped in one folder named `DeepDive - <stem>` inside the same directory as the source PDF
- a final response that reports the artifact paths

The following are **not** successful completions:
- an inline chat explanation only
- a document that skips definitions, theorems, or examples
- claiming completion before `.tex` and `.pdf` are both generated
- a generic chapter outline instead of full explanations

## When to Use

Use this skill when the user wants any of the following from a PDF lecture:

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
Read and Segment (global read, split by chapter/section)
    |
    v
Extract and Explain (per segment: extract knowledge points, write 3D explanations)
    |
    v
Assemble and Verify (merge into draft, check coverage)
    |
    v
Write TeX and compile PDF
    |
    v
Final check and report
```

## Step 1. Intake

- Confirm source PDF path.
- **Ask the user for the corresponding lecture/course PDF** if the source is a homework/assignment document. The lecture PDF is needed for accurate page references. If the user does not provide one, note this limitation and only reference the source document's own pages.
- Confirm output directory. Default: `DeepDive - <stem>` inside the same directory as the source file.
- Determine scope: whole document or specific chapters. Default: whole document.
- If `xelatex` is unavailable, report the blocker immediately.
- Record any style instructions from the user.
- Create the output folder as the first filesystem step.

## Step 2. Read and Segment

- Use `pdf` skill to read the entire document.
- Identify chapter boundaries, section titles, definition blocks, theorem blocks, example blocks.
- Build a segmentation map: which pages belong to which chapter/section.
- Segment by natural chapter/section boundaries. Each segment should be self-contained.
- For very long documents (100+ pages), segmentation is mandatory.

## Step 3. Extract and Explain

For each segment, extract ALL knowledge points and produce a three-dimensional explanation for each one.

Do not skip definitions or theorems. Do not collapse multiple knowledge points into one. Do not skip page references.

### Dimension 1: Definition & Theorem (定义与定理)

- Quote the exact mathematical statement with page reference: `[文件名] 第X页 [定义/定理Y]`.
- List equivalent definitions or conditions when applicable.
- **Every theorem must be followed by its COMPLETE proof** — show every step. If the source omits the proof, supply one. Write `$\qed$` at the end.
- After each definition, provide a concrete example verifying it with actual values.

### Dimension 2: Intuitive Understanding (直观理解)

- Plain language explanation in at least one substantial paragraph (5+ sentences).
- Intuitive interpretation with analogies when applicable.
- Answer: "Why does this concept exist? What problem does it solve?"
- Address common misconceptions explicitly.
- Include a concrete numerical example.

### Dimension 3: Source Location (讲义定位)

- `[文件名] 第X页 [节号]`
- Cross-reference related concepts from earlier/later pages.

### Narrative Flow

- Each section/chapter starts with a **transition paragraph** (2--4 sentences) explaining what question was left unresolved and how this section addresses it.
- When a concept depends on earlier material, briefly recap before continuing.
- Each section/chapter ends with a **本节小结** (3--5 key takeaways).

## Step 4. Assemble and Verify

After assembling the full draft, verify coverage:
- Every definition in the source has a corresponding explanation.
- Every theorem in the source has a corresponding explanation.
- Every important example is covered.
- All three dimensions are present for each knowledge point.
- Page references are accurate.
- No section of the source was silently dropped.

Fix any gaps before proceeding to file output.

## Step 5. Write TeX and Compile PDF

1. Use `ctexart` document class.
2. Only use packages from this whitelist: `amsmath`, `amssymb`, `amsthm`, `geometry`, `graphicx`. Do NOT use `xcolor`, `tcolorbox`, `enumitem`, `hyperref`, or any other package.
3. Document structure:
   ```latex
   \section{<Chapter Title>}
   \subsection{概述}
   % transition paragraph
   \subsection{<Knowledge Point>}
   \subsubsection{定义与定理}
   % statement + full proof + example
   \subsubsection{直观理解}
   % substantial paragraph with analogy, misconceptions, numerical example
   \subsubsection{讲义定位}
   % page reference + cross-references
   \subsection{本节小结}
   % 3-5 key takeaways
   ```
4. Compile in `/tmp` to avoid intermediate files in the output folder:
   ```bash
   cp "<output-folder>/DeepDive - <stem>.tex" /tmp/ && cd /tmp && xelatex "DeepDive - <stem>.tex" && xelatex "DeepDive - <stem>.tex" && cp "DeepDive - <stem>.pdf" "<output-folder>/"
   ```
5. Confirm both `.tex` and `.pdf` exist in the output folder.

## Output Naming

- output folder: `DeepDive - <stem>` inside source PDF directory
- LaTeX source: `DeepDive - <stem>.tex`
- compiled PDF: `DeepDive - <stem>.pdf`

## Common Failure Modes

- skipping definitions or theorems because they seem "obvious"
- producing a generic outline instead of full explanations
- writing "proof sketch" instead of a complete proof
- writing only 2--3 sentences for intuitive understanding (minimum 5 sentences)
- omitting concrete numerical examples for major concepts
- missing transition paragraphs between sections
- missing 本节小结 at the end of each section
- producing output significantly shorter than 50% of source page count
- losing page references during merge
- writing a chat-only explanation without generating files
- scattering files outside the `DeepDive - <stem>` folder
- using LaTeX packages not in the whitelist (causing compilation failure)
- claiming completion before `.tex` and `.pdf` both exist in the output folder
