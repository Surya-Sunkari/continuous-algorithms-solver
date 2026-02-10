---
name: solve
description: Solve a continuous algorithms homework assignment with verified proofs
user-invocable: true
argument-hint: assignmentN
---

# CS 395T Homework Solver

You are solving a graduate-level Continuous Algorithms homework assignment. **Mathematical correctness is the #1 priority.** Follow every instruction in CLAUDE.md precisely.

The target assignment folder is: `$ARGUMENTS`

Execute the following phases in order. Do NOT skip any phase.

---

## PHASE 1: Validation & Setup

1. Verify that `$ARGUMENTS/problems/` exists and contains `.tex` files. If not, stop and tell the user.
2. Verify that `$ARGUMENTS/notes/` exists and contains `.pdf` files. If not, stop and tell the user.
3. Read `example.tex` from the project root to internalize the formatting conventions.
4. Create the `knowledge_base/` directory at the project root if it does not exist.

---

## PHASE 2: Knowledge Base Indexing

For each PDF file in `$ARGUMENTS/notes/`:

1. Derive the YAML filename: `knowledge_base/<pdf-filename-without-extension>.yaml`
2. Check if this YAML file already exists. If it does, skip this PDF.
3. If not indexed yet, read the PDF using the Read tool.
4. Extract ALL of the following into structured YAML:
   - **Definitions** (with number and full statement)
   - **Theorems** (with number, name if any, full statement, and proof sketch if short)
   - **Lemmas** (with number, name if any, full statement)
   - **Corollaries** (with number and full statement)
   - **Propositions** (with number and full statement)
   - **Key remarks** (only if they state a useful result)
5. Write the YAML file following the schema in CLAUDE.md.

Also read ALL existing YAML files in `knowledge_base/` so you have the full theorem inventory from all previous assignments available.

---

## PHASE 3: Problem Analysis

1. Use the Glob tool to find all `.tex` files in `$ARGUMENTS/problems/` and sort them by filename.
2. Read each problem file.
3. For each problem, identify:
   - What is being asked (prove, compute, construct, find counterexample, etc.)
   - Which theorems/definitions from the knowledge base are likely relevant
   - Whether the problem has subparts (and how many)
   - The difficulty level and key mathematical techniques needed

---

## PHASE 4: Solution Generation (Per Problem)

For EACH problem, execute this multi-subagent pipeline. Process problems sequentially (not in parallel) to maintain focus and quality.

### Step A: Draft Solution

Launch a Task subagent (subagent_type: "general-purpose", model: "opus") with this prompt structure:

```
You are a strong graduate math student solving a Continuous Algorithms problem.

PROBLEM:
[paste the full problem text]

AVAILABLE THEOREMS FROM LECTURE NOTES:
[paste relevant theorems from the knowledge base YAML files]

FORMATTING RULES (from example.tex — follow EXACTLY):
- Use \begin{proof}...\end{proof} for proofs
- Use align* for multi-line math, \[...\] for single display math
- Use \textbf{Case N:} for case analysis
- Use \textbf{Claim:} when stating intermediate claims
- Cite theorems as "By Theorem X.Y (Lecture Z)"
- No problem restatement — start directly with the solution
- Subparts use \subsection*{(i)}, \subsection*{(ii)}, etc.

WRITING STYLE (match example.tex exactly):
- Confident, direct, terse prose
- Minimal text between equations — let the math carry the argument
- No filler phrases (see CLAUDE.md for the full banned list)
- Introduce variables inline when first needed
- Use parenthetical justifications: \quad \text{(convexity of } f\text{)}

MATHEMATICAL RIGOR:
- Complete proof, no hand-waving
- Verify every inequality direction
- Handle all edge cases explicitly
- Check that cited theorem hypotheses are satisfied
- Introduce all variables before use

Produce the complete LaTeX solution for this problem. Output ONLY the LaTeX content (no preamble, no \begin{document} — just the \section*{Problem N} through the end of the last proof environment).
```

### Step B: Proof Verification

Launch a SEPARATE Task subagent (subagent_type: "general-purpose", model: "opus") with this prompt:

```
You are a rigorous mathematical proof verifier for a graduate Continuous Algorithms course. Your job is to find errors — be skeptical and thorough.

ORIGINAL PROBLEM:
[paste the full problem text]

PROPOSED SOLUTION:
[paste the draft solution from Step A]

AVAILABLE THEOREMS:
[paste the theorems cited in the solution, from the knowledge base]

Verify the solution by checking EACH of the following:

1. LOGICAL CORRECTNESS: Does each step follow from the previous? Are there any non-sequiturs or gaps?
2. INEQUALITY DIRECTIONS: Are all inequalities (≤, ≥, <, >) in the correct direction? Check each one.
3. THEOREM APPLICATION: For each cited theorem, are ALL hypotheses satisfied? List each hypothesis and verify.
4. EDGE CASES: Are boundary cases handled? (zero vectors, empty sets, degenerate inputs, equality cases)
5. VARIABLE SCOPING: Are all variables properly introduced before use? Are quantifiers correct?
6. ALGEBRAIC CORRECTNESS: Check key algebraic manipulations, especially signs, exponents, and factoring.
7. CONCLUSION: Does the final statement actually prove what was asked?

If you find ANY errors, output them as:
ERROR: [description of the error, which step it's in, and what the correct approach should be]

If the proof is correct, output exactly: VERIFIED

Be extremely thorough. It is better to flag a potential issue than to miss a real error.
```

**If the verifier returns errors:**
- Take the error feedback and launch a NEW solution generator subagent with the original problem PLUS the error feedback. Include the instruction: "The previous attempt had these errors: [errors]. Fix them while maintaining the same formatting and style."
- Re-verify the new solution with a fresh verifier subagent.
- Repeat up to 3 times. If still failing after 3 attempts, include the best attempt in the solution but add a LaTeX comment `% NOTE: This solution could not be fully verified — review manually` at the top of that problem's section.

### Step C: Style & Formatting Check

After verification passes, launch a Task subagent (subagent_type: "general-purpose", model: "sonnet") with this prompt:

```
You are a style and formatting editor for a LaTeX math homework document.

REFERENCE DOCUMENT STYLE (example.tex patterns):
- \section*{Problem N} for problem headings
- \subsection*{(i)} for subparts (lowercase roman numerals)
- \begin{proof}...\end{proof} for proofs
- align* for multi-line math
- \[...\] for display math (NEVER $$...$$)
- \textbf{} for structural labels
- \bigskip\noindent\textit{Counterexample.} for counterexamples
- Parenthetical justifications in math: \quad \text{(reason)}

SOLUTION TO CHECK:
[paste the verified solution]

Check for:
1. FORMATTING: Does it match the reference patterns above exactly?
2. AI-SOUNDING PHRASES: Flag any of these — "delve", "it's important to note", "let's", "straightforward", "it can be shown that", "obviously", "clearly" (unless truly trivial), "we can see that", "interestingly", "notably", "crucial", "vital", "in order to", excessive "Note that"
3. TONE: Does it sound like a strong, confident math student? Not like a textbook, not like AI?
4. LATEX CORRECTNESS: Are there any LaTeX syntax issues?

If changes are needed, output the corrected LaTeX.
If no changes needed, output exactly: APPROVED

When making corrections, output ONLY the corrected LaTeX content.
```

If changes were made, use the corrected version. Store the final solution for this problem.

---

## PHASE 5: Assembly

1. Read `example.tex` to extract the preamble (everything from `\documentclass` through `\maketitle`).
2. Extract the homework number from the folder name (e.g., "assignment2" → 2).
3. Construct the full `solution.tex`:
   - Use the same preamble as example.tex
   - Update `\title{CS 395T Homework N}` with the correct number
   - Update `\date{...}` with today's date
   - Concatenate all problem solutions in order
   - Close with `\end{document}`
4. Write the file to `$ARGUMENTS/solution.tex`.

---

## PHASE 6: Compilation (if pdflatex available)

1. Try running `pdflatex -interaction=nonstopmode $ARGUMENTS/solution.tex` via Bash.
2. If pdflatex is not found, tell the user: "LaTeX is not installed. Install TeX Live or MiKTeX to compile the solution. The .tex file has been generated at $ARGUMENTS/solution.tex."
3. If compilation fails:
   - Read the `.log` file to identify errors
   - Fix the LaTeX issues in solution.tex
   - Recompile (max 3 attempts)
4. If compilation succeeds, inform the user and clean up auxiliary files (.aux, .log, .out).

---

## PHASE 7: Final Report

Tell the user:
- Which problems were solved
- Which problems passed verification on the first attempt vs. required retries
- Any problems that could not be fully verified (flagged for manual review)
- Where the solution file is saved
- Whether compilation succeeded (or if LaTeX needs to be installed)
