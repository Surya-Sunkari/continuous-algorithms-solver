---
name: learn
description: Index lecture note PDFs into the knowledge base
---

# Knowledge Base Indexer

You are indexing lecture notes from a CS 395T (Continuous Algorithms) assignment into a persistent knowledge base of theorems, definitions, and lemmas.

The target assignment folder is: `$ARGUMENTS`

---

## PHASE 1: Validation & Discovery

1. Verify that `$ARGUMENTS/notes/` exists and contains `.pdf` files. If not, stop and tell the user.
2. Create the `knowledge_base/` directory at the project root if it does not exist.
3. List all existing YAML files in `knowledge_base/` to determine what's already indexed.
4. Build two lists:
   - **Already indexed**: PDFs whose corresponding YAML file exists in `knowledge_base/`
   - **To index**: PDFs that need processing (no corresponding YAML file)

If all PDFs are already indexed, skip to Phase 3.

---

## PHASE 2: Index New Notes (Parallel Subagents)

**If there is exactly 1 PDF to index**: Process it directly without spawning a subagent (avoid overhead).

**If there are 2+ PDFs to index**: Spawn one subagent per PDF using the Task tool with `subagent_type: "general-purpose"`. Launch ALL subagents in parallel (single message with multiple Task tool calls).

### Subagent Prompt Template

For each PDF to index, use this prompt (substitute the actual values):

```
You are extracting theorems, definitions, and lemmas from a lecture note PDF into a structured YAML file.

**Input PDF**: <full-path-to-pdf>
**Output YAML**: knowledge_base/<pdf-filename-without-extension>.yaml

Read the PDF (use chunks if longer than 10 pages). Extract ALL of the following:
- Definitions (with number and full statement)
- Theorems (with number, name if any, full statement)
- Lemmas (with number, name if any, full statement)
- Corollaries (with number and full statement)
- Propositions (with number and full statement)
- Key remarks (only if they state a useful result)
- Key intermediate results within proofs — numbered equations, intermediate claims, or steps that are independently useful

Write the YAML file following this exact schema:

```yaml
source: "filename.pdf"
lecture_number: <extract from PDF>
title: "Lecture title extracted from PDF"
items:
  - type: theorem          # theorem | lemma | definition | corollary | proposition | remark
    number: "5.1"          # numbering as it appears in the notes
    name: "Named theorem"  # if the theorem has a name, otherwise empty string
    statement: |
      Full mathematical statement in plain text with LaTeX math notation
    context: "Brief note on when/how this result is typically used"
    proof_notes:           # optional — omit if the proof has no citable internals
      - label: "(3)"       # equation/line label (e.g. "(3)", "Line 4", "Claim 1")
        content: |
          The exact equation or claim in LaTeX math notation
        description: "What this equation/step establishes"
```

Populate `proof_notes` when:
- The proof contains a numbered/labeled equation reusable independently
- An intermediate claim is stronger than the theorem statement itself
- A specific step is the crux and could be invoked in another proof

When done, report the count of items extracted (e.g., "Extracted 12 items: 3 definitions, 5 theorems, 2 lemmas, 2 corollaries").
```

### Collecting Results

After all subagents complete, read each newly created YAML file to count items for the final report.

---

## PHASE 3: Report

Tell the user:
- How many PDFs were found in `$ARGUMENTS/notes/`
- How many were newly indexed vs. already indexed
- Total number of items extracted (theorems, definitions, lemmas, etc.) from newly indexed PDFs
- Where the YAML files were saved
