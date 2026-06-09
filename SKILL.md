---
version: "2.0.0"
name: lecture-tutor
description: Use when the user wants to generate a detailed explanation document from a lecture PDF, course slides, or textbook PDF. Triggers on requests like 详细讲解, 生成讲解文档, 把这个课件讲清楚, explain this lecture, 生成深度讲解, 帮我把课件变成讲义, or any request to produce a comprehensive teaching document from source material.
---

# Lecture Tutor

读取课件/教材 PDF，自动生成一份完整的深度讲解文档（TeX + PDF）。不是压缩总结，而是把课件中每一个定义、定理、公式展开讲清楚，用「理论 + 直观 + 定位」三维模式，产出一篇可以独立阅读的学习讲义。

**核心理念：** 输入一份浓缩的课件，输出一份自学友好的深度讲解文档。不是压缩总结，而是加水泡开做教材。

## Non-Negotiables

- **REQUIRED COMPANION SKILLS:** use `firecrawl-parse` for PDF-to-markdown text extraction and `pdf` for page-to-image conversion (pdf2image) and page splitting (pypdf).
- **NO OTHER SKILLS:** when this skill applies, use only `lecture-tutor` plus `firecrawl-parse` plus `pdf`.
- **ALL INTERMEDIATE OUTPUT TO DISK:** never hold the entire PDF content in context. Save every intermediate result to `<output-folder>/.work/`.
- **CHUNKED PROCESSING:** process the PDF in segments of at most 10 pages. Load one segment at a time into context.
- Default deliverables are a **LaTeX source** file (`.tex`) **and** a compiled **PDF** file (`.pdf`). Both are always generated.
- If `xelatex` is unavailable, report the blocker and stop — there is no fallback format.
- Default output folder: `DeepDive - <pdf-stem>` inside the same directory as the source PDF.
- All generated files must live inside that output folder only.
- Default writing style is Chinese-first, with important technical terms preserved in English in parentheses.
- Each knowledge point must include source page reference: `[文件名] 第X页 [节号/定理号]`.
- Do not skip any definition, theorem, lemma, or important example from the source material.
- **Process ALL pages of the source PDF.** Never stop after the first chunk. Every page must be covered.
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

### Example Solution Requirements

Every source example must be treated as a worked example, not a summary. Examples are where readers learn how to use the definitions and theorems, so the solution must show the method and the calculation path, not just the final answer.

For each important example from the source, include:

1. **题干给定**: list only the data, assumptions, and target explicitly provided by the source. Do not mix in quantities derived later.
2. **解题目标**: state exactly what must be found, designed, verified, or proved.
3. **解题方法**: explain why the chosen theorem, criterion, or computational method applies.
4. **逐步计算**: show the necessary matrix products, ranks, determinants, characteristic polynomials, equation solving, substitutions, and simplifications.
5. **结果校核**: substitute the result back into the required equations/conditions and verify it satisfies the problem.
6. **易错点**: when relevant, point out common mistakes, especially confusing givens with derived results.

### Givens vs Derived Results

Keep problem statements and solution reasoning separate. A value computed during the solution is not a condition of the problem.

- Do not use a derived matrix, rank, characteristic polynomial, pole-placement target, feedback gain, or intermediate equation as if it were given in the题干.
- If you choose a value for design convenience, label it explicitly as a **设计选择**, **构造目标**, or **one feasible choice**, then derive the resulting matrices/gains and verify them.
- Avoid phrases such as “取教材给出的反馈矩阵” unless the source problem explicitly gives that matrix as data. Prefer: “为满足目标极点，构造如下闭环结构……由此反推出反馈矩阵……最后回代校核。”
- Never skip the derivation because the source lists an answer. If the source answer is terse, expand it into a full worked solution from the stated givens.

## Skill Boundary

- Allowed skills: `lecture-tutor`, `firecrawl-parse`, and `pdf` only.
- `firecrawl-parse` is used for PDF-to-markdown text extraction (`firecrawl parse` CLI or `mcp__firecrawl__firecrawl_parse` MCP tool).
- `pdf` is used **only** for `pdf2image` page-to-image conversion and `pypdf` page-range splitting.
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
Step 1: Intake (confirm source, output dir, scope, environment check)
    |
    v
Step 2: Chunked Text Extraction (firecrawl in segments, save to .work/)
    |
    v
Step 3: Formula Page Detection & Vision Extraction (detect garbled math → convert to images → transcribe LaTeX)
    |
    v
Step 4: Segmentation Map (identify chapters/sections, build segment-map.json)
    |
    v
Step 5: Per-Segment Explanation (load one segment's .work/ files, produce 3D explanation, save segment .tex)
    |
    v
Step 6: Assemble, Compile, Verify
```

## Step 1. Intake

- Confirm source PDF path.
- **Ask the user for the corresponding lecture/course PDF** if the source is a homework/assignment document. The lecture PDF is needed for accurate page references. If the user does not provide one, note this limitation and only reference the source document's own pages.
- Confirm output directory. Default: `DeepDive - <stem>` inside the same directory as the source file.
- Determine scope: whole document or specific chapters. Default: whole document.
- **Environment check:**
  - Verify `xelatex` is available. If not, report the blocker immediately.
  - Verify `firecrawl` CLI is available (`which firecrawl`). If not, report the blocker.
  - Check `pdf2image` availability: `python3 -c "from pdf2image import convert_from_path"`. If unavailable, warn the user that formula vision extraction will be skipped (Step 3 becomes optional). Suggest: `brew install poppler && pip install pdf2image`.
- Record any style instructions from the user.
- Create the output folder as the first filesystem step.
- Create `<output-folder>/.work/` for intermediate files.

## Step 2. Chunked Text Extraction

Use `firecrawl-parse` to extract PDF text as markdown. Save all results to `.work/` — never load the entire document into context at once.

### 2a. Determine chunk plan

- Use `pypdf` to read page count only: `PdfReader(path).len(pages)`.
- Split into non-overlapping chunks of at most 10 pages each.
- Example: 72 pages → chunks: 1-10, 11-20, ..., 71-72 (8 chunks).

### 2b. Extract each chunk

**For documents ≤ 50 pages:**

```bash
firecrawl parse "<source-pdf-path>" -o "<output-folder>/.work/full-text.md"
```

Then use `pypdf` to determine where each page boundary falls in the markdown (approximately), or simply read the file in batches using `Read` with `offset`/`limit`.

**For documents > 50 pages:**

Use `pypdf` to split the PDF into chunk PDFs first, then parse each:

```python
from pypdf import PdfReader, PdfWriter
reader = PdfReader(source_pdf)
for chunk_start in range(0, len(reader.pages), 10):
    chunk_end = min(chunk_start + 10, len(reader.pages))
    writer = PdfWriter()
    for p in range(chunk_start, chunk_end):
        writer.add_page(reader.pages[p])
    chunk_path = f"<output-folder>/.work/chunk-{chunk_start//10 + 1}.pdf"
    with open(chunk_path, "wb") as f:
        writer.write(f)
```

Then parse each chunk:

```bash
firecrawl parse "<output-folder>/.work/chunk-1.pdf" -o "<output-folder>/.work/chunk-1.md"
firecrawl parse "<output-folder>/.work/chunk-2.pdf" -o "<output-folder>/.work/chunk-2.md"
# ... etc
```

Delete the temporary chunk PDFs after markdown extraction to save disk space.

### 2c. Read each chunk from disk

Use the `Read` tool with `offset` and `limit` parameters to read each markdown file in batches of no more than 300 lines. Never read more than one chunk at a time.

### 2d. Build segmentation map

Scan each chunk's text for chapter/section titles, definition blocks, theorem blocks, example blocks. Build a JSON segmentation map:

```json
{
  "source_file": "lecture.pdf",
  "total_pages": 72,
  "chunks": ["chunk-1.md", "chunk-2.md", "..."],
  "segments": [
    {
      "id": 1,
      "title": "Chapter 1: Introduction",
      "pages": [1, 2, 3],
      "chunks": ["chunk-1.md"],
      "has_formulas": true,
      "formula_pages": [2, 3]
    }
  ]
}
```

Save to `<output-folder>/.work/segment-map.json`.

## Step 3. Formula Page Detection & Vision Extraction

**This step is optional if `pdf2image` is not available.** If skipped, note in the final document that formulas have not been vision-verified and may contain extraction errors.

### 3a. Detect formula-dense regions

Scan each chunk markdown file for formula corruption patterns:

- Isolated Unicode math symbols: α, β, γ, ∑, ∫, ∂, ∇, →, ∈, ⊂
- Fragmented equation-like text (e.g., `d x d t = A x + B u` instead of `dx/dt = Ax + Bu`)
- Symbol sequences without recognizable word patterns
- Sections labeled "定义", "定理", "引理", "证明" followed by predominantly symbolic content
- Numbered equations like `(1)`, `(2)` followed by garbled content

For each affected region, record the page number. Save the list to `<output-folder>/.work/formula-pages.json`:

```json
{
  "formula_pages": [2, 3, 15, 16, 45],
  "total": 5
}
```

**Be generous:** prefer false positives (converting a non-formula page to image) over false negatives (missing a formula page). The cost of an extra image is minimal.

### 3b. Convert flagged pages to images

Use `pdf2image` to convert each flagged page to a high-resolution PNG:

```python
from pdf2image import convert_from_path
import json

with open("<output-folder>/.work/formula-pages.json") as f:
    data = json.load(f)

for page_num in data["formula_pages"]:
    images = convert_from_path(
        source_pdf,
        first_page=page_num,
        last_page=page_num,
        dpi=300
    )
    images[0].save(f"<output-folder>/.work/page-{page_num}.png")
```

### 3c. Vision-extract formula content

For each formula page image, use the `Read` tool (which supports image files natively) to view the image. Then:

1. Identify each formula in the image.
2. Transcribe each formula into correct LaTeX notation.
3. Save the transcriptions to `<output-folder>/.work/formulas-page-<N>.md`:

```markdown
## Page <N> formulas

### Formula 1 (Theorem 3.2)
$$\frac{dx}{dt} = Ax + Bu$$

### Formula 2 (Definition 3.3)
$$G(s) = C(sI - A)^{-1}B + D$$
```

### 3d. Rules

- In the final TeX output, always prefer vision-extracted LaTeX over text-extracted content for formulas.
- If a page has no formulas, skip the vision step — do not create images for pure-text pages.
- If pdf2image is unavailable, skip this entire step and add a note in the output: "公式未经视觉校验，可能存在误差".

## Step 4. Segmentation Map

Refine the segmentation map built in Step 2d based on the full text extraction:

- Identify chapter boundaries, section titles, definition blocks, theorem blocks, example blocks.
- Assign pages to natural chapter/section boundaries. Each segment should be self-contained.
- For very long documents (100+ pages), segmentation is mandatory.
- Each segment must list its source chunk files and formula files (if applicable).
- Save the updated map to `<output-folder>/.work/segment-map.json`.

## Step 5. Extract and Explain (Per-Segment)

For each segment in the segmentation map:

### 5a. Load segment data

Read only the files belonging to this segment (not other segments' files):
- The relevant chunk markdown files from `.work/`
- Any formula transcription files for pages in this segment

Never load the entire document's text into context. Process one segment at a time.

### 5b. Extract and explain (three-dimensional mode)

For each segment, extract ALL knowledge points and produce a three-dimensional explanation for each one.

Do not skip definitions or theorems. Do not collapse multiple knowledge points into one. Do not skip page references.

#### Dimension 1: Definition & Theorem (定义与定理)

- Quote the exact mathematical statement with page reference: `[文件名] 第X页 [定义/定理Y]`.
- **Formula rule:** for any mathematical content, prefer vision-extracted LaTeX from Step 3 over text-extracted content. If the text extraction produced garbled formulas and no vision extraction is available, flag the formula for manual review.
- List equivalent definitions or conditions when applicable.
- **Every theorem must be followed by its COMPLETE proof** — show every step. If the source omits the proof, supply one. Write `$\qed$` at the end.
- After each definition, provide a concrete example verifying it with actual values.

#### Dimension 2: Intuitive Understanding (直观理解)

- Plain language explanation in at least one substantial paragraph (5+ sentences).
- Intuitive interpretation with analogies when applicable.
- Answer: "Why does this concept exist? What problem does it solve?"
- Address common misconceptions explicitly.
- Include a concrete numerical example.

#### Dimension 3: Source Location (讲义定位)

- `[文件名] 第X页 [节号]`
- Cross-reference related concepts from earlier/later pages.

#### Narrative Flow

- Each section/chapter starts with a **transition paragraph** (2--4 sentences) explaining what question was left unresolved and how this section addresses it.
- When a concept depends on earlier material, briefly recap before continuing.
- Each section/chapter ends with a **本节小结** (3--5 key takeaways).

#### Worked Example Format

When a source example appears, use a worked-example structure that separates source givens from solution reasoning. The exact number of steps may vary, but the separation must be visible.

```latex
\begin{example}[<Source example title>]
\textbf{题干给定。} % only data/conditions explicitly provided by the source
\textbf{解题目标。} % what the example asks us to find/design/prove
\textbf{第一步：选择方法并说明原因。} % theorem/criterion/method and why it applies
\textbf{第二步：代入题干数据逐步计算。} % matrix products, ranks, determinants, etc.
\textbf{第三步：求解中间变量。} % solve equations or construct a design choice
\textbf{第四步：回代校核。} % verify conditions/poles/equations/output behavior
\textbf{易错点。} % optional, but include when a likely misconception exists
\end{example}
```

For design examples, clearly distinguish a **constructive choice** from a **given condition**. Example: “We choose this closed-loop structure to meet the requested pole locations” is acceptable; “the problem gives this feedback matrix” is not acceptable unless the source explicitly states it.

### 5c. Write segment TeX

Write the segment's explanation TeX to `<output-folder>/.work/segment-<N>.tex`. Do not keep the segment TeX in context — save to disk and move on.

### 5d. Use subagents for isolation

When the platform supports it, use subagents for per-segment processing so each segment's context window starts fresh. The subagent receives the `.work/` file paths for its segment and returns the completed segment TeX file path.

## Step 6. Assemble, Compile, and Verify

### 6a. Assemble segments

Read each `.work/segment-<N>.tex` file from disk sequentially and concatenate into the final TeX document. Add preamble and `\end{document}`.

### 6b. Verify coverage

After assembling the full draft, verify:
- Every definition in the source has a corresponding explanation.
- Every theorem in the source has a corresponding explanation.
- Every important example is covered.
- All three dimensions are present for each knowledge point.
- Page references are accurate (cross-check against segment map).
- No section of the source was silently dropped.
- Every formula uses vision-extracted LaTeX (from Step 3) rather than garbled text extraction.
- Every source example has a complete worked solution, not just the final answer.
- Every worked example explicitly distinguishes **题干给定** from derived/calculated quantities.
- No intermediate result is used before it is derived from the source givens or clearly introduced as a design choice.
- Any design choice is labeled as a choice/constructive target and followed by a verification step.
- All calculations needed by the example's conclusion are shown, especially ranks, determinants, characteristic polynomials, matrix equations, pole-placement equations, and feedback gains.

Fix any gaps before proceeding to file output.

### 6c. Write TeX and compile

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

### 6d. Cleanup

The `.work/` directory may be kept (useful for debugging) but should not be counted as a deliverable. Temporary chunk PDFs should be deleted after markdown extraction.

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
- loading all chunk content into context simultaneously (must process one segment at a time)
- using garbled text-extracted formulas instead of vision-transcribed LaTeX
- skipping formula page vision extraction when pdf2image is available
- not verifying pdf2image/poppler availability before starting formula extraction
- treating a derived feedback matrix, pole-placement target, rank result, determinant, or characteristic polynomial as if it were given in the problem statement
- merging “题干给定” and “解题过程” into one paragraph so the reader cannot tell what was assumed versus derived
- omitting the method explanation before calculations in examples
- skipping determinant/rank/matrix-product steps and jumping directly to a final answer
- failing to verify a constructed controller, matrix equation solution, or derived result by substitution
