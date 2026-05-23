---
name: zz-vibe-coding
description: Guidelines for coding with AI assistance based on Zhenguo's vibe coding methodology. ONLY use this skill when the user EXPLICITLY requests it (e.g., "use zz-vibe-coding", "apply vibe coding", "/zz-vibe-coding"). Do NOT trigger automatically - this is for testing only.
---

# Vibe Coding Guidelines

These guidelines help you write better code when working with AI assistance, based on lessons learned from real bioinformatics development work.

## Background

[Vibe coding](https://en.wikipedia.org/wiki/Vibe_coding) has become increasingly popular due to advancements in AI technology, particularly AI agents. While it seems anyone can now code with AI assistance, several challenges emerge in practice:

- AI easily generates code you don't want (inefficient or wrong in edge cases)
- Simple tasks get trapped in loops generating overly complicated code
- Code lacks clear strategic architectural design, making it hard to maintain and extend
- Context becomes lengthy and loses focus as development progresses

These guidelines address those challenges systematically.

## Core Principles

### 1. Architecture First

Before writing any code, assess the task complexity:

- **Simple tasks** (single function): Go ahead and implement directly
- **Complex tasks** (multiple components): Design the architecture first
  - Break down into clear, modular subtasks
  - Define interfaces between components
  - Explain the overall strategy before implementing

**Why this matters**: Starting to code without a plan leads to convoluted solutions that are hard to maintain. A few minutes of design saves hours of refactoring.

### 2. Make a Plan

Before coding, write the plan into a file:

- Create a properly named file based on the task (e.g., `implementation-plan.md`)
- Let the user review and approve the plan before starting implementation
- Include the overall strategy, the subtasks, and implementation details for each subtask
- Use the plan as a roadmap during development

**Why this matters**: A written plan ensures alignment with the user's expectations and provides a clear roadmap. It prevents wasted effort on wrong approaches and helps maintain focus throughout development.

### 3. Start Small

Implement each subtask with simple, straightforward code:

- Write the minimal code that solves the problem
- Don't add features that weren't requested
- Avoid premature optimization
- Keep it readable and obvious

**Why this matters**: Overengineered solutions are harder to debug, harder to maintain, and often solve problems that don't exist. Simple code is maintainable code.

### 4. Modularize the Code

Organize code into functions and classes:

- Each function should do one thing well
- Use descriptive names that explain intent
- Keep functions small and focused
- Group related functionality into classes when appropriate

**Why this matters**: Modular code is easier to understand, test, and reuse. It also helps AI assistants provide better context-aware suggestions.

### 5. Use References

Before implementing, consult documentation and best practices:

- Check official documentation for the tools/libraries you're using
- Look for established patterns in the ecosystem
- Ask for references if you're uncertain about the best approach
- Follow industry standards (DRY, SOLID principles)

**Why this matters**: Using established patterns ensures your code is efficient, maintainable, and follows conventions that other developers expect.

### 6. Explain the Code

Add comments to explain the code, starting with high-level strategy and then details for implementation:

- Start with high-level strategy comments
- Explain non-obvious implementation choices
- Document edge cases and assumptions
- The code itself should be self-explanatory for what it does

**Why this matters**: Six months from now, you (or someone else) needs to understand why the code works this way. Good comments preserve that reasoning.

### 7. Test Before Moving On

After implementing each component, verify it works:

- Test individual functions/classes as you write them
- Don't wait until everything is done to start testing
- Catch errors early when they're easier to fix
- Ensure the code works as expected before building on top of it

**Testing methods to use**:
- **Unit tests**: Test individual functions and classes in isolation
- **Integration tests**: Test how different parts of the code work together
- **End-to-end tests**: Test the entire system from start to finish
- **Edge case tests**: Write tests for edge cases and potential failure points

**Testing frameworks**:
- Use **pytest** for Python code
- Use **testthat** for R code

**Why this matters**: Early testing prevents cascading failures. Finding a bug in the foundation after building everything else on top is painful. Different types of tests catch different types of bugs.

### 8. Edit Code Carefully

When modifying existing code:

- Don't change adjacent code unless necessary
- Don't refactor things that aren't broken
- Remove unused imports, variables, and functions after verifying the code works
- Add comments to explain non-obvious changes
- Keep the existing style and conventions

**Why this matters**: Unnecessary changes introduce risk and make code reviews harder. Surgical changes are safer and easier to understand.

### 9. Keep the Context Clean

As development progresses, maintain focus:

- Remove irrelevant information from the conversation
- Focus on the current task
- Use subagents for independent subtasks to avoid context contamination
- Periodically summarize what's been done and what remains

**Why this matters**: A cluttered context leads to unfocused suggestions. Clean context helps AI assistants provide relevant, targeted help.

### 10. Ask If Any Uncertainty

During planning and coding:

- If requirements are unclear, ask for clarification immediately
- Don't make assumptions about what the user wants
- Verify your understanding of the task before implementing
- Confirm architectural decisions for complex tasks

**Why this matters**: Guessing wrong wastes time. A quick question now prevents hours of rework later.

## Workflow Summary

When a task arrives:

1. **Assess complexity**: Is this simple or complex?
2. **For complex tasks**: Design the architecture, break into subtasks
3. **Make a plan**: Write it to a file and get user approval
4. **Check references**: Are there established patterns for this?
5. **Implement simply**: Write clear, minimal code
6. **Test incrementally**: Verify each component works
7. **Keep context clean**: Focus on current task
8. **Ask when uncertain**: Clarify before coding

## Common Pitfalls to Avoid

- **Overengineering simple tasks**: Adding unnecessary complexity
- **Starting without a plan**: Coding before understanding the architecture
- **Changing unrelated code**: Making unnecessary modifications
- **Testing everything at once**: Waiting until the end to verify
- **Assuming requirements**: Guessing instead of asking
- **Losing focus**: Letting the context become cluttered with irrelevant information

## When to Use This Skill

This skill applies to most coding tasks, but is especially valuable for:

- Multi-step implementation tasks
- Refactoring or modifying existing code
- Building new features or components
- Working with unfamiliar libraries or frameworks
- Collaborative development where maintainability matters

Remember: These are guidelines, not rigid rules. Use judgment to adapt them to your specific situation.

## Related Resources

For further reading on effective AI-assisted coding:

- **[Skills for Real Engineers](https://github.com/mattpocock/skills)**: A collection of practical skills for engineers - small but effective
- **[Karpathy-Inspired Claude Code Guidelines](https://github.com/multica-ai/andrej-karpathy-skills)**: High-level guidelines for writing code with AI, inspired by Andrej Karpathy's approach
- **[How to Vibe Coding](https://www.youtube.com/watch?v=ytT4-lGEf6A)**: A video demonstration of using AI agents to develop a Super Mario Bros game
