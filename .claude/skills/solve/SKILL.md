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

## IMPORTANT: Subagent Output Strategy

All subagents in this skill MUST write their output to files using the Write tool, NOT return text in their response. This ensures the main agent can reliably read subagent results.

- Solution draft subagents write to: `$ARGUMENTS/.drafts/pN_solution.tex`
- Verification subagents write to: `$ARGUMENTS/.drafts/pN_verification.txt`
- Style check subagents write to: `$ARGUMENTS/.drafts/pN_style.tex` (corrected) or `$ARGUMENTS/.drafts/pN_style.txt` (APPROVED)

The main agent creates `$ARGUMENTS/.drafts/` before launching subagents and cleans it up after assembly.

---

## PHASE 1: Validation & Setup

1. Verify that `$ARGUMENTS/problems/` exists and contains `.tex` files. If not, stop and tell the user.
2. Verify that `$ARGUMENTS/notes/` exists and contains `.pdf` files. If not, stop and tell the user.
3. Read `example.tex` from the project root to internalize the formatting conventions.
4. Create the `knowledge_base/` directory at the project root if it does not exist.
5. Create `$ARGUMENTS/.drafts/` for intermediate subagent output files.

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

Produce the complete LaTeX solution for this problem (no preamble, no \begin{document} — just the \section*{Problem N} through the end of the last proof environment).

IMPORTANT: Use the Write tool to save your solution to: [path to $ARGUMENTS/.drafts/pN_solution.tex]
Do NOT just output the text — you MUST write it to the file.
```

### Step B: Proof Verification

After the draft subagent completes, launch a SEPARATE Task subagent (subagent_type: "general-purpose", model: "opus"):

```
You are a rigorous mathematical proof verifier for a graduate Continuous Algorithms course. Your job is to find errors — be skeptical and thorough.

Read the original problem from: [path to $ARGUMENTS/problems/pN.tex]

Read the proposed solution from: [path to $ARGUMENTS/.drafts/pN_solution.tex]

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

IMPORTANT: Use the Write tool to save your verdict to: [path to $ARGUMENTS/.drafts/pN_verification.txt]
Write either "VERIFIED" if the proof is correct, or list errors as:
ERROR: [description of the error, which step it's in, and what the correct approach should be]

Be extremely thorough. It is better to flag a potential issue than to miss a real error.
```

**After the verifier completes, read `pN_verification.txt`:**
- If VERIFIED, proceed to Step C.
- If errors were found:
  - Launch a NEW solution generator subagent with the original problem PLUS the error feedback. Instruct it to read `pN_solution.tex`, fix the listed errors, and overwrite the file.
  - Launch a fresh verifier to re-check and overwrite `pN_verification.txt`.
  - Repeat up to 3 times. If still failing after 3 attempts, prepend a LaTeX comment `% NOTE: This solution could not be fully verified — review manually` to the solution file.

### Step C: Style & Formatting Check

After verification passes, launch a Task subagent (subagent_type: "general-purpose", model: "sonnet"):

```
You are a style and formatting editor for a LaTeX math homework document.

Read the reference document style from: example.tex (in the project root)

Read the solution to check from: [path to $ARGUMENTS/.drafts/pN_solution.tex]

REFERENCE PATTERNS (from example.tex):
- \section*{Problem N} for problem headings
- \subsection*{(i)} for subparts (lowercase roman numerals)
- \begin{proof}...\end{proof} for proofs
- align* for multi-line math
- \[...\] for display math (NEVER $$...$$)
- \textbf{} for structural labels
- \bigskip\noindent\textit{Counterexample.} for counterexamples
- Parenthetical justifications in math: \quad \text{(reason)}

Check for:
1. FORMATTING: Does it match the reference patterns above exactly?
2. AI-SOUNDING PHRASES: Flag any of these — "delve", "it's important to note", "let's", "straightforward", "it can be shown that", "obviously", "clearly" (unless truly trivial), "we can see that", "interestingly", "notably", "crucial", "vital", "in order to", excessive "Note that"
3. TONE: Does it sound like a strong, confident math student? Not like a textbook, not like AI?
4. LATEX CORRECTNESS: Are there any LaTeX syntax issues?

If changes are needed, use the Write tool to save the CORRECTED LaTeX to: [path to $ARGUMENTS/.drafts/pN_style.tex]
If no changes needed, use the Write tool to write "APPROVED" to: [path to $ARGUMENTS/.drafts/pN_style.txt]
```

**After the style checker completes:**
- If `pN_style.txt` exists and says APPROVED, the solution in `pN_solution.tex` is final.
- If `pN_style.tex` exists, that corrected version is the final solution.

---

## PHASE 5: Assembly

1. Read `example.tex` to extract the preamble (everything from `\documentclass` through `\maketitle`).
2. Extract the homework number from the folder name (e.g., "assignment2" → 2).
3. For each problem, read the final solution:
   - If `$ARGUMENTS/.drafts/pN_style.tex` exists, use that.
   - Otherwise use `$ARGUMENTS/.drafts/pN_solution.tex`.
4. Construct the full `solution.tex`:
   - Use the same preamble as example.tex
   - Update `\title{CS 395T Homework N}` with the correct number
   - Update `\date{...}` with today's date
   - Concatenate all problem solutions in order
   - Close with `\end{document}`
5. Write the file to `$ARGUMENTS/solution.tex`.
6. Delete the `$ARGUMENTS/.drafts/` directory and all its contents.

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
