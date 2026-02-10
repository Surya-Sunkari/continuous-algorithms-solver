---
name: convert
description: Convert problem screenshots to exact LaTeX representations
user-invocable: true
argument-hint: assignmentN
---

# Screenshot to LaTeX Converter

You are converting screenshots of homework problems into their **exact** LaTeX representations. Accuracy is paramount — the output must be a character-perfect transcription of the mathematical content in the screenshots.

The target assignment folder is: `$ARGUMENTS`

---

## PHASE 1: Validation

1. Verify that `$ARGUMENTS/screenshots/` exists. If not, stop and tell the user.
2. Use Glob to find all subdirectories in `$ARGUMENTS/screenshots/` matching the pattern `problem*` (e.g., `problem1`, `problem2`, ...). Sort them numerically.
3. If no problem folders are found, stop and tell the user.
4. Create `$ARGUMENTS/problems/` if it does not exist.

---

## PHASE 2: Conversion (Per Problem)

For each `$ARGUMENTS/screenshots/problemN/` folder:

### Step A: Read Screenshots

1. Use Glob to find all image files in the folder (*.png, *.jpg, *.jpeg, *.bmp, *.webp).
2. Sort them by filename (they may be ordered as page1.png, page2.png, etc. or similar).
3. Read each image using the Read tool. If a problem spans multiple screenshots, read them all in order to get the complete problem.

### Step B: Transcribe to LaTeX

Launch a Task subagent (subagent_type: "general-purpose", model: "opus") with this prompt:

```
You are an expert LaTeX transcriber. Your job is to produce an EXACT transcription of a math problem from screenshots into LaTeX.

SCREENSHOTS:
[include all images for this problem]

RULES — follow these precisely:
1. EXACT TRANSCRIPTION: Reproduce every symbol, subscript, superscript, fraction, summation, integral, matrix, and piece of text exactly as it appears in the screenshot. Do not paraphrase, rephrase, simplify, or add anything.
2. MATH NOTATION: Use standard LaTeX math commands. Use \mathbb for blackboard bold, \mathbf for bold vectors, \text{} for text within math mode, \operatorname{} for named operators.
3. FORMATTING: Preserve the visual structure — numbered parts like (i), (ii), (iii) or (a), (b), (c) should be kept as-is. Preserve paragraph breaks and line breaks where they appear meaningful.
4. NO PREAMBLE: Output only the problem content — no \documentclass, no \begin{document}, no packages. Just the raw LaTeX content that would go inside a document body.
5. NO SOLUTIONS: Only transcribe the problem statement. If a solution or hint appears, do not include it.
6. DISPLAY MATH: Use \[...\] for displayed equations (never $$...$$). Use align* if the original shows aligned/numbered equations.
7. INLINE MATH: Use $...$ for inline math.
8. DOUBLE-CHECK: After transcribing, re-read the screenshot and verify every symbol matches. Pay special attention to:
   - Subscripts vs superscripts
   - Similar-looking symbols (ε vs ∈, ∈ vs ∊, ≤ vs <, ⊂ vs ⊆)
   - Parentheses vs brackets vs braces
   - Bold vs regular vs calligraphic letters
   - Summation/product bounds (above vs below)
   - Fraction numerators and denominators

Output ONLY the LaTeX transcription, nothing else.
```

### Step C: Verification

Launch a SEPARATE Task subagent (subagent_type: "general-purpose", model: "opus") with this prompt:

```
You are a LaTeX transcription verifier. You will be given a screenshot of a math problem and a proposed LaTeX transcription. Your job is to find ANY discrepancies.

SCREENSHOT:
[include the original images]

PROPOSED LATEX:
[paste the transcription from Step B]

Compare the screenshot to the LaTeX character by character. Check:
1. Every mathematical symbol matches exactly
2. Every subscript and superscript is correct
3. All fractions, sums, products, integrals have correct bounds
4. Text content matches word-for-word
5. Numbering of parts/subparts matches
6. No content is missing or added
7. Parentheses, brackets, and delimiters match

If you find ANY discrepancies, list them as:
DISCREPANCY: [what's wrong and what it should be]

If the transcription is exact, output: VERIFIED
```

**If discrepancies are found:**
- Launch a new transcription subagent with the original screenshots PLUS the discrepancy feedback: "The previous transcription had these errors: [discrepancies]. Fix them."
- Re-verify with a fresh verifier subagent.
- Repeat up to 3 times. If still failing, use the best attempt and add a LaTeX comment at the top: `% NOTE: Transcription could not be fully verified — check against original screenshot`

### Step D: Write Output

Write the verified LaTeX to `$ARGUMENTS/problems/pN.tex` (where N matches the problem number from the folder name).

---

## PHASE 3: Report

Tell the user:
- How many problems were converted
- Which problems passed verification on the first attempt vs. required retries
- Any problems that could not be fully verified
- Where the output files are saved
- Remind the user they can now run `/solve $ARGUMENTS` to generate solutions
