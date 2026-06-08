---
name: learn-anything
description: Create comprehensive quick-start tutorials for programming tools, libraries, frameworks, CLI tools, algorithms, cloud services (AWS/GCP), and related technical topics. Use this skill whenever the user wants to learn about a new tool, understand how to use a package (like ggplot2, pandas, docker), get started with a service (like AWS S3, ufw firewall), or understand technical concepts (like algorithms, design patterns). Trigger on phrases like "teach me X", "how do I use X", "learn X", "tutorial for X", "quick start on X", or when the user mentions wanting to understand a new technical tool or concept.
---

# Learn Anything

Generate comprehensive, practical tutorials for technical topics by researching official documentation and creating structured learning materials.

## Workflow

Follow this sequence for every tutorial request:

### 1. Search and Find Official Documentation

Start by searching Google for the topic's official documentation.

**Use the `browser-use` skill** to perform Google searches and find official documentation pages. And use WebFetch to retrieve needed content from those pages. If WebFetch fails to retrieve usable content, the browser-use skill can extract the content directly:
- Search Google for the topic
- Open relevant results
- Extract key information from documentation pages
- If failed to find any relevant pages or the google search failed,
  ask the user to provide links to official documentation and use
  WebFetch to retrieve content from those links.

Focus on:
- Official project websites and documentation
- Tutorials and "Getting Started" guides
- GitHub repositories with README and wiki pages
- Authoritative sources (avoid random blog posts unless official docs don't exist)

After finding relevant pages with browser-use, use WebFetch to retrieve full documentation content. If WebFetch returns unusable content (JavaScript-heavy sites, blocked content), the browser-use skill can extract the content directly.

### 2. Select Documentation Pages

Fetch up to **5 pages maximum** of the most relevant documentation:
- Prioritize: installation guides, quick start tutorials, core concepts, API references
- If you find that you need more than 5 pages, **stop and ask the user for permission** before proceeding
- Explain which additional pages you want to fetch and why they're important

### 3. Create the Tutorial Structure

Based on the documentation, write a comprehensive tutorial with these sections:

#### A. Introduction
- Brief description of what the tool/topic is
- Main use cases and why someone would use it
- Prerequisites (if any)

#### B. Installation & Setup
- Step-by-step installation instructions for the user's platform
- Configuration steps if needed
- How to verify the installation worked

#### C. Key Concepts
- Core concepts the user needs to understand
- Use a table format when listing multiple concepts:

| Concept | Description | Example |
|---------|-------------|---------|
| concept1 | explanation | `code example` |

#### D. Key Functions/Commands
- Most important functions, commands, or features
- Present in a table with examples:

| Function/Command | Purpose | Example | Output/Result |
|-----------------|---------|---------|---------------|
| `function()` | what it does | `example_code()` | what you get |

- For each function/command, show:
  - Syntax
  - Parameters and what they do
  - A concrete working example
  - Expected output or result

#### E. Full Working Example
When appropriate, provide a complete, runnable example that demonstrates:
- Multiple features working together
- Real-world use case
- Commented code explaining each step

For tools like ggplot2, show building up a complete visualization. For CLI tools, show a realistic workflow. For algorithms, show implementation and usage.

#### F. Common Patterns & Best Practices
- Typical usage patterns
- Best practices and recommendations
- Common pitfalls to avoid

#### G. References & Further Reading
- Links to official documentation (the pages you consulted)
- Additional resources for deep dives
- Community resources (Stack Overflow tags, discussion forums, GitHub)

### 4. Ask When Unsure

**Never guess or make assumptions.** If you encounter any of the following, ask the user:
- Multiple installation methods and you're unsure which to recommend
- Platform-specific instructions and you don't know the user's environment
- Ambiguous documentation or conflicting information
- Whether to include advanced topics or keep it basic
- If you need more than 5 documentation pages

### 5. Determine File Format

**Use .Rmd format when:**
- The tutorial includes code that should be executed to show output
- Examples need to display visual results (plots, charts, tables)
- The user wants to see the actual output of code execution inline

**Use .md format when:**
- Only showing syntax and code examples
- No need to execute code and display output
- Purely explanatory content

For all files, the yaml front matter should set github_document as the
main format, so that it renders well on GitHub and in markdown
viewers; html_document can be used for more interactive tutorials with
embedded outputs.

### 6. Confirm File Location

Before writing the file, ask the user:
- Suggested filename: `topic-name-tutorial.md` or `topic-name-tutorial.Rmd`
- Suggested location: current working directory
- Ask: "Should I save this as `[suggested_name]` in `[suggested_path]`, or would you prefer a different name/location?"

After confirmation, write the file.

## Example Scenarios

**Scenario 1**: User asks "teach me ggplot2"
1. Use browser-use to search for ggplot2 official documentation
2. Fetch 3-5 key pages (getting started, aesthetics, geoms, themes)
3. Create tutorial covering:
   - Installation (`install.packages("ggplot2")`)
   - Key concepts table (aesthetics, geoms, scales, themes, facets)
   - Key functions table (`ggplot()`, `aes()`, `geom_point()`, `theme()`, etc.)
   - Full working example building a complex plot step by step
   - Common patterns (layering, customization workflow)
   - References
4. Use .Rmd format (since we want to show plots)
5. Ask user to confirm filename and save

**Scenario 2**: User asks "how to use ufw firewall"
1. Use browser-use to search for ufw documentation
2. Fetch installation guide and command reference
3. Create tutorial covering:
   - Installation steps
   - Key concepts (rules, defaults, profiles)
   - Key commands table (`ufw enable`, `ufw status`, `ufw allow`, `ufw deny`)
   - Full working example (securing a web server)
   - Best practices
   - References
4. Use .md format (showing command syntax, not executing)
5. Ask user to confirm filename and save

## Tips for Quality Tutorials

- **Be practical**: Focus on what users need to get started quickly
- **Use examples liberally**: Every function/command should have an example
- **Build up complexity**: Start simple, then show how to combine features
- **Test your understanding**: If you're describing something complex, make sure your explanation is clear
- **Use formatting**: Code blocks, tables, headers to make it scannable
- **Verify accuracy**: Don't invent features or syntax; stick to what's in the documentation

## Topic-Specific Guidance

For certain common topics, additional reference files provide detailed guidance. Check if a reference file exists for the topic:

- `references/r-packages.md` - R package tutorials (ggplot2, dplyr, etc.)
- `references/python-packages.md` - Python package tutorials
- `references/cli-tools.md` - Command-line tools
- `references/cloud-services.md` - AWS, GCP, Azure services

If a reference file exists for the topic category, read it for additional specific guidance before creating the tutorial.
