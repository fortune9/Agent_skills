## Pattern: Efficient Row Classification for File Parsing

### Problem

Parsing files where rows need different processing often results in O(K×N) complexity: K full-file scans (one per row type) examining each row multiple times. For large files, this causes exponential slowdown.

### Solution 1: Classify-Then-Process (Vectorized)

Best for independent row types that can be batch processed.

**Steps:**
1. **Single-pass classification** - Use cheap string matching (literal substring checks) to tag each row with a family label
2. **Pre-compute shared data** - Extract common fields (timestamps, IDs) once for all rows into parallel arrays
3. **Subset extraction** - Apply expensive operations (regex, parsing) only to rows of each family
4. **Batch assembly** - Build output structures per family, then combine
5. **Post-process joins** - Handle cross-family relationships on assembled data

**Result:** O(N) - each row examined once.

### Solution 2: Single-Loop Dispatch

Best for sequential dependencies or ordered processing.

**Steps:**
1. **Pre-compile patterns** - Define family markers upfront
2. **Cascade dispatch** - Loop once, check patterns specific-to-general, first match wins
3. **Accumulate typed** - Store results in family-specific containers
4. **Convert once** - Build final structures after loop completes

**Result:** O(N) - each row examined once, processed immediately.

### Solution 3: Hybrid

Classify independent families for batch processing; loop over dependent families with ordering requirements.

### When to Use

- **Solution 1:** Large files, independent rows, multiple patterns, maximize performance
- **Solution 2:** Sequential dependencies, simpler logic, moderate files
- **Solution 3:** Mixed independence/dependencies, complex logic

### Key Principles

- Minimize expensive operations (regex, parsing) via cheap classification
- Process each row exactly once
- Pre-compute shared data
- Build by batch/column, not row-by-row
- Profile first to identify actual bottlenecks
