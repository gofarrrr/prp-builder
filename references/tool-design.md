# Tool Design Principles

How to design, describe, and compose tools for effective agent use.

---

## The Golden Rule

> "If a human engineer cannot definitively say which tool to use based on the description, an agent cannot do better."

Tool success depends more on **description quality** than implementation sophistication.

---

## Tool Naming

### Principles

| Principle | Good | Bad |
|-----------|------|-----|
| Action-first | `search_files`, `create_user` | `file_searcher`, `user_creator` |
| Specific | `search_code_content` | `search` |
| Consistent | `get_user`, `get_order` | `fetch_user`, `retrieve_order` |
| No abbreviations | `execute_database_query` | `exec_db_qry` |

### Naming Patterns

```yaml
# Read operations
get_{resource}        # get_user, get_config
list_{resources}      # list_files, list_users
search_{scope}        # search_codebase, search_docs

# Write operations
create_{resource}     # create_file, create_branch
update_{resource}     # update_config, update_user
delete_{resource}     # delete_file, delete_cache

# Actions
run_{action}          # run_tests, run_lint
execute_{operation}   # execute_query, execute_script
validate_{target}     # validate_schema, validate_config
```

---

## Tool Descriptions

### Anatomy of Good Description

```yaml
tool:
  name: search_codebase
  description: |
    Search for patterns in source code files.

    USE WHEN:
    - Finding implementations of a class or function
    - Locating where a variable is used
    - Discovering files matching a pattern

    DO NOT USE WHEN:
    - You need full file content (use read_file instead)
    - Searching documentation (use search_docs instead)
    - Finding files by name only (use list_files with glob)

    EXAMPLES:
    - "class.*Service" finds all service classes
    - "import pandas" finds files using pandas
    - "def test_" finds all test functions

    RETURNS:
    - List of matches with file path, line number, context

    LIMITATIONS:
    - Max 100 results per query
    - Binary files are skipped
    - Regex syntax required
```

### Description Checklist

- [ ] **Purpose** - What does this tool do?
- [ ] **When to use** - Specific scenarios
- [ ] **When NOT to use** - Common misuses
- [ ] **Examples** - Concrete usage
- [ ] **Output format** - What to expect
- [ ] **Limitations** - What it can't do

### Bad vs Good Descriptions

```yaml
# Bad: Vague, no guidance
tool:
  name: search
  description: "Searches for things"

# Good: Specific, actionable
tool:
  name: search_code_content
  description: |
    Search source code files for regex patterns.
    Returns matching lines with file paths and line numbers.

    Use for: finding implementations, locating usage, discovering patterns
    Not for: reading full files, searching docs, binary files

    Example: "class.*Repository" finds repository classes
    Limit: 100 results max
```

---

## Parameter Design

### Principles

1. **Required vs Optional** - Minimize required, sensible defaults for optional
2. **Type Safety** - Use specific types, enums where possible
3. **Validation** - Document constraints in description
4. **Examples** - Show valid values

### Parameter Patterns

```yaml
parameters:
  # Required with clear purpose
  query:
    type: string
    required: true
    description: "Regex pattern to search for"
    examples: ["class.*Service", "import\\s+os"]

  # Optional with sensible default
  path:
    type: string
    required: false
    default: "."
    description: "Directory to search in (default: current directory)"

  # Enum for limited choices
  file_type:
    type: string
    required: false
    enum: ["py", "js", "ts", "go", "all"]
    default: "all"
    description: "Filter by file extension"

  # Numeric with bounds
  max_results:
    type: integer
    required: false
    default: 50
    minimum: 1
    maximum: 100
    description: "Maximum number of results to return"
```

---

## Error Handling

### Error Response Design

```yaml
error_response:
  structure:
    success: false
    error:
      code: "FILE_NOT_FOUND"
      message: "The file 'src/missing.py' does not exist"
      suggestion: "Check file path, use list_files to find available files"
      recoverable: true

  codes:
    FILE_NOT_FOUND:
      recoverable: true
      suggestion: "Verify path with list_files"

    PERMISSION_DENIED:
      recoverable: false
      suggestion: "Request user to grant permission"

    RATE_LIMITED:
      recoverable: true
      suggestion: "Wait and retry after {retry_after} seconds"

    TIMEOUT:
      recoverable: true
      suggestion: "Reduce scope or increase timeout"
```

### PRP Application

```markdown
## Tool Error Handling

### Task 2: Search for patterns
- TOOL: `search_code_content "class.*Service" src/`
- IF_ERROR FILE_NOT_FOUND: Verify src/ exists with `list_files`
- IF_ERROR NO_MATCHES: Try broader pattern or different directory
- IF_ERROR TIMEOUT: Search smaller directory subset
```

---

## Tool Composition

### Sequential Composition

```yaml
# Tools used in sequence
workflow:
  - tool: list_files
    purpose: "Find relevant files"
    output: file_paths

  - tool: read_file
    purpose: "Load file content"
    input: file_paths[0]
    output: content

  - tool: search_code_content
    purpose: "Find specific pattern in file"
    input: content
```

### Parallel Composition

```yaml
# Tools used in parallel
workflow:
  parallel:
    - tool: search_code_content
      args: {query: "class.*Service", path: "src/services/"}

    - tool: search_code_content
      args: {query: "def test_", path: "tests/"}

    - tool: fetch_documentation
      args: {url: "https://docs.example.com/api"}

  merge: "Combine all results"
```

### Conditional Composition

```yaml
# Tool choice based on condition
workflow:
  - condition: "file_extension == 'py'"
    tool: run_pytest
    args: {path: "{file}"}

  - condition: "file_extension == 'js'"
    tool: run_jest
    args: {path: "{file}"}

  - condition: "else"
    tool: echo
    args: {message: "No test runner for this file type"}
```

---

## MCP Integration

### Tool Definition for MCP

```yaml
mcp_tool:
  name: analyze_code_patterns
  description: |
    Analyzes source code to extract design patterns and conventions.
    Returns structured analysis of coding patterns, naming conventions,
    and architectural decisions.

    Use when: Starting a new PRP, need to understand existing code style
    Not for: Simple file reads, specific searches

  input_schema:
    type: object
    properties:
      directory:
        type: string
        description: "Root directory to analyze"
      depth:
        type: integer
        description: "How deep to analyze (1=surface, 3=detailed)"
        default: 2
      focus:
        type: array
        items:
          type: string
          enum: ["naming", "structure", "patterns", "testing"]
        description: "Aspects to focus analysis on"
    required: ["directory"]
```

### MCP Server Patterns

```yaml
mcp_server_design:
  # Group related tools
  resource_groups:
    codebase:
      - search_code_content
      - list_files
      - read_file
      - analyze_patterns

    documentation:
      - search_docs
      - fetch_url
      - read_markdown

    execution:
      - run_command
      - run_tests
      - run_lint

  # Share context within group
  shared_context:
    codebase:
      - project_root
      - file_tree_cache
      - pattern_cache
```

---

## Tool Selection Guide

Help agents choose the right tool:

```yaml
selection_guide:
  "I need to find files":
    - name_pattern_known: list_files with glob
    - content_pattern_known: search_code_content
    - unsure_what_exists: list_files then search

  "I need file content":
    - specific_file: read_file
    - specific_lines: read_file with line_range
    - pattern_in_file: search_code_content

  "I need to run code":
    - tests: run_tests
    - linting: run_lint
    - arbitrary_command: run_command (with caution)

  "I need external info":
    - library_docs: fetch_documentation
    - api_reference: fetch_url
    - search_web: web_search
```

---

## PRP Tool Section

How to specify tools in PRPs:

```markdown
## Available Tools

### Required for This PRP
- `search_code_content` - Find patterns in codebase
- `read_file` - Load specific files
- `write_file` - Create/modify files
- `run_tests` - Execute pytest

### Optional (Load if Needed)
- `fetch_documentation` - External library docs
- `run_lint` - Code style validation

### Tool Usage Pattern
```yaml
task_pattern:
  discovery:
    - list_files → identify relevant files
    - search_code_content → find patterns

  implementation:
    - read_file → load pattern reference
    - write_file → create new file

  validation:
    - run_lint → check style
    - run_tests → verify functionality
```
```

---

## Tool Design Checklist

When designing or documenting tools:

- [ ] Name follows action-first convention?
- [ ] Description includes when to use?
- [ ] Description includes when NOT to use?
- [ ] Examples are concrete and helpful?
- [ ] Parameters have sensible defaults?
- [ ] Error codes are documented?
- [ ] Recovery suggestions are provided?
- [ ] Limitations are stated?
