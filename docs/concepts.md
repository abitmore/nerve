## Concepts

### What is Nerve?
Nerve is an **Agent Development Kit (ADK)** designed to let you build intelligent agents using Large Language Models (LLMs) with minimal effort. It provides a declarative YAML-based syntax, powerful CLI tools, and optional integration with the Model Context Protocol (MCP).

Nerve is designed for developers and cybersecurity professionals who:
- Are comfortable with the terminal, Python, and Git.
- Are curious about LLMs but don’t want to build everything from scratch.
- Want to create programmable agents rather than chatbots.

### Agent
An **agent** is a YAML file that defines:
- The agent's "role" (system prompt)
- A task (objective or behavior)
- The tools it can use (e.g., shell commands, HTTP requests, Python functions)

Agents run in a loop, invoking tools and modifying state until they complete or fail their task.

```yaml
agent: You are a cybersecurity assistant.
task: Scan the system for open ports.
using:
  - shell
```

### Prompt Interpolation and Variables
Nerve supports [Jinja2](https://jinja.palletsprojects.com/) templating for dynamic prompt construction. You can:
- Inject command line arguments (`{{ target }}`)
- Use tool outputs (`{{ get_logs_tool() }}`)
- Include external files (`{% include 'prompt.md' %}`)
- Reference built-in variables like `{{ CURRENT_DATE }}` or `{{ LOCAL_IP }}`

### Tools
Tools extend the agent’s capabilities. They can be:
- **Shell commands** (interpolated into a shell script)
- **Python functions** (via annotated `tools.py` files)
- **Remote tools via MCP** (from another Nerve instance or a compatible server)

Tools must be documented with a description and arguments:
```yaml
tools:
  - name: get_weather
    description: Get weather info for a city.
    arguments:
      - name: city
        description: Name of the city.
        example: Rome
    tool: curl wttr.in/{{ city }}
```

### Workflows
A **workflow** is a YAML file that chains multiple agents sequentially. Each agent in the pipeline can:
- Use a different model
- Receive input from the previous agent
- Define its own tools and prompt

This is useful for simple **linear pipelines** — for example:
```yaml
actors:
  step1: { generator: openai://gpt-4o }
  step2: { generator: anthropic://claude }
```
Each agent is executed in order, with shared state passed between them. Read more about [workflows in the specific page](workflows.md).

> For more complex orchestrations (e.g., branching logic, sub-agents, delegation), it's better to use **Nerve as an MCP server**. This way, agents can expose themselves as tools to a primary orchestrator agent. Refer to the [MCP documentation](mcp.md).

### Evaluation
Nerve supports **agent evaluation** with test cases to validate correctness, track regressions, or benchmark models.
You define input cases (via YAML, Parquet, or folders), and Nerve runs the agent against them.

```bash
nerve eval path/to/evaluation --output results.json
```

Read more in the [dedicated page](evaluation.md).

### MCP (Model Context Protocol)
Nerve can:
- **Use MCP tools**: connect to external memory, filesystem, or custom tool servers
- **Expose agents as MCP servers**: let other agents call them as tools

```yaml
mcp:
  memory:
    command: npx
    args: ["-y", "@modelcontextprotocol/server-memory"]
```

Read more in the [dedicated page](mcp.md).

### Diagram: Nerve Agent Execution (simplified)

```mermaid
graph TD
    A[Start Agent] --> B[Inject Prompt]
    B --> C{Tools Required?}
    C -- Yes --> D[Call Tool]
    D --> E[Update State]
    E --> C
    C -- No --> F{Task Complete?}
    F -- Yes --> G[Exit]
    F -- No --> B
```

This loop continues until the task is complete, failed, or times out.

