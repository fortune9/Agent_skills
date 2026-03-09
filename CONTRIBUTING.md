# Contributing to Agent Skills

Thank you for your interest in contributing to the Agent Skills repository! This document provides guidelines for creating and submitting new skills.

---

## Quick Start

1. **Use the skill-creator skill**: We highly recommend using [Anthropic's skill-creator skill](https://github.com/anthropics/skills) to help you build high-quality skills
2. **Check for duplicates**: Search existing skills to avoid duplication
3. **Follow our structure**: Use the directory structure and format described below

---

## Creating a New Skill

### 1. Choose the Right Category

Determine which category your skill belongs to:

| Category | Description |
|----------|-------------|
| **r-lib** | R package development and learning workflows with the r-lib ecosystem |

Feel free to propose new categories if needed.

---

### 2. Skill Structure

Each skill should follow this structure:

```
category-name/
└── your-skill-name/
    ├── SKILL.md              # Required: Main skill file
    ├── references/           # Optional: Supporting documentation
    │   └── guide.md
    ├── scripts/              # Optional: Helper scripts
    │   └── helper.R
    └── templates/            # Optional: Document templates
        └── template.md
```

**Note**: Do NOT create a README.md within individual skill directories. Documentation about skill organization, design principles, or resources should go in the skill category's README.md (e.g., `r-lib/README.md`).

Runnable R scripts should start with a shebang line, include internal usage documentation, use minimal package dependencies, and error if required packages are missing.

```r
#!/usr/bin/env Rscript

# Usage:
# Rscript helper.R <argument>
#
# Arguments:
# argument: Description of the argument
#
# Examples:
# Rscript helper.R "value"

args <- commandArgs(trailingOnly = TRUE)

# Check arguments
if (length(args) < 1) {
  cat("Error: Missing required argument\n")
  cat("Usage: Rscript helper.R <argument>\n")
  quit(status = 1)
}

# Your code here
```

---

### 3. SKILL.md Format

The `SKILL.md` file is the core of your skill. It must include YAML frontmatter:

```markdown
---
name: your-skill-name
description: >
  A clear, concise description of what this skill does and when to use it.
  Focus on the use cases and capabilities. Claude will read this to decide
  when to activate your skill.
---

# Skill Name

[Detailed instructions for Claude on how to execute this skill]

## When to Use This Skill

- Use case 1
- Use case 2
- Use case 3

## Instructions

[Step-by-step instructions Claude should follow]

## Examples

[Real-world examples showing the skill in action]

## Resources

- Links to relevant documentation
- Helper scripts
- Reference materials
```

---

### 4. Writing Effective Skill Instructions

**For Claude, not end users**: Write instructions for Claude to follow, not for human users. Think of it as teaching Claude how to help users.

**Be specific and actionable**: Provide clear, step-by-step instructions that Claude can follow autonomously.

**Include examples**: Show concrete examples of inputs, outputs, and workflows.

**Handle edge cases**: Document how Claude should handle errors, ambiguity, and special situations.

**Reference supporting files**: Use relative paths to reference other files in your skill directory:

```markdown
See `references/guide.md` for detailed formatting requirements.
```

---

### 5. Best Practices

- **Focus on real use cases**: Base your skill on actual needs, not hypothetical scenarios
- **Keep it focused**: One skill should do one thing well. If you find yourself adding many unrelated features, consider splitting into multiple skills
- **Provide comprehensive documentation**: Write clear Claude-facing instructions in SKILL.md
- **Test across platforms**: Verify your skill works in Claude.ai, Claude Code, and via API
- **Use clear naming**: Skill names should be descriptive and use kebab-case
- **Document dependencies**: If your skill requires specific tools or packages, document them clearly
- **Include error handling**: Guide Claude on how to handle common errors

---

### 6. Using the skill-creator Skill

We **strongly recommend** using [Anthropic's skill-creator skill](https://github.com/anthropics/skills) to build your skill. This skill provides:

- Guidance on effective skill structure
- Help with writing clear instructions
- Validation of your skill format
- Best practices and examples

To use the skill-creator:

1. Install the skill from Anthropic's repository
2. Start a conversation with Claude about creating your skill
3. Claude will guide you through the process using skill-creator's expertise

---

## Adding Your Skill to the Repository

### 1. Fork and Clone

```bash
git clone https://github.com/fortune9/Agent_skills.git
cd Agent_skills
```

### 2. Create Your Skill Directory

```bash
mkdir -p category-name/your-skill-name
cd category-name/your-skill-name
```

### 3. Add Your Skill Files

Create your SKILL.md and any supporting files following the structure above.

### 4. Update marketplace.json

Add your skill to the appropriate category plugin in `.claude-plugin/marketplace.json`. Skills are organized by category plugins. Find the plugin that matches your skill's category and add your skill path to its `skills` array:

```json
{
  "name": "category",
  "description": "Collection of skills for [category purpose]",
  "source": "./",
  "strict": false,
  "skills": [
    "./existing-category/existing-skill",
    "./your-category/your-skill-name"  // Add your skill here
  ]
}
```

**Example**: If you're adding a skill to the r-lib category:

```json
{
  "name": "r-lib",
  "description": "Collection of skills for R package developers",
  "source": "./",
  "strict": false,
  "skills": [
    "./r-lib/learn-R",
    "./r-lib/your-new-skill"  // Your new skill
  ]
}
```

**If creating a new category**: Add a new plugin entry to the `plugins` array:

```json
{
  "name": "your-category",
  "description": "Collection of skills for [category purpose]",
  "source": "./",
  "strict": false,
  "skills": [
    "./your-category/your-skill-name"
  ]
}
```

### 5. Update the Skill Category README (Optional)

You may optionally document your skill in the skill category's README.md (e.g., `r-lib/README.md`). This is where you can add:

- Notes about your skill's organization or structure
- Design principles and architectural decisions
- Resources, references, or related documentation used

Not every skill needs to be listed or documented in the group README. Use this space when you have important context to share with future contributors or users.

If you're adding a **new category directory**, create a README.md with this structure:

```markdown
# Category Name Skills

Brief description of what skills in this category do.

## Skills

Brief notes about individual skills (optional - only document when needed):

- **[skill-name](./skill-name/)** - Organization notes, design principles, or key resources

## Common Use Cases

- Use case 1
- Use case 2
```

### 6. Test Your Skill

Before submitting:

1. **Install locally**:
   ```bash
   cp -r category-name/your-skill-name ~/.config/claude-code/skills/
   ```

2. **Test with Claude Code**: Verify Claude activates your skill appropriately

3. **Test edge cases**: Try various inputs and scenarios

4. **Check formatting**: Ensure YAML frontmatter is valid and markdown is well-formatted

### 7. Submit a Pull Request

Open a Pull Request on GitHub with:

- Clear description of what the skill does
- Test results showing it works
- Any dependencies or requirements
- Links to related issues (if applicable)

---

## Pull Request Guidelines

### Required Information

Your PR description should include:

- **Purpose**: What does this skill do?
- **Use cases**: When would someone use this skill?
- **Testing**: How did you test it? What scenarios did you try?
- **Dependencies**: Does it require any specific tools, packages, or configurations?
- **Documentation**: Is the skill well-documented for both Claude and users?

---

## Code of Conduct

This project follows a Code of Conduct. By participating, you agree to:

- Be respectful and inclusive
- Welcome newcomers
- Give and receive constructive feedback gracefully
- Focus on what's best for the community
- Show empathy towards others

---

## Questions?

If you have questions about contributing:

1. Check the [Claude Skills documentation](https://support.claude.com/en/articles/12512198-creating-custom-skills)
2. Look at existing skills for examples
3. Open an issue to discuss your idea
4. Use the skill-creator skill for guidance

---

## Resources

- [Claude Skills Documentation](https://support.claude.com/en/articles/12512180-using-skills-in-claude)
- [Creating Custom Skills Guide](https://support.claude.com/en/articles/12512198-creating-custom-skills)
- [Anthropic's skill-creator](https://github.com/anthropics/skills)
- [Skills API Documentation](https://docs.claude.com/en/api/skills-guide)
- [btw package documentation](https://posit-dev.github.io/btw/)
