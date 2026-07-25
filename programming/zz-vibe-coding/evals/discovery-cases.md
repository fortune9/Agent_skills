# zz-vibe-coding — Discovery Evals

These evals test **skill discovery**: given a realistic user prompt in a fresh
context, does the agent invoke the `zz-vibe-coding` skill (or not) as intended?

They do **not** test the *content* of the skill (whether the guidelines are
good). They only test the `description` field's triggering behavior:

> Guidelines for coding with AI assistance based on Zhenguo's vibe coding
> methodology. Use this skill when coding is needed, including code planning,
> code implementation, code revision, writing test code.

So the skill SHOULD fire on: **code planning, implementation, revision, and
writing test code.** It should NOT fire on tasks that involve no code
authoring (pure questions, data lookups, running an existing tool, ops).

## How to score a case

A case **PASSES** when the agent's behavior matches `expect`:

- `expect: trigger`   → the agent announces/invokes `zz-vibe-coding` before
  doing the coding work (look for `Skill(zz-vibe-coding)` or an "Using
  zz-vibe-coding" announcement).
- `expect: no-trigger` → the agent answers/acts **without** invoking
  `zz-vibe-coding`.

Run each prompt in a **fresh context** (a subagent), because skill discovery
depends on the initial prompt, not on accumulated conversation. See
`run-discovery-evals.md` for the runner.

Note on ambiguity: cases marked `expect: trigger` are ones a reasonable reader
of the description would agree are coding tasks. Negatives are deliberately
adjacent (they mention code/files/bioinformatics) but require no code
authoring, so they stress-test over-triggering.

---

## Positive cases (MUST trigger)

### P1 — implement a new function
```
Write a Python function that parses a VCF file and returns a dict of
variant counts per chromosome.
```
- id: P1
- expect: trigger
- rationale: direct "code implementation"

### P2 — build a multi-component tool
```
I need a command-line tool that takes a FASTQ file, trims low-quality
bases, and writes a cleaned FASTQ. Can you build it?
```
- id: P2
- expect: trigger
- rationale: complex implementation → planning + implementation

### P3 — revise/refactor existing code
```
Here's my R script for merging expression matrices. It works but it's
slow and messy — can you clean it up and make it faster?
```
- id: P3
- expect: trigger
- rationale: "code revision"

### P4 — write tests
```
Can you add pytest unit tests for the parse_bed() function in utils.py?
```
- id: P4
- expect: trigger
- rationale: "writing test code"

### P5 — planning a coding task (no code yet)
```
I want to write a Snakemake-like mini workflow runner in Python. Before
we write anything, help me plan the architecture.
```
- id: P5
- expect: trigger
- rationale: "code planning" — explicitly the planning stage

### P6 — fix a bug in code
```
This bash script exits with "unbound variable" on line 12. Fix it.
```
- id: P6
- expect: trigger
- rationale: code revision / editing existing code

### P7 — indirect phrasing, still coding
```
I have a folder of CSVs with per-sample metrics. I'd like a script that
combines them into one tidy table and flags samples below a threshold.
```
- id: P7
- expect: trigger
- rationale: implementation, phrased as a data task but requires writing code

---

## Negative cases (must NOT trigger)

### N1 — conceptual question, no code
```
What's the difference between a BAM and a CRAM file?
```
- id: N1
- expect: no-trigger
- rationale: knowledge question, no code authored

### N2 — run an existing tool
```
Run `samtools flagstat` on /data/sample.bam and tell me the mapping rate.
```
- id: N2
- expect: no-trigger
- rationale: operating an existing tool, not writing code

### N3 — interpret results
```
Here's my DESeq2 results table. Which genes are significantly
upregulated at padj < 0.05?
```
- id: N3
- expect: no-trigger
- rationale: data interpretation, no code authoring requested

### N4 — environment / ops
```
My conda environment won't solve — it's stuck on the samtools dependency.
How do I fix the environment?
```
- id: N4
- expect: no-trigger
- rationale: environment troubleshooting, not code authoring
  (borderline; documents the intended boundary)

### N5 — pure explanation of code (read-only)
```
Can you explain what this awk one-liner does? `awk '{sum+=$2} END{print sum}'`
```
- id: N5
- expect: no-trigger
- rationale: explanation/reading, not planning/implementing/revising/testing
  (borderline; if this trips, the description's boundary is too wide)

### N6 — non-technical / unrelated
```
Summarize the key findings of the attached grant progress report.
```
- id: N6
- expect: no-trigger
- rationale: no code at all

---

## Coverage map

| Trigger condition (from description) | Positive case(s) |
|--------------------------------------|------------------|
| code planning                        | P5               |
| code implementation                  | P1, P2, P7       |
| code revision                        | P3, P6           |
| writing test code                    | P4               |

| Over-trigger risk                    | Negative case(s) |
|--------------------------------------|------------------|
| conceptual / knowledge question      | N1               |
| running existing tools               | N2               |
| data interpretation                  | N3               |
| ops / environment                    | N4               |
| read-only code explanation           | N5               |
| non-coding                           | N6               |
```
