# [OpenAI Cookbook Pro](https://chatgpt.com/canvas/shared/6825e9f6e8d88191bf9ef4de00b29b0f)
**An Advanced Implementation Guide to GPT-4.1: Real-World Applications, Prompting Strategies, and Agent Workflows**

Welcome to **OpenAI Cookbook Pro** — a comprehensive, practical, and fully extensible resource tailored for engineers, developers, and researchers working with the GPT-4.1 API and related OpenAI tools. This repository distills best practices, integrates field-tested strategies, and supports high-performing workflows with enhanced reliability, precision, and developer autonomy.

> If you're familiar with the original OpenAI Cookbook, think of this project as an expanded version designed for production-grade deployments, advanced prompt development, tool integration, and agent design.


## 🔧 What This Cookbook Offers

* **Structured examples** of effective prompting for instruction following, planning, tool usage, and dynamic interactions.
* **Agent design frameworks** built around persistent task completion and context-aware iteration.
* **Tool integration patterns** using OpenAI's native tool-calling API — optimized for accuracy and reliability.
* **Custom workflows** for coding tasks, debugging, testing, and patch management.
* **Long-context strategies** including prompt shaping, content selection, and information compression for up to 1M tokens.
* **Production-aligned system prompts** for customer service, support bots, and autonomous coding agents.

Whether you're building an agent to manage codebases or optimizing a high-context knowledge retrieval system, the examples here aim to be direct, reproducible, and extensible.


## 📘 Table of Contents

1. [Getting Started](#getting-started)
2. [Prompting for Instruction Following](#prompting-for-instruction-following)
3. [Designing Agent Workflows](#designing-agent-workflows)
4. [Tool Use and Integration](#tool-use-and-integration)
5. [Chain of Thought and Planning](#chain-of-thought-and-planning)
6. [Handling Long Contexts](#handling-long-contexts)
7. [Code Fixing and Diff Management](#code-fixing-and-diff-management)
8. [Real-World Deployment Scenarios](#real-world-deployment-scenarios)
9. [Prompt Engineering Reference Guide](#prompt-engineering-reference-guide)
10. [API Usage Examples](#api-usage-examples)


## Getting Started

OpenAI Cookbook Pro assumes a basic working knowledge of OpenAI’s Python SDK, the GPT-4.1 API, and how to use the `functions`, `tools`, and `system prompt` fields.

If you're new to OpenAI's tools, start here:

* [OpenAI Platform Documentation](https://platform.openai.com/docs)
* [Original OpenAI Cookbook](https://github.com/openai/openai-cookbook)

This project builds on those foundations, layering in advanced workflows and reproducible examples for:

* Task persistence
* Iterative debugging
* Prompt shaping and behavior targeting
* Multi-step tool planning


## Prompting for Instruction Following

GPT-4.1’s instruction-following capabilities have been significantly improved. To ensure the model performs consistently:

* Be explicit. Literal instruction following means subtle ambiguities may derail output.
* Use clear formatting for instruction sets (Markdown, XML, or numbered lists).
* Place instructions **at both the top and bottom** of long prompts if the context window exceeds 100K tokens.

### Example: Instruction Template

```markdown
# Instructions
1. Read the user’s message carefully.
2. Do not generate a response until you've gathered all needed context.
3. Use a tool if more information is required.
4. Only respond when you can complete the request correctly.
```

> See `/examples/instruction-following.md` for more variations and system prompt styles.


## Designing Agent Workflows

GPT-4.1 supports agentic workflows that require multi-step planning, tool usage, and long turn durations. Designing effective agents starts with a disciplined structure:

### Include Three System Prompt Anchors:

* **Persistence**: Emphasize that the model should continue until task completion.
* **Tool usage**: Make it clear that it must use tools if it lacks context.
* **Planning**: Encourage the model to write out plans and reflect after each action.

See `/agent_design/swe_bench_agent.md` for a complete agent example that solves live bugs in open-source repositories.


## Tool Use and Integration

Leverage the `tools` parameter in OpenAI's API to define functional calls. Avoid embedding tool descriptions in prompts — the model performs better when tools are registered explicitly.

### Tool Guidelines

* Name your tools clearly.
* Keep descriptions concise but specific.
* Provide optional examples in a dedicated `# Examples` section.

> Tool-based prompting increases reliability, reduces hallucinations, and helps maintain output consistency.


## Chain of Thought and Planning

While GPT-4.1 does not inherently perform internal reasoning, it can be prompted to **think out loud**:

```markdown
First, identify what documents may be relevant. Then list their titles and relevance. Finally, provide a list of IDs sorted by importance.
```

Use structured strategies to enforce planning:

1. Break down the query.
2. Retrieve and assess context.
3. Prioritize response steps.
4. Deliver a refined output.

> See `/prompting/chain_of_thought.md` for templates and performance impact.


## Handling Long Contexts

GPT-4.1 supports up to **1 million tokens**. To manage this effectively:

* Use structure: XML or markdown sections help the model parse relevance.
* Repeat critical instructions **at the top and bottom** of your prompt.
* Scope responses by separating external context from user queries.

### Example Format

```xml
<instructions>
Only answer based on External Context. Do not make assumptions.
</instructions>
<user_query>
How does the billing policy apply to usage overages?
</user_query>
<context>
<doc id="12" title="Billing Policy">
[...]
</doc>
</context>
```

> See `/examples/long-context-formatting.md` for formatting guidance.


## Code Fixing and Diff Management

GPT-4.1 includes support for a **tool-compatible diff format** that enables:

* Patch generation
* File updates
* Inline modifications with full context

Use the `apply_patch` tool with the recommended V4A diff format. Always:

* Use clear before/after code snippets
* Avoid relying on line numbers
* Use `@@` markers to indicate scope

> See `/tools/apply_patch_examples/` for real-world patch workflows.


## Real-World Deployment Scenarios

### Use Cases

* **Support automation** using grounded answers and clear tool policies
* **Code refactoring bots** that operate on large repositories
* **Document summarization** across thousands of pages
* **High-integrity report generation** from structured prompt templates

Each scenario includes:

* Prompt formats
* Tool definitions
* Behavior checks

> Explore the `/scenarios/` folder for ready-to-run templates.


## Prompt Engineering Reference Guide

A distilled reference for designing robust prompts across various tasks.

### Sections:

* General prompt structures
* Common failure patterns
* Formatting styles (Markdown, XML, JSON)
* Long-context techniques
* Instruction conflict resolution

> Found in `/reference/prompting_guide.md`


## API Usage Examples

Includes starter scripts and walkthroughs for:

* Tool registration
* Chat prompt design
* Instruction tuning
* Streaming outputs

All examples use official OpenAI SDK patterns and can be run locally.


## Contributing

We welcome contributions that:

* Improve clarity
* Extend agent workflows
* Add new prompt techniques
* Introduce tool examples

To contribute:

1. Fork the repo
2. Create a new folder under `/examples` or `/tools`
3. Submit a PR with a brief description of your addition


## License

This project is released under the MIT License.


## Acknowledgments

This repository builds upon the foundational work of the original [OpenAI Cookbook](https://github.com/openai/openai-cookbook). All strategies are derived from real-world testing, usage analysis, and OpenAI’s 4.1 Prompting Guide (April 2025).


For support or suggestions, feel free to open an issue or connect via [OpenAI Developer Forum](https://community.openai.com).
