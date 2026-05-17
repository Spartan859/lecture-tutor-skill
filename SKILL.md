---
version: "1.1.0"
name: lecture-tutor
description: Use when the user wants to generate a detailed explanation document from a lecture PDF, course slides, textbook PDF, or markdown notes. Triggers on requests like 详细讲解, 生成讲解文档, 把这个课件讲清楚, explain this lecture, 生成深度讲解, 帮我把课件变成讲义, or any request to produce a comprehensive teaching document from source material.
---

# Lecture Tutor

读取课件/教材 PDF 或 Markdown，自动生成一份完整的深度讲解文档。不是压缩总结，而是把课件中每一个定义、定理、公式展开讲清楚，用「理论 + 直观 + 定位」三维模式，产出一篇可以独立阅读的学习讲义。

**核心理念：** 输入一份浓缩的课件，输出一份自学友好的深度讲解文档。不是压缩总结，而是加水泡开做教材。

## Non-Negotiables

- **REQUIRED COMPANION SKILL:** use `pdf` for PDF intake and reading. This workflow is not optional.
- **NO OTHER SKILLS:** when this skill applies, use only `lecture-tutor` plus `pdf`. Do not invoke other skills.
- Default deliverables are a **Markdown** file (`.md`) **and** a compiled **PDF** file (`.pdf`). Both are always generated.
- Markdown provides easy reading and editing; PDF provides formatted print-ready output.
- If `xelatex` is unavailable, Markdown is still delivered; report the PDF blocker to the user.
- Default output folder: `DeepDive - <pdf-stem>` inside the same directory as the source PDF.
- All generated files must live inside that output folder only.
- Default writing style is Chinese-first, with important technical terms preserved in English in parentheses.
- Each knowledge point must include source page reference: `[文件名] 第X页 [节号/定理号]`.
- Do not skip any definition, theorem, lemma, or important example from the source material.
- **Read ALL pages of the source PDF.** Never stop after the first page. Always confirm total page count and read every page. Missing pages = missing content = failed deliverable.
- **LaTeX content must be identical in depth to Markdown.** Do NOT write a compressed summary in `.tex` while the `.md` is detailed. Both files must contain the same full proofs, examples, and explanations.
- **Compile in `/tmp`**, only copy `.tex` and `.pdf` back to output folder. No intermediate files (`.aux`, `.log`, `.toc`) should ever appear in the output folder.
- A chat-only explanation is **not** a successful deliverable unless the user explicitly opts out of file creation.

### Depth Requirements (Critical — Do Not Violate)

These rules prevent the most common failure: producing a compressed summary instead of an expanded teaching document.

- **Target content ratio**: output document should be at least 50--70% of the source page count. A 35-page lecture should produce at least 18--25 pages of explanation, not 13. If output is significantly below this ratio, the explanations are too compressed.
- **Every theorem must have a FULL proof**, not just a proof sketch. Write out every step of the derivation. If the source does not provide the proof, supply a complete proof yourself. Never write "by induction" without showing the base case and inductive step explicitly.
- **Every major concept must have a concrete numerical example** (explicit calculation, worked example with actual numbers, etc.). Abstract definitions without examples are not acceptable.
- **Intuitive understanding sections must be at least one full paragraph** (5--8 sentences). Two or three sentences are not enough. Cover: intuitive meaning, analogy, "why was this invented", common misconceptions.
- **Every section/chapter must have a transition paragraph** explaining why this section follows from the previous one and what new question it answers.
- **Every section/chapter must end with a "本节小结"** listing the 3--5 key takeaways.
- **Do NOT compress to save space.** It is better to be slightly verbose than to leave the reader confused. Each concept should be explained as if the reader has never seen it before.

## Skill Boundary

- Allowed skills: `lecture-tutor` and `pdf` only.
- Forbidden: all other skills.
- Task or subagent mechanisms may still be used when the platform supports them.

## Required Outcome

The default successful outcome is:
- a saved Markdown file named `DeepDive - <stem>.md`
- a saved LaTeX source named `DeepDive - <stem>.tex`
- a compiled PDF named `DeepDive - <stem>.pdf`
- all generated files grouped in one folder named `DeepDive - <stem>` inside the same directory as the source PDF
- a final response that reports the artifact paths

The following are **not** successful completions:
- an inline chat explanation only
- a document that skips definitions, theorems, or examples
- claiming completion before the Markdown file is written and saved
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
Write Markdown file
    |
    v
Write LaTeX and compile PDF with xelatex
    |
    v
Final artifact check
    |
    v
Report result
```

## Step 1. Intake

- Confirm source PDF path or Markdown file path.
- **Ask the user for the corresponding lecture/course PDF** if the source is a homework/assignment document. The lecture PDF is needed for accurate page references in the 讲义定位 dimension. If the user does not provide a lecture PDF, note this limitation and only reference the source document's own pages.
- Confirm output directory. Default: `DeepDive - <stem>` inside the same directory as the source file.
- Determine scope: whole document or specific chapters. Default: whole document.
- Determine output format: Markdown + PDF (default). If `xelatex` is missing, Markdown only and report the blocker.
- Record any style instructions from the user (e.g., application domain focus, language preference).
- Create the output folder as the first filesystem step.

## Step 2. Read the Source Globally

- Use `pdf` skill to read the entire document enough to understand macro-structure.
- Identify chapter boundaries, section titles, definition blocks, theorem blocks, example blocks.
- Build a segmentation map: which pages belong to which chapter/section.
- For very long documents (100+ pages), segmentation is mandatory.

## Step 3. Segment the Source

- Segment by natural chapter/section boundaries.
- Use the coarsest segmentation that preserves topic integrity.
- Each segment should be self-contained enough for an independent reader to process.

## Step 4. Segment Reader Phase

For each segment, the reader must extract (reference Step 3 for segmentation):

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

## Step 5. Generate Three-Dimensional Explanations

For every knowledge point extracted in Step 4, produce a three-dimensional explanation:

### Dimension 1: Definition & Theorem (定义与定理)

- Quote the exact mathematical definition/theorem from the source.
- Include the theorem statement precisely and completely.
- Reference: `[文件名] 第X页 [定义/定理Y]`
- If there are equivalent definitions or conditions, list them all.
- **Every theorem/proposition must be followed by its COMPLETE proof** (not just a sketch). Show every algebraic step. If the source omits the proof, supply one. Write "$\qed$" at the end.
- After each definition, provide a concrete example verifying it (e.g., verify that a specific instance satisfies the definition with actual values).

### Dimension 2: Intuitive Understanding (直观理解)

This is the most important dimension for learning. It must be substantial, not token.

- **Minimum length**: at least one full paragraph (5--8 sentences). Do NOT write only 2--3 sentences.
- Plain language explanation of what the definition means.
- Intuitive interpretation when applicable (e.g., use analogies and everyday language to explain abstract concepts).
- Answer: "Why does this concept exist? What problem does it solve?"
- Address common misconceptions explicitly (e.g., point out typical errors and show the correct form).
- Include a concrete numerical example (e.g., a worked calculation showing the concept in action).
- Use physical or everyday analogies when they genuinely help.

### Dimension 3: Source Location (讲义定位)

- `[文件名] 第X页 [节号]`
- Cross-reference related concepts from earlier/later pages.

## Step 6. Build Narrative Flow

The document must read like a textbook, not a list of facts:

- Start each chapter/section with a **transition paragraph** (2--4 sentences) explaining: what question was left unresolved by the previous section, and how this section addresses it. Example: "在上一节中，我们学习了概念 A 在特殊条件下的性质。但现实中大多数情况并不满足这个条件。那么，对于一般情况，我们能得到什么结论？本节引入的概念 B 给出了答案..."
- Show logical chains explicitly: "Definition A enables Theorem B, which we then use to prove Lemma C."
- When a concept depends on earlier material, briefly recap (2--3 sentences) before continuing. Never assume the reader remembers.
- End each chapter/section with a **本节小结** (numbered list of 3--5 key takeaways from this section).

## Step 7. Verifier Phase

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

## Step 8. Write Output Files

### 8a. Markdown Output

Document structure:
```markdown
# 深度讲解: <source title>

## <Chapter 1 Title>

### 概述

(transition paragraph + what this section covers)

### <Knowledge Point 1>

#### 定义与定理

(precise statement + full proof + example)

> [文件名] 第X页 [定义/定理Y]

#### 直观理解

(5-8 sentences, analogy, misconceptions, numerical example)

#### 讲义定位

[文件名] 第X页 [节号]

(cross-references to related concepts)

### <Knowledge Point 2>
...

### 本节小结

1. key takeaway 1
2. key takeaway 2
3. key takeaway 3

## <Chapter 2 Title>
...

## 全讲总结

(overall summary at the end)
```

Markdown rules:
- Use standard Markdown with LaTeX math (`$...$` inline, `$$...$$` display).
- All mathematical expressions must use proper LaTeX math notation for rendering in tools like Typora, VS Code, Obsidian.
- Use `>` blockquotes for source page references.
- Use `---` horizontal rules to separate major sections.
- Keep the document self-contained — no external image dependencies.

### 8b. LaTeX/PDF Output

After writing the Markdown, also generate a LaTeX file and compile to PDF:

1. Check that `xelatex` is available. If not, skip PDF generation and report the blocker.
2. Convert the Markdown content to LaTeX using `ctexart` document class.
3. Only use packages from this whitelist: `amsmath`, `amssymb`, `amsthm`, `geometry`, `graphicx`. Do NOT use `xcolor`, `tcolorbox`, `enumitem`, `hyperref`, or any other package.
4. Compile in `/tmp` to avoid intermediate files polluting the output folder, then copy only `.tex` and `.pdf` back:
   ```bash
   cp "<output-folder>/DeepDive - <stem>.tex" /tmp/ && cd /tmp && xelatex "DeepDive - <stem>.tex" && xelatex "DeepDive - <stem>.tex" && cp "DeepDive - <stem>.pdf" "<output-folder>/"
   ```
   Intermediate files (`.aux`, `.log`, `.toc`) remain in `/tmp` and are cleaned by the OS automatically.
5. Confirm both `.tex` and `.pdf` exist in the output folder.

## Step 9. Final Check

- Confirm the Markdown file exists and is saved in the output folder.
- If `xelatex` is available, confirm `.tex` and `.pdf` both exist and compile succeeded.
- Check for: broken math syntax, missing page references, incomplete sections.

## Output Naming

- output folder: `DeepDive - <stem>` inside source PDF directory
- Markdown file: `DeepDive - <stem>.md`
- LaTeX source: `DeepDive - <stem>.tex`
- compiled PDF: `DeepDive - <stem>.pdf`

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
- claiming completion before the Markdown file is written and saved
- claiming completion before `.tex` and `.pdf` are both generated (when xelatex is available)
- using LaTeX packages not in the whitelist (causing compilation failure)
- writing LaTeX/PDF without verifying coverage first
