---
name: nextflow-module-development
description: |
  Create or modify Nextflow DSL2 modules following nf-core conventions. Use this skill whenever the user wants to:
  - Create a new Nextflow process module
  - Wrap an existing script (bash, R, Python) in a Nextflow module
  - Add or fix tests for a Nextflow module
  - Follow nf-core module structure and best practices
  - Debug issues with Nextflow modules (parameter handling, testing, file I/O)
  - Understand nf-test testing framework
  
  Trigger when the user mentions "Nextflow module", "create a module", "wrap this script", references the modules/ directory, or asks about nf-test. Make sure to use this skill for any Nextflow module development work, even if the user doesn't explicitly say "use the nextflow module skill".
---

# Nextflow Module Development

Comprehensive guide for creating production-ready Nextflow DSL2 modules following nf-core conventions.

## Quick Start

When the user asks to create a Nextflow module:

1. **Clarify requirements** - Ask about inputs, outputs, script dependencies
2. **Create structure** - Set up directories and files
3. **Implement** - Write main.nf, meta.yml, environment.yml
4. **Test** - Create comprehensive test suite with stub and real tests
5. **Validate** - Run tests, fix issues
6. **Commit** - Single commit after all tests pass

## Module Structure

Every Nextflow module follows this structure:

```
modules/<namespace>/<category>/<tool>/
├── main.nf              # Process definition (required)
├── meta.yml             # Module metadata (required)
├── environment.yml      # Conda dependencies (optional)
└── tests/
    ├── main.nf.test     # nf-test test suite (required)
    ├── main.nf.test.snap # Test snapshots (auto-generated)
    └── tags.yml         # Test tags (required)
```

## Development Workflow

### 1. Understand Requirements

**Before writing any code**, clarify these points with the user:

- **What does this module do?** Get a clear description
- **What are the inputs?** 
  - Meta map fields (required vs optional)?
  - Input files (single/paired, formats)?
  - Parameters (where do they come from)?
- **What are the outputs?**
  - Output files (names, formats)?
  - Stats or log files?
  - Versions file?
- **Does it wrap an existing script?**
  - Where is the script located (bin/)?
  - Does it need modifications?
  - What are its arguments?
- **Are there different modes?**
  - Single-end vs paired-end?
  - Different algorithms or file types?

### 2. Check for Similar Modules

Look at existing modules in the codebase for patterns:
- Parameter handling approaches
- Testing strategies
- Output naming conventions

Common reference modules:
- `modules/exact-nf/fastq/split/` - Multiple outputs, file splitting
- `modules/exact-nf/fastq/sampling/` - Dual-source parameters, deterministic operations
- `modules/exact-nf/fastq/fastqc/` - Simple tool wrapper

### 3. Create Directory Structure

```bash
mkdir -p modules/<namespace>/<category>/<tool>/tests
```

### 4. Write main.nf

#### Basic Template

```groovy
process TOOL_NAME {
    tag "${meta.id}"
    label 'process_low'  // or process_medium, process_high

    conda "${moduleDir}/environment.yml"
    
    input:
    tuple val(meta), path(files)
    val optional_param
    
    output:
    tuple val(meta), path("*.output"), emit: results
    path "*.stats.txt", emit: stats
    path "versions.yml", emit: versions
    
    when:
    task.ext.when == null || task.ext.when
    
    script:
    def args = task.ext.args ?: ''
    def prefix = task.ext.prefix ?: "${meta.id}"
    
    // Parameter validation
    def final_param = meta.param ?: optional_param
    if (!final_param) {
        error("param must be provided in meta.param or as parameter")
    }
    
    // Handle variable file counts
    def file_count = files instanceof List ? files.size() : 1
    def file1 = files instanceof List ? files[0] : files
    def file2 = files instanceof List && file_count > 1 ? files[1] : ""
    
    """
    # Your commands here
    tool_command ${args} --input ${file1} --output ${prefix}.output
    
    # Generate versions
    cat <<-END_VERSIONS > versions.yml
    "${task.process}":
        tool: \$(tool --version | sed 's/^.*version //i')
    END_VERSIONS
    """
    
    stub:
    def prefix = task.ext.prefix ?: "${meta.id}"
    
    """
    # Create stub outputs (uncompressed to avoid MD5 issues)
    touch ${prefix}.output
    touch ${prefix}.stats.txt
    
    cat <<-END_VERSIONS > versions.yml
    "${task.process}":
        tool: 1.0.0
    END_VERSIONS
    """
}
```

#### Key Patterns

**Dual-source parameters** (accept from meta OR parameter):
```groovy
def final_value = meta.value ?: input_param
if (!final_value) {
    error("value must be provided in meta.value or as parameter")
}
```

**Deterministic operations** (reproducible seeds):
```groovy
"""
# Generate deterministic seed from sample ID
seed=\$(echo -n "${meta.id}" | cksum | cut -d' ' -f1)
tool --seed \$seed
"""
```

**Handle single/paired files**:
```groovy
def file_count = files instanceof List ? files.size() : 1
def file1 = files instanceof List ? files[0] : files
def file2 = files instanceof List && file_count > 1 ? files[1] : ""
```

**Named script parameters** (avoid positional ambiguity):
```groovy
"""
# GOOD: Clear and unambiguous
script.sh --input ${file1} --output ${prefix} --seed \$seed

# BAD: Positional args can be misinterpreted
# script.sh ${file1} ${prefix} \$seed
"""
```

### 5. Write meta.yml

```yaml
---
# yaml-language-server: $schema=https://raw.githubusercontent.com/nf-core/modules/master/modules/meta-schema.json
name: "tool_name"
description: Clear description of what this module does
keywords:
  - relevant
  - keywords
tools:
  - "tool":
      description: "Tool description"
      homepage: "https://github.com/author/tool"
      documentation: "https://github.com/author/tool"
      tool_dev_url: "https://github.com/author/tool"
      doi: ""
      licence: ['MIT']

input:
  - meta:
      type: map
      description: |
        Groovy Map containing sample information.
        Required fields: id
        Optional fields: param (description)
  
  - files:
      type: file
      description: Description of input files
      pattern: "*.{ext}"

output:
  - meta:
      type: map
      description: |
        Groovy Map containing sample information
  
  - results:
      type: file
      description: Description of output files
      pattern: "*.output"
  
  - versions:
      type: file
      description: File containing software versions
      pattern: "versions.yml"

authors:
  - "@username"
maintainers:
  - "@username"
```

### 6. Write environment.yml

Only if conda dependencies are needed:

```yaml
---
# yaml-language-server: $schema=https://raw.githubusercontent.com/nf-core/modules/master/modules/environment-schema.json
name: "module_name"
channels:
  - conda-forge
  - bioconda
  - defaults
dependencies:
  - "bioconda::tool=1.0"
```

### 7. Write Tests

For comprehensive nf-test documentation, see [references/nf-test-reference.md](references/nf-test-reference.md).

#### tests/tags.yml

```yaml
module_name:
  - modules/<namespace>/<category>/<tool>/**
```

#### tests/main.nf.test - Basic Structure

```groovy
nextflow_process {
    name "Test Process TOOL_NAME"
    script "../main.nf"
    process "TOOL_NAME"
    
    tag "modules"
    tag "tool_name"

    // Stub test - fast structural validation
    test("Should run basic case - stub") {
        options "-stub"
        
        when {
            process {
                """
                input[0] = [
                    [ id:'test', param: 'value' ],
                    [ file(params.test_data['dataset']['file'], checkIfExists: true) ]
                ]
                input[1] = []  // Empty list for optional parameter, not null
                """
            }
        }

        then {
            assert process.success
            assert snapshot(process.out).match()
        }
    }

    // Real test - functional validation
    test("Should run basic case") {
        when {
            process {
                """
                input[0] = [
                    [ id:'test_real', param: 'value' ],
                    [ file(params.test_data['dataset']['file'], checkIfExists: true) ]
                ]
                input[1] = []
                """
            }
        }

        then {
            assert process.success
            
            // Check outputs exist
            assert process.out.results.size() == 1
            assert process.out.versions.size() == 1
            
            // Validate output content (convert to file/path objects first!)
            def result_file = file(process.out.results[0][1])
            assert result_file.name.contains("test_real")
            
            // Only snapshot deterministic outputs
            assert snapshot(process.out.versions).match()
        }
    }

    // Validation test - error handling
    test("Should fail with invalid input - stub") {
        options "-stub"
        
        when {
            process {
                """
                input[0] = [
                    [ id:'test_fail' ],  // Missing required param
                    [ file(params.test_data['dataset']['file'], checkIfExists: true) ]
                ]
                input[1] = []
                """
            }
        }

        then {
            assert process.failed
        }
    }
}
```

### 8. Run and Validate Tests

```bash
# Run tests with snapshot updates
nf-test test modules/<path>/tests/main.nf.test --update-snapshot

# Verify tests pass without updates
nf-test test modules/<path>/tests/main.nf.test

# Run verbose for debugging
nf-test test modules/<path>/tests/main.nf.test --verbose
```

## Common Pitfalls & Solutions

### 1. Optional Parameter Handling

**❌ WRONG:**
```groovy
input[1] = null  // Causes "channel evaluates to null" error
```

**✅ CORRECT:**
```groovy
input[1] = []  // Use empty list for optional parameters

// In process:
def param = (optional_param instanceof List && optional_param.isEmpty()) ? null : optional_param
```

### 2. Stub File Formats

**❌ WRONG:**
```groovy
stub:
"""
touch ${prefix}.fq.gz  // Empty .gz fails MD5 checks
"""
```

**✅ CORRECT:**
```groovy
output:
path("*.fq*"), emit: reads  // Matches .fq and .fq.gz

stub:
"""
touch ${prefix}.fq  // Uncompressed avoids MD5 issues
"""
```

### 3. Path/File Conversions in Tests

**❌ WRONG:**
```groovy
def file = process.out.files[0]
assert file.text.contains("data")  // String has no .text property
```

**✅ CORRECT:**
```groovy
// Use path() for content, file() for metadata
def content = path(process.out.files[0]).text
assert content.contains("data")

def filename = file(process.out.files[0][1]).name
assert filename.contains("expected")
```

### 4. Positional vs Named Parameters

**❌ WRONG:**
```bash
# Ambiguous - seed interpreted as file2 in single-end mode!
script.sh frac outbase file1 [file2] seed
```

**✅ CORRECT:**
```bash
# Clear and unambiguous
script.sh frac outbase file1 [file2] --seed value
```

### 5. Non-Deterministic Snapshots

**❌ WRONG:**
```groovy
// Timestamps change every run!
assert snapshot(process.out.stats).match()
```

**✅ CORRECT:**
```groovy
// Validate content manually
def stats = path(process.out.stats[0])
assert stats.text.contains("Sample ID:")
assert stats.text.contains("Timestamp:")

// Snapshot only deterministic outputs
assert snapshot(process.out.versions).match()
```

## Testing Best Practices

1. **Mix stub and real tests** - Stubs validate structure, real tests validate functionality
2. **Test all modes** - Single-end, paired-end, different parameter sources
3. **Test validation** - Ensure processes fail appropriately with bad inputs
4. **Use descriptive names** - "Should handle paired-end with custom seed" not "test_pe"
5. **Keep tests fast** - Use small test files, stub when possible
6. **Avoid flaky tests** - Don't snapshot timestamps, random data, or PIDs
7. **Convert types correctly** - Use path() and file() for file operations
8. **Test edge cases** - Missing parameters, empty files, boundary values

## Commit Strategy

**Wait until ALL files are complete and tests pass before committing!**

```bash
# Stage all related files
git add bin/script.sh modules/<namespace>/<category>/<tool>/

# Comprehensive commit message
git commit -m "feat: add <tool> module with <key features>

- Add modules/<path>/ module
  - Feature 1: description
  - Feature 2: description
  - Dual-source parameter handling
  - Supports single-end and paired-end modes
  
- Update bin/script.sh (if applicable)
  - Add --seed named parameter
  - Maintain backward compatibility
  
- Add comprehensive test suite
  - 6 tests: 4 stub + 2 real
  - Tests parameter sources, validation, both modes
  - All tests passing

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
```

## Workflow Checklist

- [ ] Clarify requirements with user
- [ ] Check existing similar modules
- [ ] Create directory structure
- [ ] Write main.nf (script + stub sections)
- [ ] Write meta.yml with complete documentation
- [ ] Write environment.yml (if needed)
- [ ] Write tests/tags.yml
- [ ] Write tests/main.nf.test (mix of stub and real tests)
- [ ] Run tests and generate snapshots
- [ ] Fix any issues
- [ ] Verify all tests pass
- [ ] Commit all files together

## When to Ask for Help

Stop and ask the user if you're uncertain about:

- What the module should output
- How to handle a specific parameter pattern
- Test data locations or formats
- Whether to modify an existing script
- nf-core conventions for this specific case

Better to clarify upfront than iterate multiple times!

## Additional Resources

- **nf-test documentation**: See [references/nf-test-reference.md](references/nf-test-reference.md)
- **nf-core modules**: https://github.com/nf-core/modules
- **Nextflow patterns**: https://nextflow-io.github.io/patterns/
- **nf-test website**: https://www.nf-test.com/

## Example Modules to Study

In the current codebase:
- `modules/exact-nf/fastq/split/` - File splitting, multiple outputs
- `modules/exact-nf/fastq/sampling/` - Parameter handling, deterministic seeding
- `modules/exact-nf/fastq/fastqc/` - Simple wrapper pattern
