# [API Usage Examples with GPT-4.1](https://chatgpt.com/canvas/shared/6825f96694a48191af7648cad2996158)

## Overview

This guide provides detailed, real-world examples of using the OpenAI GPT-4.1 API effectively, with a focus on instruction-following, tool integration, agent persistence, and prompt structuring. These examples are designed to help developers and engineers build resilient, production-ready systems using GPT-4.1 across various applications, including customer service, bug fixing, document analysis, and data labeling.

Each example illustrates system prompt construction, tool schema definitions, interaction workflows, and failure mitigation strategies.



## Example 1: Customer Support Agent with Tool Use

### Objective

Deploy a GPT-4.1 assistant to handle user questions about policies and account status.

### System Prompt

```markdown
You are a helpful assistant for NewTelco. Your job is to assist users with account and policy information.

# Instructions
- Always greet the user.
- Use tools to retrieve account and policy information.
- If needed data is missing, ask the user before calling tools.
- Avoid internal knowledge on restricted topics.

# Workflow
1. Identify intent
2. Call tools if sufficient input
3. Message user before and after tool calls
4. Cite results
```

### Tools

```json
[
  {
    "name": "lookup_policy_document",
    "description": "Searches internal docs by topic",
    "parameters": {
      "type": "object",
      "properties": {
        "topic": {"type": "string"}
      },
      "required": ["topic"]
    }
  },
  {
    "name": "get_user_account_info",
    "description": "Retrieves account details using phone number",
    "parameters": {
      "type": "object",
      "properties": {
        "phone_number": {"type": "string"}
      },
      "required": ["phone_number"]
    }
  }
]
```

### User Input

```text
How much will it cost for international service in France?
```

### Assistant Output

```text
Hi, you've reached NewTelco, how can I help you? 🌍✈️

You're asking about international service while traveling to France. Let me check that for you — one moment, please. 🕑
```

### Tool Call

```json
{
  "name": "lookup_policy_document",
  "arguments": {"topic": "international service cost France"}
}
```



## Example 2: Software Bug Fix Agent with Diff Application

### Objective

Use GPT-4.1 to investigate and fix a bug in a codebase using the `apply_patch` tool.

### System Prompt Highlights

```markdown
# Objective
Fix the issue provided by the user.

# Instructions
- Plan each step
- Reflect after each function call
- Never guess code — read it first using tools
- Only stop when all tests pass

# Workflow
1. Understand issue deeply
2. Investigate codebase
3. Draft patch
4. Apply patch
5. Run tests
6. Reflect and finalize
```

### Tool Definition

```json
{
  "name": "python",
  "description": "Execute code or apply a patch",
  "parameters": {
    "type": "object",
    "properties": {
      "input": {"type": "string"}
    },
    "required": ["input"]
  }
}
```

### Tool Call Example

```bash
%%bash
apply_patch <<"EOF"
*** Begin Patch
*** Update File: src/core.py
@@ def is_valid():
-    return False
+    return True
*** End Patch
EOF
```

### Test Execution

```json
{
  "name": "python",
  "arguments": {"input": "!python3 run_tests.py"}
}
```



## Example 3: Long-Context Document Analyzer

### Objective

Summarize and extract insights from up to 1M tokens of context.

### Prompt Sections

```markdown
# Instructions
- Process documents in 10k token blocks
- Reflect after each segment
- Label relevance and extract core ideas

# Strategy
1. Read → summarize
2. Score relevance
3. Synthesize into unified output
```

### Input Format

```xml
<doc id="21" title="Policy Update">
<summary>Changes to international billing rules</summary>
<content>...</content>
</doc>
```

### Assistant Behavior

* Chunk input into 10k token sections
* After each, provide a summary and document scores
* Compile findings at end



## Example 4: Data Labeling Assistant

### Objective

Assist with structured classification tasks.

### Prompt Template

```markdown
# Instructions
- Label each entry using the provided schema
- Do not guess; if unsure, flag for human

# Labeling Categories
- Urgent
- Normal
- Spam

# Output Format
{"text": ..., "label": ...}

# Example
{"text": "Win money now!", "label": "Spam"}
```

### User Input

```json
[
  "New system update available",
  "Limited time offer! Click now",
  "Server crashed, need help ASAP"
]
```

### Assistant Output

```json
[
  {"text": "New system update available", "label": "Normal"},
  {"text": "Limited time offer! Click now", "label": "Spam"},
  {"text": "Server crashed, need help ASAP", "label": "Urgent"}
]
```



## Example 5: Chain-of-Thought for Multi-Hop Reasoning

### Objective

Support a planning task by explicitly breaking down the steps.

### Prompt Template

```markdown
# Instructions
First, think carefully step by step. Then output the result.

# Reasoning Strategy
1. Identify user question
2. Extract context
3. Connect information across documents
4. Output answer
```

### Example Input

```markdown
# User Question
How did the billing policy change after 2022?

# Context
<doc id="10" title="Policy 2022">...</doc>
<doc id="12" title="Policy 2023">...</doc>
```

### Model Output

```text
Step 1: Identify relevant documents → IDs 10, 12
Step 2: Compare clauses
Step 3: 2022 had flat rates, 2023 added time-of-use billing
Answer: Billing policy changed to time-based pricing in 2023.
```



## General Prompt Formatting Guidelines

### Preferred Structure

```markdown
# Role
# Instructions
# Workflow (optional)
# Reasoning Strategy (optional)
# Output Format
# Examples (optional)
```

### Tool Use Reminders

* Only call tools when sufficient information is available
* Always notify the user before and after calls
* Use example-triggered calls for teaching tool behavior

### Output Patterns

* JSON or markdown preferred
* Cite source documents if used
* Include fallback responses if uncertain (e.g., "Insufficient context")



## Best Practices Summary

| Element      | Best Practice                                 |
| ------------ | --------------------------------------------- |
| Tool Calls   | Always define schema with strong param names  |
| Planning     | Enforce pre- and post-action reflection       |
| Output       | Enforce format, validate JSON before response |
| Long Context | Use structured delimiters (Markdown, XML)     |
| Labeling     | Use few-shot examples and explicit categories |
| Diff Format  | Use V4A patch format for code updates         |



## Final Note

These examples are starting templates. Each system will benefit from iterative refinements, structured logging, and real-world user testing. Maintain modular prompts and tool schemas, and adopt evaluation frameworks to monitor performance over time.

**Clarity, structure, and instruction adherence are the cornerstones of production-grade GPT-4.1 API design.**
