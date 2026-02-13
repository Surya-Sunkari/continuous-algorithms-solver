# CS 395T Homework Solver

Automated tool for generating mathematically rigorous LaTeX solutions for CS 395T (Continuous Algorithms) homework assignments.

## Project Structure

```
homework_solver/
├── example.tex                  # Formatting template — DO NOT MODIFY
├── CLAUDE.md                    # This file
├── knowledge_base/              # Auto-indexed theorem database (YAML, one file per lecture)
├── .claude/skills/
│   ├── solve/SKILL.md           # The /solve skill
│   ├── convert/SKILL.md         # The /convert skill
│   ├── learn/SKILL.md           # The /learn skill
│   └── combine/SKILL.md         # The /combine skill
├── assignmentN/
│   ├── homework.pdf             # Input: full homework PDF (for /convert, PDF mode)
│   ├── screenshots/             # Input: problem screenshots (for /convert, screenshot mode)
│   │   ├── problem1/            #   one folder per problem with image files
│   │   ├── problem2/
│   │   └── ...
│   ├── problems/                # Input/Output: one .tex file per problem (p1.tex, p2.tex, ...)
│   ├── notes/                   # Input: lecture note PDFs
│   ├── solutions/               # Output: individual solved problems (p1.tex, p2.tex, ...)
│   └── solution.tex             # Output: combined final document
```

## Usage

### Option A: Problems already in LaTeX
1. Create `assignmentN/problems/` and place individual problem `.tex` files (p1.tex, p2.tex, etc.)
2. Create `assignmentN/notes/` and place relevant lecture note PDFs
3. Run `/learn assignmentN`
4. Run `/solve assignmentN 1`, `/solve assignmentN 2`, etc. for each problem
5. Run `/combine assignmentN`

### Option B: Problems as a single PDF
1. Place the homework PDF in `assignmentN/` (e.g., `assignmentN/homework.pdf`)
2. Create `assignmentN/notes/` and place relevant lecture note PDFs
3. Run `/convert assignmentN` — auto-detects the PDF, splits into individual `.tex` files in `assignmentN/problems/`
4. Run `/learn assignmentN`
5. Run `/solve assignmentN 1`, `/solve assignmentN 2`, etc. for each problem
6. Run `/combine assignmentN`

### Option C: Problems as screenshots
1. Create `assignmentN/screenshots/problem1/`, `problem2/`, etc. and place screenshot images in each
2. Create `assignmentN/notes/` and place relevant lecture note PDFs
3. Run `/convert assignmentN` — converts screenshots to `.tex` files in `assignmentN/problems/`
4. Run `/learn assignmentN`
5. Run `/solve assignmentN 1`, `/solve assignmentN 2`, etc. for each problem
6. Run `/combine assignmentN`

## Knowledge Base

The `knowledge_base/` directory stores extracted theorems, lemmas, definitions, and corollaries from lecture notes as structured YAML files. These are auto-indexed when `/learn` runs and persist across assignments.

### YAML Schema

```yaml
source: "filename.pdf"
lecture_number: 5
title: "Lecture title extracted from PDF"
items:
  - type: theorem          # theorem | lemma | definition | corollary | proposition | remark
    number: "5.1"          # numbering as it appears in the notes
    name: "Named theorem"  # if the theorem has a name, otherwise empty string
    statement: |
      Full mathematical statement in plain text with LaTeX math notation
    context: "Brief note on when/how this result is typically used"
```

## LaTeX Style Guide (from example.tex)

These conventions MUST be followed exactly in all generated solutions:

- **Document class**: `\documentclass[12pt]{article}`
- **Packages**: amsmath, amssymb, amsthm, mathtools, enumerate, graphicx, subcaption, float, geometry
- **Margins**: `\geometry{margin=1in}`
- **Title**: `CS 395T Homework N`
- **Author**: `Surya Sunkari (svs879)`
- **Date**: The date the solution is generated
- **Problem headings**: `\section*{Problem N}`
- **Subpart headings**: `\subsection*{(i)}`, `\subsection*{(ii)}`, etc. (lowercase roman numerals)
- **Proofs**: `\begin{proof}...\end{proof}` environment from amsthm
- **Multi-line math**: `align*` environment (NOT `eqnarray`)
- **Single display math**: `\[...\]` (NEVER `$$...$$`)
- **Bold labels for structure**: `\textbf{Case 1:}`, `\textbf{Claim:}`, `\textbf{Construction of $X$:}`
- **Counterexamples**: `\bigskip\noindent\textit{Counterexample.}`
- **Enumerated steps**: `\begin{enumerate}` with `\item`
- **Theorem citations**: `By Theorem X.Y (Lecture Z)` or `By Lemma X (Lecture Z)`
- **No problem restatement** — jump directly into the solution/proof

## Writing Style Rules

The output must sound like a confident, strong math graduate student. Match the exact voice and density of `example.tex`.

### DO
- State claims directly and confidently
- Use "we" sparingly and naturally (e.g., "We show that...", "We verify that...")
- Write tight, dense prose between equations
- Use sentence fragments for labels: `Base case ($d=1$):`, `Inductive step:`
- Let the math speak — minimal surrounding text
- Introduce variables right when needed, inline with statements
- Use parenthetical notes for justifications: `\quad \text{(convexity of } f\text{)}`

### NEVER USE THESE PHRASES
- "delve", "it's important to note", "it's worth noting"
- "let's", "let us" (use "we" or imperative instead)
- "straightforward", "straightforwardly"
- "it can be shown that", "one can show that"
- "obviously", "clearly" (unless the step is truly a single trivial observation)
- "we can see that", "we can observe that"
- "interestingly", "notably", "crucially", "vitally"
- "in order to" (just use "to")
- "it follows that" (overused — vary the phrasing)
- "without loss of generality" (only if actually WLOG)

### AVOID
- Excessive transition words between steps
- Meta-commentary about the proof strategy ("We will use the following approach...")
- Explaining *why* you are doing each step — just do it
- Over-qualifying statements with hedging language
- Starting sentences with "Note that" repeatedly
- Long English paragraphs between equations — keep it tight

## Mathematical Rigor Requirements

**Correctness is the #1 priority.** These are graduate-level continuous algorithms problems.

- Every proof must be complete with no hand-waving or gaps
- Every inequality must have its direction verified
- All edge cases must be handled explicitly (e.g., boundary cases, zero vectors, degenerate inputs)
- When citing a theorem from lecture notes, verify that ALL hypotheses are satisfied
- Variables must be introduced before use (or simultaneously with their first use)
- Quantifiers must be explicit: "for all", "there exists", with correct ordering
- Dimensional analysis / type-checking: ensure operations are valid (e.g., not adding scalars to vectors)
- When taking limits, infima, or suprema, verify they exist
- For inequalities in chains, verify each step independently
