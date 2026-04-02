---
name: nextflow-workflow-development
description: |
  Create Nextflow DSL2 workflows and subworkflows following nf-core conventions.
  Use this skill when the user wants to create a Nextflow workflow or subworkflow,
  orchestrate modules, handle samplesheets, build pipelines, or work with complex
  channel operations. Trigger when the user mentions workflows/ or subworkflows/
  directories, wants to combine multiple modules, or needs to parse input files
  and create channels. This is for workflow orchestration - for creating individual
  process modules, use the nextflow-module-development skill instead.
compatibility:
  tools:
    - Read
    - Write
    - Edit
    - Bash
    - Glob
---

# Nextflow Workflow Development

Comprehensive guide for creating production-ready Nextflow DSL2 workflows and subworkflows following nf-core conventions.

## Quick Start

When the user asks to create a workflow:

1. **Clarify requirements** - Workflow vs subworkflow? Inputs? Modules to orchestrate?
2. **Understand structure** - Entry point workflow or reusable subworkflow?
3. **Implement** - Write workflow, channel operations, parameter handling
4. **Test** - Create nf-test suite with stub and real tests
5. **Validate** - Run tests, fix issues
6. **Commit** - Single commit after all tests pass

## Workflow vs Subworkflow

### Workflows (Entry Points)

Located in `workflows/`, these are executable pipelines with:
- Parameter definitions and help messages
- Samplesheet parsing (often)
- Module/subworkflow orchestration
- Output handling instructions
- Logging and completion messages

Example: `workflows/downsample_fastq.nf`

### Subworkflows (Reusable Components)

Located in `subworkflows/<namespace>/`, these are composable units with:
- Defined `take:`, `main:`, `emit:` sections
- Multiple module orchestration
- Reusable across workflows
- Own test suites

Example: `subworkflows/exact-nf/read_samplesheet/main.nf`

## Workflow Structure

### Entry Point Workflow Pattern

```groovy
#!/usr/bin/env nextflow

nextflow.enable.dsl=2

// Parameters
params.input = null
params.output = "results"

// Help function
def usage() {
    log.info """\
Workflow Name
=============
Description of what this workflow does.

Required parameters:
  --input    Input file or samplesheet
  --output   Output directory (default: ${params.output})

Example:
nextflow run workflows/my_workflow.nf \\
    --input samplesheet.csv \\
    --output results/
=============
""".stripIndent()
}

// Show help if no params
if (!params.input) {
    usage()
    exit 0
}

// Validate required parameters
if (!params.input) {
    exit 1, 'ERROR: --input parameter is required!'
}

// Display parameters
log.info """\
-----------------------------------------------------------------
    Workflow Name
    Input: ${params.input}
    Output: ${params.output}
    Workdir: ${workflow.workDir}
-----------------------------------------------------------------
""".stripIndent()

// Import modules and subworkflows
include { MODULE_NAME } from '../modules/namespace/category/tool/main'
include { SUBWORKFLOW_NAME } from '../subworkflows/namespace/component/main'

// Main workflow
workflow {
    // Create input channels
    ch_input = Channel
        .fromPath(params.input, checkIfExists: true)
        .splitCsv(header: true)
        .map { row ->
            // Build meta map and file paths
            [createMeta(row), collectFiles(row)]
        }

    // Run processes
    MODULE_NAME(ch_input, params.parameter)

    // Completion message
    workflow.onComplete = {
        if (workflow.success) {
            log.info """
            \u001B[32m
            ==================================================================
            Workflow completed successfully!

            Complete at: ${workflow.complete}
            Duration: ${workflow.duration}

            Results are published according to the publishDir directives
            configured in your Nextflow configuration files.
            ==================================================================
            \u001B[0m
            """.stripIndent()
        } else {
            log.info """
            \u001B[31m
            ==================================================================
            Workflow failed with errors!

            Complete at: ${workflow.complete}
            Duration: ${workflow.duration}
            Workflow: ${workflow.scriptFile}
            Workdir: ${workflow.workDir}

            Please check the error messages above for details.
            ==================================================================
            \u001B[0m
            """.stripIndent()
        }
    }
}
```

### Subworkflow Pattern

```groovy
include { MODULE_1 } from '../../modules/namespace/tool1/main'
include { MODULE_2 } from '../../modules/namespace/tool2/main'

workflow SUBWORKFLOW_NAME {
    take:
    ch_input        // channel: [meta, files]
    optional_param  // val: parameter or []

    main:
    // Process channels
    MODULE_1(ch_input, optional_param)

    MODULE_2(
        MODULE_1.out.results,
        []
    )

    // Combine outputs if needed
    ch_combined = MODULE_1.out.stats
        .join(MODULE_2.out.stats)

    emit:
    results = MODULE_2.out.results    // channel: [meta, files]
    stats   = ch_combined              // channel: [meta, stats1, stats2]
    versions = MODULE_1.out.versions
        .mix(MODULE_2.out.versions)
}
```

## Channel Operations

See `references/channel-operators.md` for comprehensive operator documentation.

### Common Patterns

#### Samplesheet Parsing

```groovy
ch_samples = Channel
    .fromPath(params.samplesheet, checkIfExists: true)
    .splitCsv(header: true, sep: ',')
    .map { row ->
        // Validate required columns
        if (!row.sampleId) {
            error "ERROR: sampleId column is required!"
        }
        if (!row.fastq_1) {
            error "ERROR: fastq_1 column is required!"
        }

        // Check files exist
        if (!file(row.fastq_1).exists()) {
            error "ERROR: fastq_1 file does not exist: ${row.fastq_1}"
        }

        // Build meta map
        def meta = [
            id: row.sampleId
        ]

        // Add optional fields
        if (row.sample_size) {
            def size_val = row.sample_size as Double
            meta.sample_size = (size_val > 1) ? size_val as Integer : size_val
        }

        // Handle single-end vs paired-end
        def reads = []
        if (row.fastq_2 && row.fastq_2 != '') {
            if (!file(row.fastq_2).exists()) {
                error "ERROR: fastq_2 file does not exist: ${row.fastq_2}"
            }
            reads = [file(row.fastq_1), file(row.fastq_2)]
            meta.single_end = false
        } else {
            reads = file(row.fastq_1)
            meta.single_end = true
        }

        return [meta, reads]
    }
```

#### Dual-Source Parameters

Handle parameters from CSV column OR command-line:

```groovy
// In workflow
MODULE(ch_input, params.sample_size ?: [])

// Module validates: meta.sample_size OR parameter
def final_value = meta.sample_size ?: input_param
if (!final_value) {
    error("Value must be provided in meta.sample_size or as parameter")
}
```

#### Joining Channels

```groovy
// Inner join by meta.id (default)
left = ch_bam      // [meta, bam]
right = ch_index   // [meta, bai]
joined = left.join(right)  // [meta, bam, bai]

// Join with custom key
left.join(right, by: 0)  // Join by first element

// Outer join (keep unmatched)
left.join(right, remainder: true)
```

#### Grouping by Key

```groovy
// Group by sample ID
ch_files
    .map { meta, file -> [meta.id, [meta, file]] }
    .groupTuple()
    .map { id, items ->
        def metas = items.collect { it[0] }
        def files = items.collect { it[1] }
        [metas[0], files]  // Use first meta, all files
    }
```

#### Combining Channels

```groovy
// Cartesian product
samples = Channel.of(1, 2, 3)
conditions = Channel.of('A', 'B')
samples.combine(conditions)  // [1,'A'], [1,'B'], [2,'A'], ...

// Mix multiple channels
ch_all = ch_fastq
    .mix(ch_bam)
    .mix(ch_cram)
```

#### Branching

```groovy
ch_input
    .branch {
        single: it[0].single_end == true
        paired: it[0].single_end == false
    }
    .set { branched }

// Use separate branches
PROCESS_SE(branched.single)
PROCESS_PE(branched.paired)
```

#### Collecting Results

```groovy
// Collect all items into list
ch_files
    .collect()  // [[meta1, file1], [meta2, file2], ...]

// Collect into file
ch_lines
    .collectFile(name: 'combined.txt', newLine: true)

// Group and collect
ch_results
    .map { meta, file -> [meta.group, file] }
    .groupTuple()
    .map { group, files -> [group, files.collect()] }
```

## Testing Workflows and Subworkflows

### Test File Structure

Workflows and subworkflows use nf-test. Reference `references/nf-test-reference.md` for comprehensive testing documentation.

```
subworkflows/exact-nf/my_subworkflow/
├── main.nf
├── meta.yml
└── tests/
    ├── main.nf.test      # Test suite
    ├── main.nf.test.snap # Snapshots (auto-generated)
    └── tags.yml          # Test tags
```

### Testing Subworkflows

```groovy
nextflow_workflow {
    name "Test SUBWORKFLOW_NAME"
    script "../main.nf"
    workflow "SUBWORKFLOW_NAME"

    tag "subworkflows"
    tag "subworkflow_name"

    test("Should process paired-end data") {
        when {
            workflow {
                """
                input[0] = Channel.of(
                    [
                        [ id:'test_pe', single_end: false ],
                        [
                            file(params.test_data['sarscov2']['illumina']['test_1_fastq_gz'], checkIfExists: true),
                            file(params.test_data['sarscov2']['illumina']['test_2_fastq_gz'], checkIfExists: true)
                        ]
                    ]
                )
                input[1] = []  // Optional parameter
                """
            }
        }

        then {
            assert workflow.success

            // Check output channels exist
            assert workflow.out.results
            assert workflow.out.stats
            assert workflow.out.versions

            // Validate structure
            with(workflow.out.results) {
                assert size() == 1
                assert get(0)[0].id == 'test_pe'
            }

            // Snapshot deterministic outputs
            assert snapshot(workflow.out.versions).match()
        }
    }

    test("Should handle single-end data") {
        when {
            workflow {
                """
                input[0] = Channel.of(
                    [
                        [ id:'test_se', single_end: true ],
                        file(params.test_data['sarscov2']['illumina']['test_1_fastq_gz'], checkIfExists: true)
                    ]
                )
                input[1] = []
                """
            }
        }

        then {
            assert workflow.success
            assert workflow.out.results.size() == 1
        }
    }
}
```

### Testing Full Workflows

For entry point workflows, test with realistic inputs:

```groovy
nextflow_pipeline {
    name "Test workflow downsample_fastq"
    script "workflows/downsample_fastq.nf"

    tag "workflows"
    tag "downsample_fastq"

    test("Should downsample FASTQ files from samplesheet") {
        when {
            params {
                samplesheet = "${projectDir}/test_data/samplesheet_test.csv"
                sample_size = 25
                outdir = "results"
            }
        }

        then {
            assert workflow.success

            // Check key outputs exist
            assert path("${params.outdir}").exists()

            // Validate specific results
            def stats_files = path("${params.outdir}").list()
                .findAll { it.name.endsWith('.sampling_stats.txt') }
            assert stats_files.size() > 0
        }
    }
}
```

### Test Data Setup

Tests use `params.test_data` from `tests/config/test_data.config`:

```groovy
// Reference test data in nf-test
file(params.test_data['sarscov2']['illumina']['test_1_fastq_gz'], checkIfExists: true)
```

The nf-test.config points to the test config:

```groovy
config {
    testsDir "."
    workDir ".nf-test"
    configFile "tests/config/nf-test.config"
}
```

## Best Practices

### Parameter Handling

1. **Always validate required parameters**
```groovy
if (!params.input) {
    exit 1, 'ERROR: --input parameter is required!'
}
```

2. **Use optional parameters with `[]`**, not `null`
```groovy
MODULE(ch_input, params.optional_param ?: [])
```

3. **Provide clear help messages** when no parameters given

4. **Display parameters at start** for transparency

### Channel Operations

1. **Validate files exist** before processing
```groovy
if (!file(row.fastq_1).exists()) {
    error "ERROR: File does not exist: ${row.fastq_1}"
}
```

2. **Use checkIfExists for fromPath**
```groovy
Channel.fromPath(params.input, checkIfExists: true)
```

3. **Build clean meta maps** with required fields
```groovy
def meta = [
    id: row.sampleId,
    single_end: !row.fastq_2
]
```

4. **Use descriptive channel names**
```groovy
ch_fastq_paired
ch_bam_sorted
ch_alignment_stats
```

### Workflow Organization

1. **Keep workflows focused** - One clear purpose per workflow
2. **Extract reusable logic** to subworkflows
3. **Document inputs and outputs** in comments
4. **Use consistent naming** - follow nf-core conventions
5. **Log important steps** for debugging

### Output Handling

1. **Don't hard-code publishDir** in workflows
```groovy
// ✗ BAD - hard-coded in workflow
publishDir "${params.outdir}/results", mode: 'copy'

// ✓ GOOD - handled by config
// Let config files define publishDir directives
```

2. **Emit meaningful channel names** from subworkflows
```groovy
emit:
results = PROCESS.out.files
stats   = PROCESS.out.stats
versions = PROCESS.out.versions
```

3. **Provide completion messages** with status

### Testing Strategy

1. **Test both stub and real execution**
2. **Cover different input scenarios** (single-end, paired-end, edge cases)
3. **Validate channel structure** and content
4. **Snapshot deterministic outputs** only
5. **Use descriptive test names**
6. **Reference test data correctly** via params.test_data

## Common Patterns

### Multi-Sample Processing

```groovy
// Parse samplesheet
ch_samples = Channel
    .fromPath(params.samplesheet, checkIfExists: true)
    .splitCsv(header: true)
    .map { row -> [buildMeta(row), collectFiles(row)] }

// Process each sample
PROCESS(ch_samples, params.options)

// Collect results by group
ch_grouped = PROCESS.out.results
    .map { meta, file -> [meta.group, [meta, file]] }
    .groupTuple()
```

### Conditional Processing

```groovy
// Branch by condition
ch_input
    .branch {
        process_a: it[0].condition == 'A'
        process_b: it[0].condition == 'B'
        skip: true  // Default branch
    }
    .set { branched }

PROCESS_A(branched.process_a)
PROCESS_B(branched.process_b)
```

### Merging Results

```groovy
// Join outputs from different processes
ch_merged = PROCESS_1.out.results
    .join(PROCESS_2.out.results, by: 0)  // Join by meta.id
    .map { meta, result1, result2 ->
        [meta, [result1, result2]]
    }
```

### Handling Optional Inputs

```groovy
// Optional parameter channel
def ch_reference = params.reference
    ? Channel.fromPath(params.reference, checkIfExists: true)
    : Channel.empty()

// Use with module
PROCESS(ch_input, ch_reference)
```

## Troubleshooting

### Common Issues

1. **"Channel evaluates to null"**
   - Use `[]` instead of `null` for optional parameters
   - Check that all required inputs are provided

2. **Samplesheet parsing errors**
   - Validate column names match expectations
   - Check CSV format (commas, quotes)
   - Ensure files referenced in CSV exist

3. **Join mismatches**
   - Verify meta maps have matching IDs
   - Use `.view()` to debug channel contents
   - Consider using `remainder: true` for outer joins

4. **Memory issues with collect()**
   - Avoid collecting large channels into memory
   - Process in batches or use streaming operators

### Debugging Tips

1. **Use `.view()` to inspect channels**
```groovy
ch_input
    .view { "Input: ${it}" }
```

2. **Check channel cardinality**
```groovy
ch_input
    .count()
    .view { "Total items: ${it}" }
```

3. **Validate meta maps**
```groovy
ch_input
    .map { meta, files ->
        assert meta.id
        [meta, files]
    }
```

4. **Run tests with verbose output**
```bash
nf-test test path/to/test.nf.test --verbose
```

## Commit Strategy

Wait until ALL files are complete and tests pass before committing:

```bash
# Stage workflow files
git add workflows/my_workflow.nf
git add subworkflows/exact-nf/my_subworkflow/

# Comprehensive commit message
git commit -m "feat: add my_workflow with subworkflow

- Add workflows/my_workflow.nf
  - Samplesheet parsing with validation
  - Dual-source parameter handling
  - Supports single-end and paired-end modes

- Add subworkflows/exact-nf/my_subworkflow/
  - Orchestrates MODULE_1 and MODULE_2
  - Handles optional parameters
  - Joins results by sample ID

- Add comprehensive test suite
  - Tests both single-end and paired-end
  - Validates channel structure
  - All tests passing

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
```

## Workflow Checklist

**Planning:**
- [ ] Clarify workflow vs subworkflow
- [ ] Identify input sources (files, samplesheets, parameters)
- [ ] List modules/subworkflows to orchestrate
- [ ] Determine output channels needed

**Implementation:**
- [ ] Write help/usage function (workflows only)
- [ ] Define and validate parameters
- [ ] Parse input sources (samplesheet, files)
- [ ] Build meta maps with required fields
- [ ] Include necessary modules/subworkflows
- [ ] Implement channel operations
- [ ] Add completion messages (workflows only)

**Testing:**
- [ ] Create tests/main.nf.test
- [ ] Create tests/tags.yml
- [ ] Write test cases (stub and real)
- [ ] Use params.test_data for test files
- [ ] Validate outputs and channel structure
- [ ] Run tests and generate snapshots
- [ ] Verify all tests pass

**Finalization:**
- [ ] Review code for clarity
- [ ] Check error handling
- [ ] Verify logging is appropriate
- [ ] Commit all files together

## Additional Resources

- **Channel operators**: See `references/channel-operators.md`
- **nf-test guide**: See `references/nf-test-reference.md`
- **Module development**: Use `nextflow-module-development` skill
- **Nextflow patterns**: https://nextflow-io.github.io/patterns/
- **nf-core guidelines**: https://nf-co.re/docs/contributing/guidelines
