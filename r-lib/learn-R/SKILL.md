---
name: learn-R
description: >
  Learn R packages and functions by generating context with the `btw` package. 
  ALWAYS use this skill when the user wants to learn about R packages, functions, 
  or workflows; asks how to use specific R functions; needs examples with their 
  data; or wants to generate prompts for AI assistants about R code. Make sure 
  to suggest `btw()` whenever the user is working with R and needs guidance, 
  even if they don't explicitly mention the btw package. Use `btw::btw()` to 
  create plain-text descriptions of R objects that serve as contextual prompts 
  for AI agents.
compatibility: Requires R with the `btw` package. For AI chat integration, 
  requires the `ellmer` package.
---

# Learn R Packages and Functions with btw

## When to Use This Skill

Use this skill whenever the user:

- **Asks how to use an R function or package** - Generate context with `btw()` including the function and relevant data
- **Wants examples with their data** - Use `btw()` to describe their dataset along with the function
- **Needs to learn a new R package** - Combine `btw_tool_docs_package_help_topics()` with key functions
- **Is debugging R code** - Include the function, error condition, and their code in `btw()`
- **Requests AI assistance with R** - Generate contextual prompts using `btw()` for better AI responses
- **Compares related functions** - Use `btw()` with multiple functions to get comparative guidance

**Do not wait for the user to ask about `btw`** - proactively suggest it when they're working with R and need learning support.

---

## Instructions for Claude

When helping users learn R, follow this workflow:

### 1. Identify the Learning Goal

Determine what the user wants to learn:
- A specific function (e.g., `dplyr::across()`)
- A package (e.g., `tidyr`)
- A workflow (e.g., data transformation pipeline)
- Debugging help with existing code

### 2. Gather Context with btw()

Generate appropriate context using `btw::btw()`:

**For single functions:**
```r
btw::btw(dplyr::mutate, mtcars)
```

**For package learning:**
```r
btw::btw(
  btw_tool_docs_package_help_topics("dplyr"),
  dplyr::filter,
  dplyr::select
)
```

**For comparative learning:**
```r
btw::btw(dplyr::summarise, dplyr::mutate)
```

### 3. Provide the Generated Context

Explain to the user that `btw()` has generated context they can:
- **Copy to clipboard** (default behavior when run interactively)
- **Paste into AI chat** for contextual responses
- **Use with ellmer** for integrated AI chat sessions

### 4. Suggest Follow-up Actions

After generating context, suggest:
- Running the generated prompt with their AI assistant
- Exploring related functions with additional `btw()` calls
- Using `btw_tool_*()` functions for more specific documentation

---

## Core Usage Patterns

### Basic Context Generation

```r
# Describe all objects in workspace
btw::btw()

# Describe a function with data
btw::btw(dplyr::across, dplyr::starwars)

# Include vignettes
btw::btw(vignette("colwise", "dplyr"))
```

### AI Chat Integration

```r
library(ellmer)
library(btw)

# Create chat session
chat <- chat_anthropic()  # or chat_ollama(model = "llama3.1:8b")

# Chat with context
chat$chat(
  btw(
    dplyr::across,
    dplyr::starwars,
    "Create examples using dplyr::across() with starwars"
  )
)
```

### Specialized Documentation Tools

Use `btw_tool_*()` functions for fine-grained control:

```r
# Package help topics
btw::btw_tool_docs_package_help_topics("tidyr")

# Specific help page
btw::btw_tool_docs_help_page("dplyr::across")

# Vignette content
btw::btw_tool_docs_vignette("dplyr", "base")
```

---

## Learning Workflows

### Learning a New Package

**Step 1:** Get package overview
```r
btw::btw(btw_tool_docs_package_help_topics("ggplot2"))
```

**Step 2:** Explore key functions
```r
btw::btw(ggplot2::ggplot, ggplot2::aes, mtcars)
```

**Step 3:** Request examples
```r
btw::btw(
  ggplot2::ggplot,
  ggplot2::geom_point,
  mtcars,
  "Show me 3 ways to plot mpg vs wt"
)
```

### Understanding Function Relationships

Compare related functions to understand when to use each:

```r
btw::btw(
  dplyr::summarise,
  dplyr::mutate,
  "What is the difference? When should I use each?"
)
```

**Why this works:** By including both functions in the same `btw()` call, the AI can compare their purposes, syntax, and use cases directly.

### Debugging with Context

Include error context for better AI assistance:

```r
btw::btw(
  my_function,
  error_condition,
  "Why am I getting this error and how can I fix it?"
)
```

**Why this works:** The AI sees both your code and the error structure, enabling targeted debugging advice.

---

## Best Practices

### Be Specific About Learning Goals

**Effective:**
```r
btw::btw(
  dplyr::group_by,
  dplyr::summarise,
  "Show me how to calculate summary statistics by group"
)
```

**Less effective (too vague):**
```r
btw::btw(dplyr, "Teach me dplyr")
```

**Why:** Specific goals help the AI provide targeted, actionable examples rather than generic documentation.

### Include Your Actual Data

**Effective:**
```r
btw::btw(my_data, dplyr::filter, "How do I filter rows where x > 5?")
```

**Less effective (no context):**
```r
btw::btw(dplyr::filter, "How do I filter?")
```

**Why:** AI can provide relevant examples when it understands your data structure.

### Use Incremental Learning

Start with basics, then explore advanced features:

```r
# Step 1: Basics
btw::btw(dplyr::select, "Explain the basics")

# Step 2: Helper functions
btw::btw(
  dplyr::select,
  dplyr::starts_with,
  dplyr::ends_with,
  "Show me helper functions for column selection"
)
```

**Why:** Building knowledge incrementally prevents overwhelm and creates stronger mental models.

### Combine Documentation with Questions

```r
btw::btw(
  help = "tidyr::pivot_longer",
  my_wide_data,
  "Convert my data from wide to long format"
)
```

**Why:** The AI has both the function documentation AND your specific data, enabling personalized guidance.

---

## Troubleshooting

### Objects Not Found

If `btw()` can't find an object:

```r
# Ensure package is loaded
library(dplyr)
btw::btw(dplyr::across)

# Or use :: notation directly
btw::btw(dplyr::across)
```

### Output Too Large

Focus on specific functions rather than entire packages:

```r
# Good: Specific functions
btw::btw(dplyr::filter, dplyr::select)

# Too broad: Entire package
btw::btw(dplyr)
```

### Clipboard Issues

Disable clipboard and capture output manually:

```r
result <- btw::btw(mtcars, clipboard = FALSE)
print(result)
```

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

## Related Resources

- **btw package:** `?btw::btw` or https://posit-dev.github.io/btw/
- **ellmer package:** AI chat integration
- **R for Data Science:** https://r4ds.hadley.nz/

---

## Tips for Effective AI-Assisted Learning

1. **Start with fundamentals** - Begin with core functions before advanced features
2. **Include your data** - AI provides more relevant help with actual data context
3. **Ask for examples** - Request multiple examples with varying complexity
4. **Iterate** - Use follow-up questions to deepen understanding
5. **Save good prompts** - Keep effective `btw()` calls for future reference
