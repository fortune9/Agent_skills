# Nextflow Channel Operators Reference

Comprehensive guide for channel operators in Nextflow workflows. Reference this when working with complex channel operations.

## Overview

Channel operators transform, filter, combine, and manipulate data flowing through Nextflow pipelines. They enable powerful data processing patterns without explicit loops.

## Table of Contents

- [Channel Factories](#channel-factories)
- [Filtering Operators](#filtering-operators)
- [Transforming Operators](#transforming-operators)
- [Splitting Operators](#splitting-operators)
- [Combining Operators](#combining-operators)
- [Forking Operators](#forking-operators)
- [Collecting Operators](#collecting-operators)
- [Grouping Operators](#grouping-operators)
- [Mathematical Operators](#mathematical-operators)
- [Utility Operators](#utility-operators)

---

## Channel Factories

Create channels from various sources.

### Channel.of

Creates a channel from explicit values.

```groovy
// Single values
Channel.of(1, 2, 3, 4, 5)

// Tuples
Channel.of(['A', 1], ['B', 2], ['C', 3])

// Mixed types
Channel.of('string', 42, [list], [key: 'value'])
```

**Use case**: Testing, static inputs, small datasets

### Channel.fromPath

Creates a channel from file paths matching a glob pattern.

```groovy
// Single pattern
Channel.fromPath('data/*.fastq.gz', checkIfExists: true)

// Multiple patterns
Channel.fromPath(['data/*.fq', 'more/*.fq.gz'])

// Recursive search
Channel.fromPath('data/**/*.bam')

// Options
Channel.fromPath('data/*.txt',
    checkIfExists: true,  // Fail if no matches
    type: 'file',         // 'file', 'dir', 'any'
    hidden: true,         // Include hidden files
    followLinks: false    // Don't follow symlinks
)
```

**Use case**: Reading input files, reference data

### Channel.fromFilePairs

Creates a channel of paired files (e.g., R1/R2 FASTQ files).

```groovy
// Default pattern (*_{1,2}.fastq)
Channel.fromFilePairs('data/*_{1,2}.fastq.gz')
// Emits: ['sample1', [R1, R2]], ['sample2', [R1, R2]]

// Custom pattern
Channel.fromFilePairs('data/*_R{1,2}.fq.gz') { file ->
    file.name.toString().tokenize('_').get(0)
}

// Options
Channel.fromFilePairs('data/*_{1,2}.fq',
    size: 2,              // Expected number of files per pair
    flat: true,           // Emit [id, file1, file2] instead of [id, [file1, file2]]
    checkIfExists: true
)
```

**Use case**: Paired-end sequencing data

### Channel.fromSRA

Download files from NCBI SRA database.

```groovy
Channel.fromSRA('SRR1234567')
Channel.fromSRA(['SRR1234567', 'SRR1234568'])
```

**Use case**: Public sequencing data retrieval

### Channel.empty

Creates an empty channel.

```groovy
def ch_ref = params.reference
    ? Channel.fromPath(params.reference)
    : Channel.empty()
```

**Use case**: Optional inputs, conditional processing

---

## Filtering Operators

Select items based on conditions.

### filter

Emit items that satisfy a condition.

```groovy
// Filter by value
Channel.of(1, 2, 3, 4, 5)
    .filter { it % 2 == 1 }
    // Emits: 1, 3, 5

// Filter by meta field
ch_samples
    .filter { meta, files -> meta.single_end == false }
    // Only paired-end samples

// Filter by file size
Channel.fromPath('data/*.txt')
    .filter { it.size() > 1000 }
    // Only files larger than 1KB
```

**Use case**: Conditional processing, quality filtering

### unique

Remove all duplicate values from the channel.

```groovy
// Remove duplicate values
Channel.of(1, 3, 1, 2, 3, 4)
    .unique()
    // Emits: 1, 3, 2, 4

// Unique by property
ch_samples
    .unique { meta, files -> meta.id }
    // One sample per ID

// Unique by closure
Channel.of(['A', 1], ['B', 2], ['A', 3])
    .unique { it[0] }
    // Emits: ['A', 1], ['B', 2]
```

**Use case**: Deduplication, removing redundant processing

### distinct

Remove consecutive duplicate values.

```groovy
Channel.of(1, 1, 2, 2, 2, 3, 1, 1, 2)
    .distinct()
    // Emits: 1, 2, 3, 1, 2
```

**Use case**: Stream deduplication

### first

Emit only the first item.

```groovy
Channel.of(1, 2, 3, 4, 5)
    .first()
    // Emits: 1

// First N items
Channel.of(1, 2, 3, 4, 5)
    .first(3)
    // Emits: 1, 2, 3
```

**Use case**: Sampling, quick tests

### last

Emit only the last item.

```groovy
Channel.of(1, 2, 3, 4, 5)
    .last()
    // Emits: 5
```

**Use case**: Final result selection

### take

Take the first N items.

```groovy
Channel.of(1, 2, 3, 4, 5)
    .take(3)
    // Emits: 1, 2, 3
```

**Use case**: Limiting processing, testing

### until

Emit items until condition is met.

```groovy
Channel.of(1, 2, 3, 4, 5)
    .until { it > 3 }
    // Emits: 1, 2, 3
```

**Use case**: Conditional termination

---

## Transforming Operators

Apply transformations to channel items.

### map

Transform each item with a function.

```groovy
// Simple transformation
Channel.of(1, 2, 3)
    .map { it * it }
    // Emits: 1, 4, 9

// Add meta field
ch_files
    .map { file ->
        def meta = [id: file.baseName]
        [meta, file]
    }

// Extract from tuple
Channel.of([1, 'a'], [2, 'b'])
    .map { num, letter -> num * 2 }
    // Emits: 2, 4

// Complex transformation
ch_bam
    .map { meta, bam ->
        meta.aligned = true
        meta.tool = 'bwa'
        [meta, bam]
    }
```

**Use case**: Adding metadata, format conversion

### flatMap

Transform and flatten nested structures.

```groovy
// Flatten lists
Channel.of(1, 2, 3)
    .flatMap { n -> [n, n*2, n*3] }
    // Emits: 1, 2, 3, 2, 4, 6, 3, 6, 9

// Split file pairs
Channel.of(['sample1', [file1, file2]])
    .flatMap { id, files ->
        files.collect { [id, it] }
    }
    // Emits: ['sample1', file1], ['sample1', file2]

// Process multi-sample groups
ch_grouped
    .flatMap { group, items ->
        items.collect { meta, file -> [meta, file] }
    }
```

**Use case**: Splitting grouped data, expanding collections

### buffer

Collects items into batches.

```groovy
// Fixed-size batches
Channel.of(1, 2, 3, 4, 5, 6, 7)
    .buffer(size: 3)
    // Emits: [1,2,3], [4,5,6], [7]

// Skip overlapping items
Channel.of(1, 2, 3, 4, 5)
    .buffer(size: 3, skip: 2)
    // Emits: [1,2,3], [3,4,5]
```

**Use case**: Batch processing, windowing

### collate

Similar to buffer, creates batches.

```groovy
Channel.of(1, 2, 3, 4, 5, 6, 7)
    .collate(3)
    // Emits: [1,2,3], [4,5,6], [7]

// With remainder
Channel.of(1, 2, 3, 4, 5, 6, 7)
    .collate(3, false)
    // Emits: [1,2,3], [4,5,6]  (drops incomplete batch)
```

**Use case**: Batch processing

### flatten

Flattens nested lists.

```groovy
Channel.of([1, [2, 3]], [4, [5, 6]])
    .flatten()
    // Emits: 1, 2, 3, 4, 5, 6
```

**Use case**: Unnesting structures

---

## Splitting Operators

Split data into multiple items.

### splitCsv

Parse and split CSV/TSV files.

```groovy
// Basic CSV
Channel.fromPath('samples.csv')
    .splitCsv()
    // Emits each row as a list

// With header
Channel.fromPath('samples.csv')
    .splitCsv(header: true)
    .map { row ->
        [row.sampleId, row.fastq_1]
    }

// TSV files
Channel.fromPath('data.tsv')
    .splitCsv(sep: '\t', header: true)

// Custom separator
Channel.fromPath('data.txt')
    .splitCsv(sep: '|', header: ['col1', 'col2', 'col3'])

// Skip lines
Channel.fromPath('samples.csv')
    .splitCsv(header: true, skip: 2)  // Skip first 2 lines
```

**Use case**: Samplesheet parsing, metadata loading

### splitText

Split text by lines or chunks.

```groovy
// By lines
Channel.fromPath('data.txt')
    .splitText()
    // Emits each line

// By chunks
Channel.fromPath('data.txt')
    .splitText(by: 100)
    // Emits 100-line chunks

// Remove newlines
Channel.fromPath('data.txt')
    .splitText { it.trim() }
```

**Use case**: Log processing, text parsing

### splitFasta

Split FASTA files into records.

```groovy
Channel.fromPath('sequences.fasta')
    .splitFasta()
    // Emits each record

// By chunk size
Channel.fromPath('sequences.fasta')
    .splitFasta(by: 10)
    // 10 records per chunk

// As record map
Channel.fromPath('sequences.fasta')
    .splitFasta(record: [id: true, seqString: true])
    .map { [it.id, it.seqString] }
```

**Use case**: Sequence processing, parallel alignment

### splitFastq

Split FASTQ files into records.

```groovy
Channel.fromPath('reads.fastq')
    .splitFastq()
    // Emits each read

// By chunks
Channel.fromPath('reads.fastq')
    .splitFastq(by: 10000)
    // 10K reads per chunk

// Paired-end
Channel.fromFilePairs('data/*_{1,2}.fq')
    .splitFastq(by: 50000, pe: true, file: true)
    // Splits both files synchronously
```

**Use case**: Parallel read processing

---

## Combining Operators

Combine channels in various ways.

### mix

Merge multiple channels into one.

```groovy
ch1 = Channel.of(1, 2, 3)
ch2 = Channel.of(4, 5, 6)
ch3 = Channel.of(7, 8, 9)

ch1.mix(ch2, ch3)
// Emits: 1, 2, 3, 4, 5, 6, 7, 8, 9 (order may vary)
```

**Use case**: Combining results from different processes

### merge

Merge channels by index position.

```groovy
ch1 = Channel.of('A', 'B', 'C')
ch2 = Channel.of(1, 2, 3)

ch1.merge(ch2)
// Emits: ['A', 1], ['B', 2], ['C', 3]
```

**Use case**: Pairing synchronized data

### join

Join channels by matching key.

```groovy
// Inner join (default)
left = Channel.of(['A', 1], ['B', 2], ['C', 3])
right = Channel.of(['A', 10], ['B', 20])

left.join(right)
// Emits: ['A', 1, 10], ['B', 2, 20]

// Outer join (keep unmatched)
left.join(right, remainder: true)
// Emits: ['A', 1, 10], ['B', 2, 20], ['C', 3, null]

// Custom key index
left.join(right, by: 0)  // Join on first element

// Multiple keys
left.join(right, by: [0, 1])
```

**Use case**: Matching samples with metadata, pairing results

### combine

Create cartesian product of channels.

```groovy
// All combinations
numbers = Channel.of(1, 2, 3)
letters = Channel.of('a', 'b')

numbers.combine(letters)
// Emits: [1,'a'], [1,'b'], [2,'a'], [2,'b'], [3,'a'], [3,'b']

// Combine by key
left = Channel.of([1, 'A'], [1, 'B'], [2, 'C'])
right = Channel.of([1, 'x'], [2, 'y'])

left.combine(right, by: 0)
// Emits: [1, 'A', 'x'], [1, 'B', 'x'], [2, 'C', 'y']
```

**Use case**: Parameter sweeps, sample-condition combinations

### concat

Concatenate channels sequentially.

```groovy
ch1 = Channel.of(1, 2, 3)
ch2 = Channel.of(4, 5, 6)

ch1.concat(ch2)
// Emits: 1, 2, 3, 4, 5, 6 (in order)
```

**Use case**: Sequential processing

### cross

Pairs items from two channels with matching key.

```groovy
left = Channel.of([1, 'A'], [2, 'B'])
right = Channel.of([1, 'x'], [1, 'y'], [2, 'z'])

left.cross(right)
// Emits: [[1, 'A'], [1, 'x']], [[1, 'A'], [1, 'y']], [[2, 'B'], [2, 'z']]
```

**Use case**: Many-to-many matching

---

## Forking Operators

Split channel into multiple output channels.

### branch

Route items to different outputs based on conditions.

```groovy
Channel.of(1, 2, 3, 40, 50, 60)
    .branch {
        small: it < 10
        large: it >= 10
    }
    .set { result }

result.small  // Contains: 1, 2, 3
result.large  // Contains: 40, 50, 60

// With meta
ch_samples
    .branch {
        se: it[0].single_end == true
            return it
        pe: it[0].single_end == false
            return it
    }
    .set { branched }

PROCESS_SE(branched.se)
PROCESS_PE(branched.pe)
```

**Use case**: Conditional routing, different processing paths

### multiMap

Transform items into multiple output channels.

```groovy
Channel.of(1, 2, 3)
    .multiMap {
        doubled: it * 2
        squared: it * it
    }
    .set { result }

result.doubled  // Contains: 2, 4, 6
result.squared  // Contains: 1, 4, 9

// Complex example
ch_input
    .multiMap { meta, files ->
        fastq: [meta, files]
        metadata: meta
        sampleId: meta.id
    }
    .set { split }
```

**Use case**: Creating multiple derived channels

### separate

Deprecated. Use multiMap instead.

---

## Collecting Operators

Gather items into collections.

### collect

Collect all items into a single list.

```groovy
Channel.of(1, 2, 3, 4)
    .collect()
    // Emits: [1, 2, 3, 4]

// Collect with transformation
Channel.of(1, 2, 3)
    .collect { it * 2 }
    // Emits: [2, 4, 6]

// Collect files
ch_results
    .map { meta, file -> file }
    .collect()
    // Emits: [file1, file2, file3, ...]
```

**Use case**: Aggregating results, creating file lists

### collectFile

Collect items into file(s).

```groovy
// Single file
Channel.of('line1', 'line2', 'line3')
    .collectFile(name: 'output.txt', newLine: true)

// By key (multiple files)
Channel.of(['A', 'text1'], ['B', 'text2'], ['A', 'text3'])
    .collectFile { key, value ->
        ["${key}.txt", value + '\n']
    }

// With header/footer
Channel.of('data1', 'data2')
    .collectFile(
        name: 'output.txt',
        seed: 'HEADER\n',
        storeDir: 'results'
    )

// Concatenate files
ch_files
    .collectFile(name: 'combined.txt')
```

**Use case**: Creating reports, merging outputs

### toList

Same as collect(), collects all items into list.

```groovy
Channel.of(1, 2, 3)
    .toList()
    // Emits: [1, 2, 3]
```

### toSortedList

Collect and sort items.

```groovy
Channel.of(3, 1, 4, 1, 5, 9, 2)
    .toSortedList()
    // Emits: [1, 1, 2, 3, 4, 5, 9]

// Custom comparator
Channel.of(['C', 3], ['A', 1], ['B', 2])
    .toSortedList { a, b -> a[0] <=> b[0] }
```

**Use case**: Sorted aggregation

---

## Grouping Operators

Group items by keys or properties.

### groupTuple

Group items by matching key.

```groovy
// Simple grouping
Channel.of(['A', 1], ['B', 2], ['A', 3], ['B', 4])
    .groupTuple()
    // Emits: ['A', [1, 3]], ['B', [2, 4]]

// Group by custom key
ch_files
    .map { meta, file -> [meta.sample, [meta, file]] }
    .groupTuple()
    .map { sample, items ->
        def metas = items.collect { it[0] }
        def files = items.collect { it[1] }
        [metas[0], files]
    }

// With size limit
Channel.of(['A', 1], ['A', 2], ['A', 3])
    .groupTuple(size: 2)
    // Emits: ['A', [1, 2]] (waits for 2 items)

// Sort values
Channel.of(['A', 3], ['A', 1], ['A', 2])
    .groupTuple(sort: true)
    // Emits: ['A', [1, 2, 3]]
```

**Use case**: Grouping by sample, lane merging

### reduce

Reduce channel to single value via accumulation.

```groovy
// Sum
Channel.of(1, 2, 3, 4, 5)
    .reduce { a, b -> a + b }
    // Emits: 15

// With seed
Channel.of(1, 2, 3)
    .reduce(10) { acc, val -> acc + val }
    // Emits: 16

// Complex reduction
ch_stats
    .map { meta, file -> [meta.group, file] }
    .reduce([:]) { map, pair ->
        def (group, file) = pair
        map[group] = (map[group] ?: []) + file
        map
    }
```

**Use case**: Aggregation, summary statistics

### transpose

Transposes lists of tuples.

```groovy
Channel.of([1, [2, 3]], [4, [5, 6]])
    .transpose()
    // Emits: [1, 2], [1, 3], [4, 5], [4, 6]
```

**Use case**: Unpacking grouped data

---

## Mathematical Operators

Perform calculations on channel items.

### count

Count the number of items.

```groovy
Channel.of(1, 2, 3, 4, 5)
    .count()
    // Emits: 5

// Count with condition
Channel.of(1, 2, 3, 4, 5)
    .count { it % 2 == 0 }
    // Emits: 2
```

### min

Find minimum value.

```groovy
Channel.of(3, 1, 4, 1, 5)
    .min()
    // Emits: 1

// By comparator
Channel.of(['A', 3], ['B', 1], ['C', 2])
    .min { a, b -> a[1] <=> b[1] }
    // Emits: ['B', 1]
```

### max

Find maximum value.

```groovy
Channel.of(3, 1, 4, 1, 5)
    .max()
    // Emits: 5
```

### sum

Sum all numeric values.

```groovy
Channel.of(1, 2, 3, 4, 5)
    .sum()
    // Emits: 15
```

---

## Utility Operators

Debugging and inspection operators.

### view

Print channel contents to stdout.

```groovy
// Basic view
ch_input
    .view()

// With custom message
ch_input
    .view { "Processing: ${it}" }

// With prefix
ch_input
    .view { meta, file -> "Sample ${meta.id}: ${file.name}" }
```

**Use case**: Debugging, monitoring

### dump

Similar to view, prints with more detail.

```groovy
ch_input
    .dump(tag: 'input', pretty: true)
```

### tap

Duplicate channel for side effects.

```groovy
ch_input
    .tap { ch_backup }
    .map { ... }

// ch_backup has copy of original channel
```

### ifEmpty

Provide default value if channel is empty.

```groovy
ch_optional
    .ifEmpty('default_value')

// With channel
ch_results
    .ifEmpty(Channel.of([null, []]))
```

**Use case**: Default values, error handling

### randomSample

Randomly sample items from channel.

```groovy
Channel.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
    .randomSample(3)
    // Emits: 3 random items

// With seed for reproducibility
Channel.of(1..100)
    .randomSample(10, seed: 42)
```

**Use case**: Subsampling, testing

### subscribe

Execute closure for each item (side effects).

```groovy
ch_results
    .subscribe { meta, file ->
        log.info "Completed: ${meta.id}"
    }

// With completion handler
ch_results
    .subscribe(
        onNext: { println it },
        onComplete: { println "Done!" },
        onError: { println "Error: $it" }
    )
```

**Use case**: Custom output handling, logging

### set

Store channel in variable.

```groovy
Channel.of(1, 2, 3)
    .map { it * 2 }
    .set { ch_doubled }

// Use later
ch_doubled.view()
```

---

## Complex Patterns

### Multi-way Join

```groovy
// Join three channels
ch_bam = Channel.of(['sample1', 'a.bam'], ['sample2', 'b.bam'])
ch_bai = Channel.of(['sample1', 'a.bai'], ['sample2', 'b.bai'])
ch_meta = Channel.of(['sample1', [group: 'A']], ['sample2', [group: 'B']])

ch_bam
    .join(ch_bai)
    .join(ch_meta)
    .map { id, bam, bai, meta ->
        [meta + [id: id], bam, bai]
    }
```

### Conditional Broadcasting

```groovy
ch_input
    .branch {
        analyze: it[0].analyze == true
        skip: true
    }
    .set { branched }

// Process both paths
ANALYZE(branched.analyze)
PASS_THROUGH(branched.skip)

// Merge results
ANALYZE.out.mix(PASS_THROUGH.out)
```

### Dynamic Grouping and Processing

```groovy
ch_samples
    .map { meta, files ->
        def key = "${meta.group}_${meta.condition}"
        [key, meta, files]
    }
    .groupTuple()
    .map { key, metas, files ->
        def combined_meta = metas[0].clone()
        combined_meta.sample_count = metas.size()
        [combined_meta, files.flatten()]
    }
    .set { ch_grouped }

PROCESS_BATCH(ch_grouped)
```

### Parallel Parameter Sweep

```groovy
// Generate parameter combinations
ch_samples = Channel.fromPath('samples/*.fq')
ch_params = Channel.of(
    [tool: 'bwa', min_score: 20],
    [tool: 'bwa', min_score: 30],
    [tool: 'bowtie2', min_score: 20]
)

ch_samples
    .combine(ch_params)
    .map { file, params ->
        def meta = [
            id: file.baseName,
            tool: params.tool,
            min_score: params.min_score
        ]
        [meta, file, params]
    }
    .set { ch_sweep }

ALIGN(ch_sweep)
```

## Best Practices

1. **Use appropriate operators** - `join` for matching by key, `combine` for all combinations
2. **Check channel cardinality** - Use `.count().view()` to debug
3. **Avoid collecting large channels** - Memory intensive
4. **Use `.view()` liberally** during development
5. **Group before joining** when dealing with multiple files per sample
6. **Use `by:` parameter** in joins/combines for custom keys
7. **Prefer immutable transformations** - Don't modify meta maps in place
8. **Chain operators** for readability

## Performance Tips

1. **Avoid unnecessary collects** - Stream when possible
2. **Use `buffer` for batch processing** instead of collecting everything
3. **Branch early** to avoid unnecessary processing
4. **Use `first()` or `take()`** to limit processing during testing
5. **Group once** - Avoid repeated grouping operations

## References

- Official Nextflow operators: https://www.nextflow.io/docs/latest/operator.html
- Nextflow patterns: https://nextflow-io.github.io/patterns/
