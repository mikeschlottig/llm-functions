# CLAUDE.md - AI Assistant Guide for LLM Functions

## Project Overview

LLM Functions is a framework for building LLM tools and agents using Bash, JavaScript, and Python. It enables function calling to connect LLMs directly to custom code for executing system commands, processing data, and interacting with APIs.

**Primary use case**: Creating tools and agents for use with [AIChat](https://github.com/sigoden/aichat) CLI.

## Tech Stack

- **Build System**: [argc](https://github.com/sigoden/argc) - Bash command-line framework
- **JSON Processing**: jq
- **Languages**: Bash, JavaScript (Node.js), Python
- **MCP Support**: Model Context Protocol for tool/agent integration
- **CI/CD**: GitHub Actions (Ubuntu, macOS, Windows)

## Project Structure

```
llm-functions/
├── Argcfile.sh              # Main build system and commands
├── tools/                   # Individual tool scripts
│   ├── demo_sh.sh           # Demo tools for each language
│   ├── demo_js.js
│   ├── demo_py.py
│   ├── execute_command.sh   # Execute shell commands
│   ├── fs_*.sh              # File system operations
│   └── web_search_*.sh      # Various web search tools
├── agents/                  # Agent directories
│   ├── coder/               # Code generation agent
│   ├── demo/                # Demo agent
│   ├── todo/                # Todo management agent
│   └── sql/                 # SQL execution agent
├── scripts/                 # Runtime and build scripts
│   ├── run-tool.{sh,js,py}  # Tool execution runners
│   ├── run-agent.{sh,js,py} # Agent execution runners
│   └── build-declarations.* # JSON schema generators
├── mcp/                     # Model Context Protocol
│   ├── server/              # MCP server implementation
│   └── bridge/              # Bridge for external MCP tools
├── docs/                    # Documentation
├── bin/                     # Generated executables (created by build)
└── cache/                   # Runtime cache directory
```

## Key Files

| File | Purpose |
|------|---------|
| `Argcfile.sh` | Main build system with all argc commands |
| `tools.txt` | List of tools to build (user-created) |
| `agents.txt` | List of agents to build (user-created) |
| `functions.json` | Auto-generated function declarations |
| `scripts/run-tool.*` | Tool execution scripts for each language |
| `scripts/run-agent.*` | Agent execution scripts for each language |
| `scripts/build-declarations.*` | Generate JSON schemas from comments |

## Development Setup

### Prerequisites
- [argc](https://github.com/sigoden/argc) - Bash command-line framework
- [jq](https://github.com/jqlang/jq) - JSON processor
- Node.js (for JavaScript tools)
- Python 3.x (for Python tools)

### Quick Start

```bash
# Clone the repository
git clone https://github.com/sigoden/llm-functions
cd llm-functions

# Create tools.txt with desired tools
echo "get_current_weather.sh" > tools.txt
echo "execute_command.sh" >> tools.txt

# Create agents.txt with desired agents
echo "coder" > agents.txt
echo "todo" >> agents.txt

# Build tools and agents
argc build

# Check environment and dependencies
argc check

# Link to AIChat (optional)
argc link-to-aichat
```

## Common Commands

```bash
# Build everything
argc build

# Build only tools
argc build@tool

# Build only agents
argc build@agent

# Run a tool
argc run@tool <tool-name> '<json-args>'

# Run an agent action
argc run@agent <agent-name> <action> '<json-args>'

# Check dependencies and environment
argc check

# Run tests
argc test

# List available tools
argc list@tool

# List available agents
argc list@agent

# Clean build artifacts
argc clean

# Create a new tool script
argc create@tool my_tool.sh param1! param2

# Link web search tool
argc link-web-search web_search_perplexity.sh

# Link code interpreter
argc link-code-interpreter execute_py_code.py

# Show version info
argc version
```

## Testing

### Test Framework
Tests are run via argc commands using the demo tools and agents.

### Run Tests
```bash
argc test                    # Run all tests
argc test@tool               # Test all tools
argc test@agent              # Test all agents
argc test-demo@tool          # Test demo tools only
argc test-demo@agent         # Test demo agents only
```

### CI Environment
GitHub Actions runs tests on Ubuntu, macOS, and Windows with:
- Python 3.11
- Node.js (latest)
- argc and jq installed

## Code Conventions

### Tool Script Pattern

Each tool is a single file in `tools/` directory with comments that generate JSON declarations.

**Bash** (`tools/my_tool.sh`):
```bash
#!/usr/bin/env bash
set -e

# @describe Brief description of the tool
# @option --param1! Required string parameter
# @option --param2  Optional parameter
# @flag --verbose   Boolean flag

main() {
    echo "Result" >> "$LLM_OUTPUT"
}

eval "$(argc --argc-eval "$0" "$@")"
```

**JavaScript** (`tools/my_tool.js`):
```javascript
/**
 * Brief description of the tool
 * @typedef {Object} Args
 * @property {string} param1 - Required parameter
 * @property {string} [param2] - Optional parameter
 * @param {Args} args
 */
exports.run = function({ param1, param2 }) {
    return "Result";
}
```

**Python** (`tools/my_tool.py`):
```python
def run(param1: str, param2: str = None):
    """Brief description of the tool
    Args:
        param1: Required parameter
        param2: Optional parameter
    """
    return "Result"
```

### Agent Structure

Each agent is a directory in `agents/` with:
- `index.yaml` - Agent definition (name, description, instructions, variables)
- `tools.{sh,js,py}` - Agent-specific tool functions
- `tools.txt` - List of shared tools from `tools/` directory
- `functions.json` - Auto-generated declarations

### Parameter Suffixes (for `argc create@tool`)
- `!` - Required parameter
- `*` - Optional array
- `+` - Required array
- (none) - Optional parameter

### Environment Variables

Tools receive these environment variables:
- `LLM_OUTPUT` - File path for tool output
- `LLM_ROOT_DIR` - Path to llm-functions directory
- `LLM_TOOL_NAME` - Current tool name
- `LLM_TOOL_CACHE_DIR` - Tool-specific cache directory
- `LLM_AGENT_NAME` - Agent name (for agent tools)
- `LLM_AGENT_VAR_<NAME>` - Agent variables

## When Making Changes

1. **Follow comment conventions** - Tool declarations are auto-generated from comments
2. **Use the demo tools as reference** - See `tools/demo_{sh,js,py}.*` for examples
3. **Test after changes** - Run `argc test` to verify everything works
4. **Build after adding tools/agents** - Run `argc build` to regenerate declarations
5. **Check dependencies** - Run `argc check` to verify environment variables and dependencies
6. **Support multiple languages** - Consider implementing tools in all three languages for flexibility

### Adding a New Tool

1. Create tool script in `tools/` with proper comments
2. Add filename to `tools.txt`
3. Run `argc build@tool`
4. Test with `argc run@tool <tool-name> '<json>'`

### Adding a New Agent

1. Create directory in `agents/<agent-name>/`
2. Create `index.yaml` with metadata and instructions
3. Create `tools.{sh,js,py}` with agent functions
4. Optionally create `tools.txt` for shared tools
5. Add agent name to `agents.txt`
6. Run `argc build@agent`
7. Test with `argc run@agent <agent-name> <action> '<json>'`

## MCP Integration

- **mcp/server** - Expose LLM Functions tools/agents via Model Context Protocol
- **mcp/bridge** - Use external MCP tools within LLM Functions

## Common Issues

- **Missing argc/jq**: Install these prerequisites first
- **Tool not found**: Ensure tool is listed in `tools.txt` and run `argc build`
- **Environment variables missing**: Check with `argc check` and set required vars in `.env`
- **Python dependencies**: Create `.venv` and install required packages
- **Permission errors**: Ensure scripts have execute permissions
