# Agent Skills

A collection of Claude Skills for AI agent development and R programming workflows. Claude Skills extend Claude's capabilities with specialized knowledge and workflows. Skills are automatically activated by Claude based on your task and can be used in Claude.ai, Claude Code, or via the Claude API.

Learn more at the [Claude Skills documentation](https://support.claude.com/en/articles/12512180-using-skills-in-claude).

---

## Available Skills

### R Programming Skills

R package development skills for working with the r-lib ecosystem and modern R workflows.

- **[learn-R](./r-lib/learn-R/)** - Learn R packages and functions using `btw::btw()` for AI agent context generation. This skill provides guidance on generating contextual prompts for learning R packages and functions with the [btw package](https://posit-dev.github.io/btw/). Covers:
  - Using `btw::btw()` to describe R objects (functions, data frames, packages)
  - Integration with ellmer for AI chat sessions
  - Specialized `btw_tool_*()` functions for fine-grained control
  - Best practices for learning R with AI assistants
  - Workflow examples for data transformation, visualization, and statistical modeling

---

## Installation

### Using `npx skills add` (Any Agent)

Install skills from this repository into any supported coding agent (Claude Code, Codex, Cursor, Cline, and [many more](https://github.com/vercel-labs/skills)) using the `npx skills add` CLI:

```bash
# List available skills without installing
npx skills add fortune9/Agent_skills --list

# Install all skills from this repository
npx skills add fortune9/Agent_skills --all

# Install specific skills by name
npx skills add fortune9/Agent_skills --skill learn-R

# Install to Claude Code only, globally
npx skills add fortune9/Agent_skills --agent claude-code --global
```

**Note:** Replace `fortune9` with your actual GitHub username where this repository is hosted.

### Claude Code

#### Method 1: Add Marketplace

Add this repository as a plugin marketplace in Claude Code:

```
/plugin marketplace add fortune9/Agent_skills
```

Then browse and install the skill categories you need through the Claude Code UI.

#### Method 2: Direct Installation

Install specific skill categories directly:

```
/plugin install r-lib@fortune9-agent-skills
```

#### Method 3: Manual Installation

For customization or offline use:

1. Clone this repository:
```bash
git clone https://github.com/fortune9/Agent_skills.git
cd Agent_skills
```

2. Copy individual skills to your Claude Code skills directory:
```bash
cp -r r-lib/learn-R ~/.config/claude-code/skills/
```

3. Restart Claude Code to activate the skill.

### Claude.ai

Skills can be uploaded to Claude.ai following the [Creating Custom Skills guide](https://support.claude.com/en/articles/12512198-creating-custom-skills).

1. Download or clone this repository
2. Zip the skill folder you want to add (e.g., `learn-R`)
3. Upload the zip file to Claude.ai following the guide

### Claude API

Use the [Skills API](https://docs.claude.com/en/api/skills-guide) to programmatically load and manage skills in your applications.

---

## Using Skills

Once installed, Claude will automatically activate relevant skills based on your task. You don't need to explicitly invoke them.

For example, with the `learn-R` skill installed:

```
You: How do I use dplyr::across() with the starwars dataset?
Claude: I'll help you learn about dplyr::across(). Let me gather some context...
```

Claude will use the skill's knowledge to guide you through learning the function with proper context generation using `btw::btw()`.

### Example Workflow

```r
# With the learn-R skill, Claude can help you generate prompts like:
library(btw)
library(ellmer)

chat <- chat_anthropic()
chat$chat(
  btw(
    dplyr::across,
    dplyr::starwars,
    "Create examples using dplyr::across() with the starwars dataset"
  )
)
```

---

## Skill Categories

This repository organizes skills into categories to make it easier to find and install skills relevant to your work:

| Category | Description |
| --------------- | ----------------------------------------------------------- |
| **r-lib** | R package development and learning workflows with the r-lib ecosystem |

---

## Contributing

We welcome contributions! Please see [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines on creating new skills.

**We highly recommend using Anthropic's [skill-creator](https://github.com/anthropics/skills) skill** to help you build high-quality skills.

---

## License

This repository is licensed under the MIT License. See [LICENSE](./LICENSE) for details.

---

## Resources

- [Claude Skills Overview](https://www.anthropic.com/news/skills)
- [Using Skills in Claude](https://support.claude.com/en/articles/12512180-using-skills-in-claude)
- [Creating Custom Skills](https://support.claude.com/en/articles/12512198-creating-custom-skills)
- [Skills API Documentation](https://docs.claude.com/en/api/skills-guide)
- [Anthropic's Official Skills Repository](https://github.com/anthropics/skills)
- [btw package documentation](https://posit-dev.github.io/btw/)

---

## Support

If you have questions or encounter issues, check the [Claude Skills documentation](https://support.claude.com/en/articles/12512180-using-skills-in-claude) or [open an issue](https://github.com/fortune9/Agent_skills/issues/new) on GitHub.

---

**Built with ❤️ + ☕ + 🤖**
