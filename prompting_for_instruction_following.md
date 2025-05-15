# [Prompting for Instruction Following](https://chatgpt.com/canvas/shared/6825ebe022148191bceb9fa5473a34eb)

## Overview

GPT-4.1 represents a significant shift in how developers should structure prompts for reliable, deterministic, and consistent behavior. Unlike earlier models which often inferred intent liberally, GPT-4.1 adheres to instructions in a far more literal, detail-sensitive manner. This brings both increased control and greater responsibility for developers: well-designed prompts yield exceptional results, while ambiguous or conflicting instructions may result in brittle or unexpected behavior.

This guide outlines best practices, real-world examples, and design patterns to fully utilize GPT-4.1’s instruction-following improvements across a variety of applications. It is structured to help you:

* Understand GPT-4.1’s instruction handling behavior
* Design high-integrity prompt scaffolds
* Debug prompt failures and mitigate ambiguity
* Align instructions with OpenAI’s guidance around tool usage, task persistence, and planning

This file is designed to stand alone for practical use and is fully aligned with the broader `openai-cookbook-pro` repository.

---

## Why Instruction-Following Matters

Instruction following is central to:

* **Agent behavior**: models acting in multi-step environments must reliably interpret commands
* **Tool use**: execution hinges on clearly-defined tool invocation criteria
* **Support workflows**: factual grounding depends on accurate boundary adherence
* **Security and safety**: systems must not misinterpret prohibitions or fail to enforce policy constraints

With GPT-4.1’s shift toward literal interpretation, instruction scaffolding becomes the primary control interface.

---

## GPT-4.1 Instruction Characteristics

### 1. **Literal Compliance**

GPT-4.1 follows instructions with minimal assumption. If a step is missing or unclear, the model is less likely to “fill in” or guess the user’s intent.

* **Previous behavior**: interpreted vague prompts broadly
* **Current behavior**: waits for or requests clarification

This improves safety and traceability but also increases fragility in loosely written prompts.

### 2. **Order-Sensitive Resolution**

When instructions conflict, GPT-4.1 favors those listed **last** in the prompt. This means developers should order rules hierarchically:

* General rules go early
* Specific overrides go later

Example:

```markdown
# Instructions
- Do not guess if unsure
- Use your knowledge if a tool isn’t available
- If both options are available, prefer the tool
```

### 3. **Format-Aware Behavior**

GPT-4.1 performs better with clearly formatted instructions. Prefer structured formats:

* Markdown with headers and lists
* XML with nested tags
* Structured sections like `# Steps`, `# Output Format`

Poorly formatted, unsegmented prompts lead to instruction bleed and undesired merging of behaviors.

---

## Recommended Prompt Structure

Organize your prompt using a structure that mirrors OpenAI’s internal evaluation standards.

### 📁 Standard Sections

```markdown
# Role and Objective
# Instructions
## Sub-categories for Specific Behavior
# Workflow Steps (Optional)
# Output Format
# Examples (Optional)
# Final Reminder
```

### Example Prompt Template

```markdown
# Role and Objective
You are a customer service assistant. Your job is to resolve user issues efficiently, using tools when needed.

# Instructions
- Greet the user politely.
- Use a tool before answering any account-related question.
- If unsure how to proceed, ask the user for clarification.
- If a user requests escalation, refer them to a human agent.

## Output Format
- Always use a friendly tone.
- Format your answer in plain text.
- Include a summary at the end of your response.

## Final Reminder
Do not rely on prior knowledge. Use provided tools and context only.
```

---

## Instruction Categories

### 1. **Task Definition**

Clearly state the model’s job in the opening lines. Be explicit:

✅ “You are an assistant that reviews and edits legal contracts.”

🚫 “Help with contracts.”

### 2. **Behavioral Constraints**

List what the model must or must not do:

* Must call tools before responding to factual queries
* Must ask for clarification if user input is incomplete
* Must not provide financial or legal advice

### 3. **Response Style**

Define tone, length, formality, and structure.

* “Keep responses under 250 words.”
* “Avoid lists unless asked.”
* “Use a neutral tone.”

### 4. **Tool Use Protocols**

Models often hallucinate tools unless guided:

* “If you don’t have enough information to use a tool, ask the user for more.”
* “Always confirm tool usage before responding.”

---

## Debugging Instruction Failures

Instruction-following failures often stem from the following:

### Common Causes

* Ambiguous rule phrasing
* Conflicting instructions (e.g., both asking to guess and not guess)
* Implicit behaviors expected, not stated
* Overloaded instructions without formatting

### Diagnosis Steps

1. Read the full prompt in sequence
2. Identify potential ambiguity
3. Reorder to clarify precedence
4. Break complex rules into atomic steps
5. Test with structured evals

---

## Instruction Layering: The 3-Tier Model

When designing prompts for multi-step tasks, layer your instructions in tiers:

| Tier | Layer Purpose               | Example                                    |
| ---- | --------------------------- | ------------------------------------------ |
| 1    | Role Declaration            | “You are an assistant for legal tasks.”    |
| 2    | Global Behavior Constraints | “Always cite sources.”                     |
| 3    | Task-Specific Instructions  | “In contracts, highlight ambiguous terms.” |

Each layer helps disambiguate behavior and provides a fallback structure if downstream instructions fail.

---

## Long Context Instruction Handling

In prompts exceeding 50,000 tokens:

* Place **key instructions** both **before and after** the context.
* Use format anchors (`# Instructions`, `<rules>`) to signal boundaries.
* Avoid relying solely on the top-of-prompt instructions.

GPT-4.1 is trained to respect these placements, especially when consistent structure is maintained.

---

## Literal vs. Flexible Models

| Capability             | GPT-3.5 / GPT-4-turbo | GPT-4.1         |
| ---------------------- | --------------------- | --------------- |
| Implicit inference     | High                  | Low             |
| Literal compliance     | Moderate              | High            |
| Prompt flexibility     | Higher tolerance      | Lower tolerance |
| Instruction debug cost | Lower                 | Higher          |

GPT-4.1 performs better **when prompts are precise**. Treat prompt engineering as API design — clear, testable, and version-controlled.

---

## Tips for Designing Instruction-Sensitive Prompts

### ✔️ DO:

* Use structured formatting
* Scope behaviors into separate bullet points
* Use examples to anchor expected output
* Rewrite ambiguous instructions into atomic steps
* Add conditionals explicitly (e.g., “if X, then Y”)

### ❌ DON’T:

* Assume the model will “understand what you meant”
* Use overloaded sentences with multiple actions
* Rely on invisible or implied rules
* Assume formatting styles (e.g., bullets) are optional

---

## Example: Instruction-Controlled Code Agent

```markdown
# Objective
You are a code assistant that fixes bugs in open-source projects.

# Instructions
- Always use the tools provided to inspect code.
- Do not make edits unless you have confirmed the bug’s root cause.
- If a change is proposed, validate using tests.
- Do not respond unless the patch is applied.

## Output Format
1. Description of bug
2. Explanation of root cause
3. Tool output (e.g., patch result)
4. Confirmation message

## Final Note
Do not guess. If you are unsure, use tools or ask.
```

> For a complete walkthrough, see `/examples/code-agent-instructions.md`

---

## Instruction Evolution Across Iterations

As your prompts grow, preserve instruction integrity using:

* Versioned templates
* Structured diffs for instruction edits
* Commented rules for traceability

Example diff:

```diff
- Always answer user questions.
+ Only answer user questions after validating tool output.
```

Maintain a changelog for prompts as you would with source code. This ensures instructional integrity during collaborative development.

---

## Testing and Evaluation

Prompt engineering is empirical. Validate instruction design using:

* **A/B tests**: Compare variants with and without behavioral scaffolds
* **Prompt evals**: Use deterministic queries to test edge case behavior
* **Behavioral matrices**: Track compliance with instruction categories

Example matrix:

| Instruction Category | Prompt A Pass | Prompt B Pass |
| -------------------- | ------------- | ------------- |
| Ask if unsure        | ✅             | ❌             |
| Use tools first      | ✅             | ✅             |
| Avoid sensitive data | ❌             | ✅             |

---

## Final Reminders

GPT-4.1 is exceptionally effective **when paired with well-structured, comprehensive instructions**. Follow these principles:

* Instructions should be modular and auditable.
* Avoid unnecessary repetition, but reinforce critical rules.
* Use formatting styles that clearly separate content.
* Assume literalism — write prompts as if programming a function, not chatting with a person.

Every prompt is a contract. GPT-4.1 honors that contract, but only if written clearly.

---

## See Also

* [`Agent Workflows`](../agent_design/swe_bench_agent.md)
* [`Prompt Format Reference`](../reference/prompting_guide.md)
* [`Long Context Strategies`](../examples/long-context-formatting.md)
* [`OpenAI 4.1 Prompting Guide`](https://platform.openai.com/docs/guides/prompting)

---

For questions, suggestions, or prompt design contributions, submit a pull request to `/examples/instruction-following.md` or open an issue in the main repo.
