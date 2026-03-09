---
name: learn-R
description: >
    Learn R packages and functions by generating context with the
    `btw` package. Use `btw::btw()` to create plain-text descriptions
    of R objects, functions, and documentation that can be used as
    prompts for AI agents. This skill provides examples and best
    practices for using `btw` to enhance your learning experience with
    R.

metadata:
  author: Zhenguo Zhang
  version: 1.0
license: MIT
---

## Overview

This skill provides guidance on using the `btw` package to generate context for learning R packages and functions. The `btw::btw()` function creates plain-text descriptions of R objects (data frames, functions, package documentation) that can be used as prompts for AI agents.

**Key use cases:**
- Learning how to use specific R functions or packages
- Generating contextual prompts for AI assistants
- Documenting computational environments for LLM interactions
- Creating reproducible context for code generation tasks

---


## Basic Usage

### Describe Objects in Your Environment

```r
# Describe all objects in your workspace
btw::btw()
```

### Describe Specific Functions and Data

```r
# Describe a function and a dataset
btw::btw(dplyr::mutate, mtcars)

# Describe multiple objects
btw::btw(dplyr::across, dplyr::starwars, tidyr::pivot_longer)
```

### Include Vignettes and Documentation

```r
# Include package vignettes
btw::btw(vignette("colwise", "dplyr"))

# Include help pages
btw::btw(help = "dplyr::across")
```

---


### Interactive Workflow

```r
# Copy context to clipboard for pasting into chat interface
btw::btw(dplyr::mutate, mtcars, clipboard = TRUE)
# Result is automatically copied to clipboard
```

---

## btw Tool Functions

Use specialized `btw_tool_*()` functions for fine-grained control:

### Package Help Topics

```r
# Get help topics for a package
btw::btw_tool_docs_package_help_topics("dplyr")
```

### Help Pages

```r
# Get specific help page
btw::btw_tool_docs_help_page("dplyr::across")
```

### Function Documentation

```r
# Get function documentation
btw::btw_tool_docs_function("dplyr", "across")
```

### Vignettes

```r
# Get vignette content
btw::btw_tool_docs_vignette("dplyr", "colwise")
```

---

## Common Patterns

### Learning a New Package

```r
# Get comprehensive context for learning a package
btw::btw(
  btw_tool_docs_package_help_topics("tidyr"),
  tidyr::pivot_longer,
  tidyr::pivot_wider,
  "Explain the key concepts of tidyr and when to use pivot operations"
)
```

### Understanding Function Relationships

```r
# Compare related functions
btw::btw(
  dplyr::summarise,
  dplyr::mutate,
  "What is the difference between summarise() and mutate()? When should I use each?"
)
```

### Getting Examples

```r
# Request examples with context
btw::btw(
  ggplot2::ggplot,
  ggplot2::aes,
  mtcars,
  "Show me 3 different ways to visualize the relationship between mpg and wt"
)
```

### Debugging Help

```r
# Get help with debugging
btw::btw(
  my_function,
  error_condition,
  "Why am I getting this error and how can I fix it?"
)
```

---

## Advanced Usage

### Custom Context Building

```r
# Build complex context programmatically
context <- c(
  btw_tool_docs_help_page("dplyr::filter"),
  btw_tool_docs_help_page("dplyr::select"),
  "Create a workflow that combines filtering and selecting"
)

btw::btw(!!!context)
```

### Multiple Packages

```r
# Learn about package interactions
btw::btw(
  dplyr::filter,
  tidyr::drop_na,
  ggplot2::ggplot,
  "Show me a complete workflow from data cleaning to visualization"
)
```

### Clipboard Control

```r
# Disable clipboard copying
result <- btw::btw(mtcars, clipboard = FALSE)

# Access the text content
content <- as.character(result)
```

---

## Best Practices

### 1. Be Specific About What You Want to Learn

```r
# Good: Specific learning goal
btw::btw(
  dplyr::group_by,
  dplyr::summarise,
  "Show me how to calculate summary statistics by group"
)

# Less helpful: Too vague
btw::btw(dplyr, "Teach me dplyr")
```

### 2. Include Relevant Data

```r
# Good: Include the actual data you're working with
btw::btw(my_data_frame, dplyr::filter, "How do I filter this data?")

# Less helpful: No context
btw::btw(dplyr::filter, "How do I filter?")
```

### 3. Combine Documentation with Examples

```r
# Good: Documentation + data + specific question
btw::btw(
  help = "tidyr::pivot_longer",
  my_wide_data,
  "Convert my data from wide to long format"
)
```

### 4. Use Incremental Learning

```r
# Start with basics
btw::btw(dplyr::select, "Explain the basics of select()")

# Then explore advanced features
btw::btw(
  dplyr::select,
  dplyr::starts_with,
  dplyr::ends_with,
  "Show me helper functions for selecting multiple columns"
)
```

---

## Workflow Examples

### Learning Data Transformation

```r
library(btw)

# Get context for learning dplyr transformations
chat_prompt <- btw(
  vignette("base", "dplyr"),
  dplyr::mutate,
  dplyr::filter,
  dplyr::select,
  mtcars,
  "Create a step-by-step tutorial for data transformation with dplyr"
)
```

### Learning Visualization

```r
# Get context for learning ggplot2
chat_prompt <- btw(
  ggplot2::ggplot,
  ggplot2::geom_point,
  ggplot2::geom_smooth,
  ggplot2::facet_wrap,
  mtcars,
  "Teach me the grammar of graphics using ggplot2"
)
```

### Learning Statistical Modeling

```r
# Get context for learning modeling
chat_prompt <- btw(
  stats::lm,
  stats::glm,
  broom::tidy,
  broom::glance,
  mtcars,
  "Explain linear modeling in R and how to interpret results"
)
```

---

## Troubleshooting

### Objects Not Found

```r
# Ensure packages are loaded
library(dplyr)
btw::btw(dplyr::across)

# Or use :: notation
btw::btw(dplyr::across)
```

### Large Output

```r
# Focus on specific functions rather than entire packages
btw::btw(dplyr::filter, dplyr::select)  # Good
btw::btw(dplyr)  # May be too much
```

### Clipboard Issues

```r
# Disable clipboard and capture output
result <- btw::btw(mtcars, clipboard = FALSE)
print(result)
```

---

## Related Resources

- **btw package documentation:** `?btw::btw`
- **btw GitHub:** https://posit-dev.github.io/btw/
- **ellmer package:** For AI chat integration
- **R for Data Science:** https://r4ds.hadley.nz/

---

## Quick Reference

| Task | Command |
|------|---------|
| Describe workspace | `btw::btw()` |
| Describe function | `btw::btw(dplyr::mutate)` |
| Describe data | `btw::btw(mtcars)` |
| Include vignette | `btw::btw(vignette("name", "pkg"))` |
| Copy to clipboard | `btw::btw(..., clipboard = TRUE)` |
| Chat with context | `chat$chat(btw(...))` |
| Package help topics | `btw_tool_docs_package_help_topics("pkg")` |
| Help page | `btw_tool_docs_help_page("pkg::func")` |

---

## Tips for AI Agent Learning

1. **Start with fundamentals:** Begin with core functions before advanced features
2. **Include your data:** AI can provide more relevant help with actual data context
3. **Ask for examples:** Request multiple examples with varying complexity
4. **Iterate:** Use follow-up questions to deepen understanding
5. **Save good prompts:** Keep effective `btw()` calls for future learning sessions
