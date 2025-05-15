# [Tool Use and Integration](https://chatgpt.com/canvas/shared/6825ee7dbfd081919e67bd643748f8de)

## Overview

GPT-4.1 introduces robust capabilities for working with tools directly through the OpenAI API’s `tools` parameter. Rather than relying solely on the model's internal knowledge, developers can now extend functionality, reduce hallucination, and enforce reliable workflows by integrating explicitly defined tools into their applications.

This document offers a comprehensive guide for designing and deploying tool-augmented applications using GPT-4.1. It includes best practices for tool registration, prompting strategies, tool schema design, usage examples, and debugging common tool invocation failures. Each section is modular and designed to help you build reliable systems that scale across contexts, task types, and user interfaces.

---

## What is a Tool in GPT-4.1?

A **tool** is an explicitly defined function or utility passed to the GPT-4.1 API, allowing the model to trigger predefined operations such as:

* Running code or bash commands
* Retrieving documents or structured data
* Performing API calls
* Applying file patches or diffs
* Looking up user account information

Tools are defined in a structured JSON schema and passed via the `tools` parameter. When the model determines a tool is required, it emits a function call rather than plain text. This enables **precise execution**, **auditable behavior**, and **tight application integration**.

---

## Why Use Tools?

| Benefit                        | Description                                                                |
| ------------------------------ | -------------------------------------------------------------------------- |
| **Reduces hallucination**      | Encourages the model to call real-world functions instead of guessing      |
| **Improves traceability**      | Tool calls are logged and interpretable as function outputs                |
| **Enables complex workflows**  | Offloads parts of the task to external systems (e.g., shell, Python, APIs) |
| **Enhances compliance**        | Limits model responses to grounded tool outputs                            |
| **Improves agent performance** | Required for persistent, multi-turn agentic workflows                      |

---

## Tool Definition: The Schema

Tools are defined using a JSON schema object that includes:

* `name`: A short, unique identifier
* `description`: A concise explanation of what the tool does
* `parameters`: A standard JSON Schema describing expected input

### Example: Python Execution Tool

```json
{
  "type": "function",
  "name": "python",
  "description": "Run Python code or terminal commands in a secure environment.",
  "parameters": {
    "type": "object",
    "properties": {
      "input": {
        "type": "string",
        "description": "The code or command to run"
      }
    },
    "required": ["input"]
  }
}
```

### Best Practices for Schema Design

* Use clear names: `run_tests`, `lookup_policy`, `apply_patch`
* Keep descriptions actionable: Describe *when* and *why* to use
* Minimize complexity: Use shallow parameter objects where possible
* Use enums or constraints to reduce ambiguous calls

---

## Registering Tools in the API

In the Python SDK:

```python
response = client.chat.completions.create(
    model="gpt-4.1",
    messages=chat_history,
    tools=[python_tool, get_user_info_tool],
    tool_choice="auto"
)
```

Set `tool_choice` to:

* `"auto"`: Allow the model to choose when to call
* A specific tool name: Force one call
* `"none"`: Prevent tool usage (useful for testing)

---

## Prompting for Tool Use

### Tool Use Prompting Guidelines

To guide GPT-4.1 toward proper tool usage:

* **Don’t rely on the model to infer when to call a tool.** Tell it explicitly when tools are required.
* **Prompt for failure cases**: Tell the model what to do when it lacks information (e.g., “ask the user” or “pause”).
* **Avoid ambiguity**: Be clear about tool invocation order and data requirements.

### Example Prompt Snippet

```markdown
Before answering any user question about billing, check if the necessary context is available.
If not, use the `lookup_policy_document` tool to find relevant information.
Never answer without citing a retrieved document.
```

### Escalation Pattern

```markdown
If the tool fails to return the necessary data, ask the user for clarification.
If the user cannot provide it, explain the limitation and pause further action.
```

---

## Tool Use in Agent Workflows

Tool usage is foundational to agent design in GPT-4.1.

### Multi-Stage Task Example: Bug Fix Agent

```markdown
1. Use `read_file` to inspect code
2. Analyze and plan a fix
3. Use `apply_patch` to update the file
4. Use `run_tests` to verify changes
5. Reflect and reattempt if needed
```

Each tool call is logged as a JSON event and can be parsed programmatically.

---

## Apply Patch: Recommended Format

One of the most powerful GPT-4.1 patterns is **patch generation** using a diff-like format.

### Patch Structure

```bash
apply_patch <<"EOF"
*** Begin Patch
*** Update File: path/to/file.py
@@ def function():
-    old_code()
+    new_code()
*** End Patch
EOF
```

### Tool Behavior

* No line numbers required
* Context determined by `@@` anchors and 3 lines of code before/after
* Errors must be handled gracefully and logged

See `/examples/apply_patch/` for templates and error-handling techniques.

---

## Tool Examples by Use Case

| Use Case              | Tool Name       | Description                                |
| --------------------- | --------------- | ------------------------------------------ |
| Execute code          | `python`        | Runs code or shell commands                |
| Apply file diff       | `apply_patch`   | Applies a patch to a source file           |
| Fetch document        | `lookup_policy` | Retrieves structured policy text           |
| Get user account data | `get_user_info` | Fetches user account info via phone number |
| Log analytics         | `log_event`     | Sends metadata to your analytics platform  |

---

## Error Handling and Recovery

Tool failure is inevitable in complex systems. Plan for it.

### Guidelines for GPT-4.1:

* Detect and summarize tool errors
* Ask for missing input
* Retry if safe
* Escalate to user if unresolvable

### Prompt Pattern: Failure Response

```markdown
If a tool fails with an error, summarize the issue clearly for the user.
Only retry if the cause of failure is known and correctable.
If not, explain the problem and ask the user for next steps.
```

---

## Tool Debugging and Logging

Enable structured logging to track model-tool interactions:

* **Log call attempts**: Include input parameters and timestamps
* **Log success/failure outcomes**: Include model reflections
* **Log retry logic**: Show how failures were handled

This creates full traceability for AI-involved actions.

### Sample Tool Call Log (JSON)

```json
{
  "tool_name": "run_tests",
  "input": "!python3 -m unittest discover",
  "result": "3 tests passed, 1 failed",
  "timestamp": "2025-05-15T14:32:12Z"
}
```

---

## Tool Evaluation and Performance Monitoring

Track tool usage metrics:

* **Tool Call Rate**: How often a tool is invoked
* **Tool Completion Rate**: How often tools finish without failure
* **Tool Contribution Score**: Impact on final task completion
* **Average Attempts per Task**: Retry behavior over time

Use this data to refine prompting and improve tool schema design.

---

## Common Pitfalls and Solutions

| Issue                        | Likely Cause                                   | Solution                                             |
| ---------------------------- | ---------------------------------------------- | ---------------------------------------------------- |
| Tool called with empty input | Missing required parameter                     | Prompt model to validate input presence              |
| Tool ignored                 | Tool not described clearly in schema or prompt | Add clear instruction for when to use tool           |
| Repeated failed calls        | No failure mitigation logic                    | Add conditionals to check and respond to tool errors |
| Model mixes tool names       | Ambiguous tool naming                          | Use short, specific, unambiguous names               |

---

## Combining Tools with Instructions

When combining tools with detailed instruction sets:

* Include a `# Tools` section in your system prompt
* Define when and why each tool should be used
* Link tool calls to reasoning steps in `# Workflow`

### Example Combined Prompt

```markdown
# Role
You are a bug-fix agent using provided tools to solve code issues.

# Tools
- `read_file`: Inspect code files
- `apply_patch`: Apply structured diffs
- `run_tests`: Validate code after changes

# Instructions
1. Always start with file inspection
2. Plan before making changes
3. Test after every patch
4. Do not finish until all tests pass

# Output
Include patch summaries, test outcomes, and current status.
```

---

## Tool Testing Templates

Create test cases that validate:

* Input formatting
* Response validation
* Prompt-tool alignment
* Handling of edge cases

Use both synthetic and real examples:

```markdown
## Tool Call Test: run_tests
**Input**: Code with known error
**Expected Output**: Test failure summary
**Follow-up Behavior**: Retry with fixed patch
```

---

## Tool Choice Design

Choose between model-directed or developer-directed tool invocation:

| Mode          | Behavior                                    | Use Case                           |
| ------------- | ------------------------------------------- | ---------------------------------- |
| `auto`        | Model decides whether and when to use tools | General assistants, exploration    |
| `none`        | Model cannot use tools                      | Testing model reasoning only       |
| `forced` name | Developer instructs tool call immediately   | Known pipeline steps, unit testing |

Choose based on control needs and task constraints.

---

## Summary: Best Practices for Tool Integration

| Area             | Best Practice                                            |
| ---------------- | -------------------------------------------------------- |
| Tool Naming      | Use action-based, unambiguous names                      |
| Prompt Structure | Clearly define when and how tools should be used         |
| Tool Invocation  | Register tools in API, not in plain prompt text          |
| Failure Handling | Provide instructions for retrying or asking the user     |
| Schema Design    | Use JSON Schema with constraints to reduce invalid input |
| Evaluation       | Track tool call success rate and contribution to outcome |

---

## Further Exploration

* [`Designing Agent Workflows`](./Designing%20Agent%20Workflows.md)
* [`Prompting for Instruction Following`](./Prompting%20for%20Instruction%20Following.md)
* [`Long Context Strategies`](./Long%20Context.md)

For community templates and tool libraries, explore the `/tools/` and `/examples/` directories in the main repository.

---

For contributions, open a pull request or submit an issue in `/tools/Tool Use and Integration.md`.
