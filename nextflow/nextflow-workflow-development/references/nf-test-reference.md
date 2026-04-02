# nf-test Reference Guide

Complete reference for testing Nextflow modules with nf-test.

## What is nf-test?

nf-test is a testing framework specifically designed for Nextflow pipelines. It provides a domain-specific language (DSL) for writing test specifications and supports testing of:
- Processes
- Workflows
- Functions
- Entire pipelines

## Installation and Setup

```bash
# Initialize nf-test in your project
nf-test init

# This creates nf-test.config
```

## Test File Structure

Test files use the `.nf.test` extension and follow this basic structure:

```groovy
nextflow_process {
    name "Test Process NAME"
    script "../main.nf"          // Relative path to process file
    process "PROCESS_NAME"       // Exact process name from main.nf
    
    tag "category"               // Tags for organizing tests
    tag "module_name"
    
    test("Test description") {
        // Optional: Nextflow options
        options "-stub"
        
        when {
            process {
                """
                // Input specification
                input[0] = [meta, files]
                input[1] = parameter
                """
            }
        }
        
        then {
            // Assertions go here
            assert process.success
            assert snapshot(process.out).match()
        }
    }
}
```

## Running Tests

```bash
# Run all tests in a file
nf-test test path/to/test.nf.test

# Run specific test case
nf-test test path/to/test.nf.test --testcase "test description"

# Update snapshots
nf-test test path/to/test.nf.test --update-snapshot

# Clean obsolete snapshots
nf-test test path/to/test.nf.test --clean-snapshot

# Run with specific profile
nf-test test path/to/test.nf.test --profile docker

# Verbose output
nf-test test path/to/test.nf.test --verbose
```

## Assertions

### Basic Assertions

```groovy
// Process success/failure
assert process.success
assert process.failed

// Output existence
assert process.out.channel_name
assert process.out.channel_name.size() == 1

// File checks
assert path(process.out.file[0]).exists()
assert file(process.out.file[0][1]).name.contains("expected")
```

### with() Assertions

Improves readability by creating a scope:

```groovy
with(process.out.my_channel) {
    assert size() == 2
    with(get(0)) {
        assert get(0) == "expected_value"
        assert get(1).name == "expected_file.txt"
    }
}
```

### contains() Assertions

Check if specific item exists:

```groovy
def testData = process.out.channel.collect { item -> item }
assert testData.contains('expected_value')
```

### assertContainsInAnyOrder

Order-agnostic comparison:

```groovy
assert process.out.channel.assertContainsInAnyOrder([
    "item1",
    "item2",
    "item3"
])
```

### assertAll

Run multiple assertions, report all failures:

```groovy
assertAll(
    { assert condition1 },
    { assert condition2 },
    { assert condition3 }
)
```

## Snapshot Testing

Snapshots capture outputs for comparison in future test runs.

### Basic Snapshot Usage

```groovy
// Snapshot entire output
assert snapshot(process.out).match()

// Snapshot specific channel
assert snapshot(process.out.reads).match()

// Snapshot multiple objects
assert snapshot(
    process.out.reads,
    process.out.stats,
    process.out.versions
).match()

// Named snapshot (useful for multiple snapshots in one test)
assert snapshot(process.out.reads).match("reads_output")

// MD5 snapshot (for large objects)
assert snapshot(hugeObject).md5().match()
```

### Snapshot Files

Snapshots are stored in `.nf.test.snap` files alongside the test files. They contain JSON representations of the outputs.

### Snapshot Best Practices

1. **Commit snapshots with code** - Snapshots are part of the test suite
2. **Review snapshot changes** - Check diffs during code review
3. **Update intentionally** - Only update when output should change
4. **Use for deterministic outputs** - Timestamps, random data cause failures
5. **MD5 for large outputs** - Reduces snapshot file size

### When to Use Snapshots

**Good for:**
- File lists and names
- Version information
- Structured output (JSON, YAML)
- Channel structure verification

**Bad for:**
- Outputs with timestamps
- Outputs with random/non-deterministic data
- Large files (use MD5 or check properties instead)

### Updating Snapshots

```bash
# Update all snapshots in a file
nf-test test module.nf.test --update-snapshot

# Clean obsolete snapshots (run all tests first)
nf-test test module.nf.test --clean-snapshot
```

## Working with Files

### path() vs file()

Use `path()` for reading file content, `file()` for file metadata:

```groovy
// Read file content
def content = path(process.out.stats[0])
assert content.text.contains("Sample ID:")

// Get file name
def filename = file(process.out.reads[0][1])
assert filename.name.contains("R1")
```

### File Assertions

```groovy
// File existence
assert path(process.out.file[0]).exists()

// File content
def fileContent = path(process.out.file[0]).text
assert fileContent.contains("expected string")

// File size
assert path(process.out.file[0]).size() > 0

// File list
def files = path(process.out.dir[0]).list()
assert files.size() == 5
```

## Common Patterns

### Testing Different Modes

```groovy
test("Single-end mode") {
    when {
        process {
            """
            input[0] = [
                [ id:'test_se' ],
                [ file("data/se.fq.gz") ]
            ]
            """
        }
    }
    then {
        assert process.success
        def output = file(process.out.reads[0][1])
        assert output.name.contains(".fq")
    }
}

test("Paired-end mode") {
    when {
        process {
            """
            input[0] = [
                [ id:'test_pe' ],
                [
                    file("data/pe_R1.fq.gz"),
                    file("data/pe_R2.fq.gz")
                ]
            ]
            """
        }
    }
    then {
        assert process.success
        def reads = process.out.reads[0][1]
        assert reads instanceof List
        assert reads.size() == 2
    }
}
```

### Testing Validation

```groovy
test("Should fail with invalid input") {
    when {
        process {
            """
            input[0] = [
                [ id:'test_invalid' ],  // Missing required field
                [ file("data/test.fq.gz") ]
            ]
            """
        }
    }
    then {
        assert process.failed
    }
}
```

### Testing Optional Parameters

Use `[]` for optional null parameters, not `null`:

```groovy
test("With optional parameter") {
    when {
        process {
            """
            input[0] = [ meta, files ]
            input[1] = 0.5  // Optional parameter provided
            """
        }
    }
    then {
        assert process.success
    }
}

test("Without optional parameter") {
    when {
        process {
            """
            input[0] = [ meta, files ]
            input[1] = []  // Use [], not null
            """
        }
    }
    then {
        assert process.success
    }
}
```

### Stub Tests

Stub tests run fast and validate structure without actual execution:

```groovy
test("Structure validation - stub") {
    options "-stub"
    
    when {
        process {
            """
            input[0] = [ meta, files ]
            """
        }
    }
    then {
        assert process.success
        assert snapshot(process.out).match()
    }
}
```

### Real Tests with Content Validation

```groovy
test("Functional validation") {
    when {
        process {
            """
            input[0] = [ meta, files ]
            """
        }
    }
    then {
        assert process.success
        
        // Check outputs exist
        assert process.out.results.size() == 1
        
        // Validate content
        def results = path(process.out.results[0])
        assert results.text.contains("expected_value")
        
        // Snapshot deterministic parts only
        assert snapshot(process.out.versions).match()
    }
}
```

## Test Organization

### Tags

Use tags to organize and filter tests:

```groovy
nextflow_process {
    name "Test Process"
    script "../main.nf"
    process "MY_PROCESS"
    
    tag "modules"
    tag "category"
    tag "tool_name"
    
    test("...") { }
}
```

Run tests by tag:
```bash
nf-test test --tag category
```

### Test Data

Reference test data using `params.test_data`:

```groovy
input[0] = [
    [ id:'test' ],
    [ file(params.test_data['sarscov2']['illumina']['test_1_fastq_gz'], checkIfExists: true) ]
]
```

## Debugging Tests

### Verbose Output

```bash
nf-test test path/to/test.nf.test --verbose
```

### Dump Channels

```bash
nf-test test path/to/test.nf.test --dump-channels
```

### Print Variables

```groovy
then {
    println "Output: ${process.out}"
    println "Files: ${process.out.reads[0]}"
    assert process.success
}
```

## Configuration

### nf-test.config

```groovy
config {
    // Test data configuration
    testsDir "tests"
    
    // Nextflow options
    profile "docker"
    
    // Plugins
    plugins {
        load "nft-utils@0.0.2"
    }
}
```

## Tips and Best Practices

1. **Mix stub and real tests** - Stubs for structure, real tests for functionality
2. **Test edge cases** - Missing inputs, invalid parameters, different modes
3. **Avoid non-deterministic snapshots** - Timestamps, PIDs, random data
4. **Use descriptive test names** - Explain what each test validates
5. **Keep tests independent** - Each test should work in isolation
6. **Snapshot only what matters** - Don't snapshot changing data
7. **Check file properties** - Name, size, content patterns
8. **Use path() and file() correctly** - path() for content, file() for metadata
9. **Test validation logic** - Ensure processes fail appropriately
10. **Commit snapshots** - They're part of the test suite

## Common Issues

### "Channel evaluates to null"
Use `[]` instead of `null` for optional parameters:
```groovy
input[1] = []  // ✓ Correct
input[1] = null  // ✗ Wrong
```

### MD5 computation failures with .gz files
Use uncompressed files in stub sections:
```groovy
stub:
"""
touch output.fq  // ✓ Uncompressed
"""
```

### Can't access .name or .text
Convert strings to path/file objects first:
```groovy
def content = path(stringPath).text  // ✓ Correct
def name = file(stringPath).name     // ✓ Correct
```

### Snapshot mismatches from timestamps
Only snapshot deterministic outputs:
```groovy
// Validate dynamic content
def stats = path(process.out.stats[0])
assert stats.text.contains("Timestamp:")

// Snapshot only static content
assert snapshot(process.out.versions).match()
```

## Further Reading

- Official documentation: https://www.nf-test.com/
- nf-core modules examples: https://github.com/nf-core/modules
- Nextflow patterns: https://nextflow-io.github.io/patterns/
