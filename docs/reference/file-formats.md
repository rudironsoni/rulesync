# File Formats

Complete reference for all Rulesync file formats.

---

## `rulesync/rules/*.md`

Rules define AI coding guidelines and constraints.

### Comprehensive Example

```yaml
---
# Canonical options (v1.0.0+)
root:
  true # v1.0.0 - **Required.** Boolean. Is this the root/overview rule file?
  # - `true`: This is the main overview file (e.g., AGENTS.md for agentsmd,
  #   CLAUDE.md for claudecode). Only one root file should exist per tool.
  # - `false`: This is a detail/context file (e.g., .agents/memories/*.md).
  #   Can have multiple detail files.
  # The root file is what the AI tool loads by default.
localRoot:
  false # v2.0.0 - Boolean. Project-specific local rules. Default: false.
  # - `true`: Creates/updates local rules file (for Claude Code: generates
  #   CLAUDE.local.md; for other tools: appends to root file)
  # - `false`: Updates the main configuration file
  # Use case: Project-specific overrides that shouldn't be committed
  # (e.g., personal preferences, local-only settings)
targets: # v1.0.0 - Array. Target AI tools. Default: ["*"].
  - "*" # - "*": Applies to all supported tools
    # - Specific tools: ["claudecode", "cursor", "copilot"]
    # Note: See Supported Tools section for complete list
description:
  "Project overview" # v1.0.0 - String. Brief summary of what this rule does.
  # - Shown in tool UIs where applicable
  # - Should briefly explain the rule's purpose
  # Example: "TypeScript coding standards for backend services"
globs: # v1.0.0 - Array. File patterns this rule applies to.
  - "**/*" # - Standard glob patterns supported (** , *, ?, character classes)
  - "!node_modules/**" # - Negation with ! prefix
  - "!.git/**" # Examples:
    #   - ["**/*.ts"] - All TypeScript files
    #   - ["src/**/*.ts", "!**/*.test.ts"] - Source files excluding tests
    #   - ["**/*", "!node_modules/**", "!.git/**"] - Everything except
    #     node_modules and git

# agentsmd specific (v2.5.0+)
agentsmd:
  subprojectPath:
    "packages/frontend" # v2.5.0 - String. Path to subproject for nested AGENTS.md support.
    # - Only valid when root: false
    # - Defines the subproject directory this rule applies to
    # - Enables monorepo support with nested agent configurations
    # Example: "packages/frontend" or "apps/api"

# claudecode specific (v3.0.0+)
claudecode:
  paths: # v3.0.0 - Array. Glob patterns for conditional rules.
    - "src/**/*.ts" # - Takes precedence over the canonical globs field
    - "tests/**/*.test.ts" # - Claude Code-specific file matching
      # Example: ["src/**/*.ts", "tests/**/*.test.ts"]

# cursor specific (v1.0.0+)
cursor:
  alwaysApply:
    true # v1.0.0 - Boolean. Whether rule always applies in Cursor. Default: false.
    # - `true`: Rule is applied globally regardless of file context
    # - `false`: Rule only applies based on globs matching
    # Use case: Global coding standards that should always be active
  description:
    "Cursor-specific" # v1.0.0 - String. Description shown in Cursor's rule UI.
    # - Displayed in the Cursor rule picker/manager
    # - Should be concise but descriptive
    # Example: "React Component Standards"
  globs: # v1.0.0 - Array. Cursor-specific file patterns.
    - "*.ts" # - Overrides canonical globs for Cursor specifically
    - "*.tsx" # - Same pattern syntax as canonical globs
      # Example: ["*.tsx", "*.ts"]

# copilot specific (v4.0.0+)
copilot:
  excludeAgent:
    "code-review" # v4.0.0 - String. Agent to exclude this rule from.
    # - "code-review": Exclude from Copilot code review agent
    # - "coding-agent": Exclude from Copilot coding agent
    # Use case: Rules that should only apply to one agent type

# antigravity specific (v2.0.0+)
antigravity:
  trigger:
    "glob" # v2.0.0 - String. Trigger mode. Default: "always_on".
    # - "always_on": Always active
    # - "glob": Active when files matching globs are involved
    # - "manual": Must be manually invoked
    # - "model_decision": AI decides when to apply based on description
  globs: # v2.0.0 - Array. File patterns. Required when trigger is "glob".
    - "src/**/*.rs" # - Defines which files trigger this rule
    - "Cargo.toml" # - Same glob syntax as canonical globs
      # Example: ["src/**/*.rs", "Cargo.toml"]
  description:
    "Apply for Rust" # v2.0.0 - String. Description for "model_decision" trigger.
    # - Helps the AI understand when to apply this rule
    # - Should describe the context/use case
    # Example: "Apply when working with Rust async code"
---
# Rule body content goes here
```

---

## `.rulesync/hooks.json`

Lifecycle hooks run scripts at specific events.

Events use **canonical camelCase** (e.g., `sessionStart`) in Rulesync configuration. Tools map these to their native event names:

- Cursor: uses camelCase as-is
- Claude Code: converts to PascalCase (e.g., `SessionStart`)
- OpenCode: maps to dot notation (e.g., `session.created`)
- Copilot: maps canonical events to tool-specific names (e.g., `afterSubmitPrompt` → `userPromptSubmitted`, `afterError` → `errorOccurred`)
- Factory Droid: converts to PascalCase (e.g., `sessionStart` → `SessionStart`)

### Supported Events by Tool

| Event                  | Cursor | Claude Code | OpenCode | Copilot | Factory Droid |
| ---------------------- | ------ | ----------- | -------- | ------- | ------------- |
| `sessionStart`         | ✓      | ✓           | ✓        | ✓       | ✓             |
| `sessionEnd`           | ✓      | ✓           |          | ✓       | ✓             |
| `preToolUse`           | ✓      | ✓           | ✓        | ✓       | ✓             |
| `postToolUse`          | ✓      | ✓           | ✓        | ✓       | ✓             |
| `stop`                 | ✓      | ✓           | ✓        |         | ✓             |
| `beforeSubmitPrompt`   | ✓      | ✓           |          |         | ✓             |
| `subagentStop`         | ✓      | ✓           |          |         | ✓             |
| `preCompact`           | ✓      | ✓           |          |         | ✓             |
| `postToolUseFailure`   | ✓      | -           | -        | -       | -             |
| `subagentStart`        | ✓      | -           | -        | -       | -             |
| `beforeShellExecution` | ✓      | -           | -        | -       | ✓             |
| `afterShellExecution`  | ✓      | -           | ✓        | -       | ✓             |
| `beforeMCPExecution`   | ✓      | -           | -        | -       | -             |
| `afterMCPExecution`    | ✓      | -           | -        | -       | -             |
| `beforeReadFile`       | ✓      | -           | -        | -       | -             |
| `afterFileEdit`        | ✓      | -           | ✓        | -       | ✓             |
| `afterAgentResponse`   | ✓      | -           | -        | -       | -             |
| `afterAgentThought`    | ✓      | -           | -        | -       | -             |
| `beforeTabFileRead`    | ✓      | -           | -        | -       | -             |
| `afterTabFileEdit`     | ✓      | -           | -        | -       | -             |
| `permissionRequest`    | -      | ✓           | ✓        | -       | ✓             |
| `notification`         | -      | ✓           | -        | -       | ✓             |
| `setup`                | -      | ✓           | -        | -       | ✓             |
| `afterSubmitPrompt`    | -      | -           | -        | ✓       | -             |
| `afterError`           | -      | -           | -        | ✓       | -             |

---

### Event Name Mappings by Tool

| Canonical Event          | Cursor               | Claude Code        | OpenCode        | Copilot               | Factory Droid      |
| ------------------------ | -------------------- | ------------------ | --------------- | --------------------- | ------------------ |
| **Session Start**        | `sessionStart`       | `SessionStart`     | `session.start` | `sessionStart`        | `SessionStart`     |
| **Session End**          | `sessionEnd`         | `SessionEnd`       | —               | `sessionEnd`          | `SessionEnd`       |
| **Stop**                 | `stop`               | `Stop`             | `session.idle`  | —                     | `Stop`             |
| **Before Submit Prompt** | `beforeSubmitPrompt` | `UserPromptSubmit` | —               | —                     | `UserPromptSubmit` |
| **After Submit Prompt**  | —                    | —                  | —               | `userPromptSubmitted` | —                  |
| **Error**                | —                    | —                  | —               | `errorOccurred`       | —                  |
| **Subagent Stop**        | `subagentStop`       | `SubagentStop`     | —               | —                     | `SubagentStop`     |
| **Pre-Compaction**       | `preCompact`         | `PreCompact`       | —               | —                     | `PreCompact`       |

---

### Comprehensive Example

```jsonc
{
  "version": 1, // v2.0.0 - Number. Schema version number. Default: 1.
  // - Defines the hooks schema version for parsing
  // - Future versions may add new features or change structure
  // Current value: 1
  "hooks": {
    // v1.0.0 - Object. Mapping event names to arrays of hook definitions.
    // - Keys are canonical camelCase event names (e.g., sessionStart, preToolUse)
    // - Values are arrays of hook definition objects
    // - See Supported Events by Tool table for per-tool availability
    "sessionStart": [
      {
        "type": "command", // v2.0.0 - String. Hook type. Default: "command".
        // - "command": Execute a shell command
        // - "prompt": Send a prompt to the AI for processing
        // Use case: Use "prompt" when you need AI reasoning;
        //           "command" for automated scripts
        "command": ".rulesync/hooks/start.sh", // v1.0.0 - String. Command to execute.
        // Required when type: "command".
        // - Shell command to run when hook triggers
        // - Cannot contain newlines (must be a single-line command)
        // - Supports environment variable expansion
        // Examples: "./scripts/start.sh", "echo 'Session started' >> log.txt"
        "timeout": 30, // v2.5.0 - Timeout in seconds (optional)
        // - Maximum time to wait for command execution
        // - Applies only to "command" type hooks
        // - Hook fails if command exceeds timeout
      },
    ],
    "preToolUse": [
      {
        "type": "command",
        "command": ".rulesync/hooks/pre-tool.sh",
        "matcher": "Write|Edit", // v2.0.0 - String. Pattern to filter hook execution.
        // - Regex-like pattern matching against tool names or event data
        // - Supports alternation with | (e.g., "Write|Edit")
        // - Cannot contain newlines
        // Common patterns:
        //   - "Write|Edit" - Match write operations
        //   - "Bash" - Match bash tool usage
        //   - "permission_prompt" - Match permission requests
        "timeout": 10,
      },
    ],
    "postToolUse": [
      {
        "type": "prompt", // v2.0.0 - Prompt-type hook
        "prompt": "Review output", // v2.0.0 - String. Prompt text.
        // Required when type: "prompt".
        // - Text sent to the AI when the hook triggers
        // - Can be multi-line (use \n for newlines in JSON)
        // Example: "Review the output above for security issues"
        "matcher": "Bash",
      },
    ],
    "stop": [
      {
        "type": "command",
        "command": ".rulesync/hooks/audit.sh",
        "loop_limit": 5, // v5.0.0 - Number. Maximum loop iterations. Default: null (unlimited).
        // - Prevents infinite loops in hook chains
        // - Applies when hooks may trigger other hooks
        // Use case: Set to 5 or 10 for safety in complex hook configurations
      },
    ],
  },
  // Tool-specific overrides (v2.0.0+)
  // Tool Override Keys: Tool-specific hook definitions.
  // - cursor.hooks: Cursor-specific hook definitions
  // - claudecode.hooks: Claude Code-specific hook definitions
  // - copilot.hooks: Copilot-specific hook definitions
  // - opencode.hooks: OpenCode-specific hook definitions
  // - factorydroid.hooks: Factory Droid-specific hook definitions
  // Behavior: Tool-specific definitions override canonical hooks for that tool only
  "cursor": {
    "hooks": {
      "afterFileEdit": [{ "command": ".cursor/hooks/format.sh" }],
      "afterShellExecution": [{ "command": ".cursor/hooks/post-shell.sh" }],
    },
  },
  "claudecode": {
    "hooks": {
      "notification": [
        {
          "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/notify.sh",
          "matcher": "permission_prompt",
        },
      ],
      "permissionRequest": [
        {
          "type": "prompt",
          "prompt": "Approve this action?",
        },
      ],
    },
  },
  "copilot": {
    "hooks": {
      "sessionStart": [{ "command": ".github/copilot/hooks/start.sh" }],
    },
  },
  "opencode": {
    "hooks": {
      "afterShellExecution": [{ "command": ".rulesync/hooks/post-shell.sh" }],
    },
  },
  "factorydroid": {
    "hooks": {
      "sessionStart": [{ "command": ".factorydroid/hooks/start.sh" }],
    },
  },
}
```

---

## `rulesync/commands/*.md`

Custom slash commands for AI tools.

### Comprehensive Example

```yaml
---
# Canonical options (v1.0.0+)
targets: # v1.0.0 - Array. Target AI tools. Default: ["*"].
  - "*" # - "*": Available in all supported tools
    # - Specific tools: ["claudecode", "copilot"]
    # Note: Some tools may have limited command support
description:
  "Review a PR" # v1.0.0 - **Required.** String. Command description.
  # - Brief explanation of what the command does
  # - Shown in command listings and help text
  # - Should be action-oriented (e.g., "Review a pull request",
  #   "Generate API documentation")
  # Example: "Review a PR for code quality issues"

# antigravity specific (v2.0.0+)
antigravity:
  trigger:
    "/review" # v2.0.0 - String. Command trigger path.
    # - Defines how users invoke the command in Antigravity
    # - Must start with / (slash command format)
    # Examples: /review, /docs, /test
  turbo:
    true # v2.0.0 - Boolean. Auto-execution mode. Default: true.
    # - true: Appends // turbo to enable auto-execution without confirmation
    # - false: Requires user confirmation before executing
    # Use case: Set to false for destructive or sensitive operations

# copilot specific (v3.0.0+)
copilot:
  description:
    "Review PR" # v3.0.0 - String. Copilot-specific description.
    # - Overrides canonical description for Copilot
    # - Used in Copilot's command UI
    # Example: "Review PR for security vulnerabilities"
---
# Command body content
target_pr = $ARGUMENTS

If target_pr is not provided, use the PR of the current branch.
```

---

## `rulesync/subagents/*.md`

Subagent definitions for specialized AI agents.

### Comprehensive Example

```yaml
---
# Canonical options (v1.0.0+)
targets: # v1.0.0 - Array. Target AI tools. Default: ["*"].
  - "*" # - "*": Available as subagent in all supported tools
    # - Specific tools: ["claudecode", "copilot", "opencode"]
    # Note: Subagent support varies by tool (see Supported Tools)
name:
  planner # v1.0.0 - **Required.** String. Subagent name.
  # - Must be unique within the project
  # - Kebab-case recommended (e.g., code-reviewer, test-writer)
  # - Used to invoke the subagent from other agents
  # Examples: planner, security-auditor, docs-generator
description:
  "General planner" # v1.0.0 - **Required.** String. Subagent description.
  # - Explains the subagent's role and capabilities
  # - Used in subagent listings and when selecting agents
  # - Should clearly state what tasks this agent specializes in
  # Example: "Security-focused code reviewer specializing in
  #           vulnerability detection"

# claudecode specific (v2.0.0+)
claudecode:
  model:
    "inherit" # v2.0.0 - String. Claude model selection. Default: "inherit".
    # - "inherit": Use the same model as the parent agent
    # - "opus": Claude 3 Opus (most capable, slowest)
    # - "sonnet": Claude 3.5 Sonnet (balanced capability/speed)
    # - "haiku": Claude 3 Haiku (fastest, good for simple tasks)
    # Use case: Use "opus" for complex reasoning, "haiku" for quick checks

# copilot specific (v3.0.0+)
copilot:
  tools: # v3.0.0 - Array. Additional tool permissions.
    - "web/fetch" # - List of extra tools this subagent can use
    - "github/repo" # - agent/runSubagent is automatically included
      # Examples:
      #   - ["web/fetch", "github/repo"] - Web and GitHub access
      #   - ["mcp/serena"] - MCP tool access

# opencode specific (v4.0.0+)
opencode:
  mode:
    "subagent" # v4.0.0 - String. OpenCode agent mode. Default: "subagent".
    # - "subagent": Standard subagent behavior
    # - Other modes may be added in future versions
    # - Defines how the agent is invoked and managed
  model:
    "anthropic/claude-sonnet-4-20250514" # v4.0.0 - String. Model identifier.
    # - Full model identifier string
    # Examples:
    #   - "anthropic/claude-sonnet-4-20250514"
    #   - "openai/gpt-4o"
    # See OpenCode documentation for supported models
  temperature:
    0.1 # v4.0.0 - Number. Sampling temperature. Range: 0.0 to 1.0.
    # - 0.0: Deterministic, focused responses
    # - 0.7: Balanced creativity and coherence
    # - 1.0: Maximum creativity, more varied responses
    # Use case: Lower for code review (consistency),
    #           higher for brainstorming
  tools: # v4.0.0 - Object. Tool permission overrides.
    write: false # Boolean values for each tool type:
    edit: false #   - write: File write permission
    bash: false #   - edit: File edit permission
    read: true #   - bash: Shell command execution
    glob: true #   - read: File read permission
    grep:
      true #   - glob: File globbing
      #   - grep: Pattern searching
      # Example: { "write": false, "edit": false, "bash": false, "read": true }
  permission: # v4.0.0 - Object. Command-specific permission overrides.
    bash: # - Maps command patterns to permission levels
      "git diff": allow # - Keys are command patterns (supports wildcards)
      "npm test":
        allow # - Values are "allow" or "deny"
        # Example:
        #   permission:
        #     bash:
        #       "git diff": allow
        #       "npm test": allow
        #       "rm -rf": deny
---
# Subagent body content
You are the planner for any tasks.
```

---

## `.rulesync/skills/*/SKILL.md`

Reusable skills with directory-based organization.

### Comprehensive Example

```yaml
---
# Canonical options (v1.0.0+)
name:
  example-skill # v1.0.0 - **Required.** String. Skill name.
  # - Must be unique across all skills
  # - Kebab-case recommended (e.g., git-workflow, error-handling)
  # - Used to reference and load the skill
  # Examples: git-workflow, test-writing, api-design
description:
  "Sample skill" # v1.0.0 - **Required.** String. Skill description.
  # - Explains what the skill does and when to use it
  # - Should be clear and actionable
  # - Displayed in skill listings and selection UIs
  # Example: "Step-by-step process for writing comprehensive unit tests"
targets: # v1.0.0 - Array. Target AI tools. Default: ["*"].
  - "*" # - "*": Skill available in all supported tools
    # - Specific tools: ["claudecode", "codexcli", "opencode"]
    # Note: Tool-specific allowed-tools configurations only apply
    #       to their respective tools

# claudecode specific (v2.0.0+)
claudecode:
  allowed-tools: # v2.0.0 - Array. Allowed tools for Claude Code.
    - "Bash" # - Restricts which tools the skill can use when loaded in
    - "Read" #   Claude Code
    - "Write" # - Uses Claude Code's tool names (PascalCase)
    - "Grep" # Common values:
    - "Glob" #   - "Bash" - Shell command execution
      #   - "Read" - File reading
      #   - "Write" - File writing
      #   - "Edit" - File editing
      #   - "Grep" - Pattern searching
      #   - "Glob" - File globbing
      # Example: ["Bash", "Read", "Grep", "Glob"]

# codexcli specific (v2.5.0+)
codexcli:
  short-description:
    "Quick skill" # v2.5.0 - String. Brief user-facing description.
    # - Shorter alternative to description for Codex CLI
    # - Used when space is limited in the UI
    # - Should be under 60 characters ideally
    # Example: "Git workflow helper"

# opencode specific (v3.0.0+)
opencode:
  allowed-tools: # v3.0.0 - Array. Allowed tools for OpenCode.
    - "bash" # - Restricts which tools the skill can use when loaded in
    - "read" #   OpenCode
    - "write" # - Uses OpenCode's tool names (lowercase)
      # Common values:
      #   - "bash" - Shell commands
      #   - "read" - File reading
      #   - "write" - File writing
      #   - "edit" - File editing
      #   - "glob" - File globbing
      #   - "grep" - Pattern searching
      # Example: ["bash", "read", "glob"]

# copilot specific (v3.0.0+)
copilot:
  license:
    "MIT" # v3.0.0 - String. License identifier for Copilot.
    # - Declares the license for this skill
    # - Used for attribution and compliance
    # Examples: "MIT", "Apache-2.0", "BSD-3-Clause"

# cline specific (v4.0.0+)
cline: {} # v4.0.0 - Object. Cline-specific options (currently empty).
  # - Placeholder for future Cline-specific configuration
  # - Currently accepts an empty object {}
  # Note: Future versions may add Cline-specific fields

# roo specific (v4.0.0+)
roo: {} # v4.0.0 - Object. Roo-specific options (currently empty).
  # - Placeholder for future Roo-specific configuration
  # - Currently accepts an empty object {}
  # Note: Future versions may add Roo-specific fields
---
# Skill body content
This skill can perform step-by-step operations.
```

---

## `.rulesync/mcp.json`

MCP (Model Context Protocol) server configuration.

### Comprehensive Example

```jsonc
{
  "mcpServers": {
    "serena": {
      "description": "Code analysis server", // v2.5.0 - String.
      // - Explains what the MCP server does
      // - Used in server listings and UI
      // Example: "Code analysis and symbol search server"
      "type": "stdio", // v2.0.0 - String. Connection type. Default: "stdio".
      // - "stdio": Standard input/output communication (spawn process)
      // - "sse": Server-Sent Events for streaming
      // - "http": HTTP/REST API communication
      // Use case: Use "stdio" for local CLI tools,
      //           "http" or "sse" for remote services
      "transport": "stdio", // v3.0.0 - String. Transport type (alias for type).
      // - Alternative field name for type
      // - Accepts same values: "stdio", "sse", "http"
      // - If both type and transport are specified,
      //   behavior is tool-dependent
      "command": "uvx", // v2.0.0 - String or Array. Command to run.
      // Required for stdio type.
      // - For string: Single command with arguments
      //   (e.g., "npx mcp-server")
      // - For array: Command and arguments as separate elements
      //   (e.g., ["node", "server.js"])
      // Examples:
      //   - "uvx --from git+url"
      //   - ["node", ".rulesync/mcp/server.js"]
      "args": ["--from", "git+url"], // v2.0.0 - Array. Command arguments.
      // - Arguments passed to the command
      // - Alternative to including args in command string
      // Example: ["--from", "git+url", "--port", "3000"]
      "env": {}, // v2.0.0 - Object. Environment variables.
      // - Key-value pairs of environment variables for the
      //   server process
      // - Values are strings
      // Example: { "API_KEY": "secret", "PORT": "3000" }
      "targets": ["*"], // v2.5.0 - Array. Target AI tools. Default: ["*"].
      // - "*": Available in all tools with MCP support
      // - Specific tools: ["claudecode", "copilot", "opencode"]
      // Note: See Supported Tools for MCP availability
      "exposed": true, // v4.0.0 - Boolean. Exposed to AI. Default: true.
      // - true: AI can discover and use this server's tools
      // - false: Server is hidden from AI (admin-only or manual use)
      // Use case: Set to false for servers requiring human oversight
      "disabled": false, // v3.0.0 - Boolean. Whether disabled. Default: false.
      // - true: Server configuration is ignored
      // - false: Server is active and available
      // Use case: Temporarily disable without removing configuration
      "cwd": "/path/to/server", // v3.0.0 - String. Working directory for the server.
      // - Absolute or relative path where server runs
      // - Relative paths resolved from project root
      // Example: "/path/to/server" or "./mcp-servers/analysis"
      "timeout": 30000, // v3.0.0 - Number. General timeout in milliseconds.
      // Default: 30000 (30s).
      // - Maximum time to wait for server responses
      // - Applies to most operations
      // Example: 60000 for 60-second timeout
      "networkTimeout": 10000, // v3.0.0 - Number. Network timeout in milliseconds.
      // Default: 10000 (10s).
      // - Specific timeout for network operations
      // - May be shorter than general timeout for faster
      //   failure detection
      // Example: 5000 for 5-second network timeout
      "trust": true, // v3.0.0 - Boolean. Whether to trust the server.
      // Default: false.
      // - true: Skip confirmation prompts for tool calls
      // - false: Ask user before executing server tools
      // Security note: Only trust servers from verified sources
      "alwaysAllow": ["search"], // v3.0.0 - Array. Tools to always allow.
      // - List of tool names that don't require confirmation
      // - Bypasses the trust check for specific tools
      // Example: ["search", "list", "read"]
      "tools": ["search", "list"], // v3.0.0 - Array. Available tools.
      // - Explicit list of tools this server provides
      // - Used for validation and discovery
      // Example: ["search", "list", "replace_symbol_body"]
      "headers": {
        // v3.0.0 - Object. HTTP headers for SSE/HTTP connections.
        "Authorization": "Bearer token", // - Key-value pairs of HTTP headers
      }, // - Used for authentication, content-type, etc.
      // Example: { "Authorization": "Bearer token",
      //            "Content-Type": "application/json" }
    },
    "http-server": {
      "type": "http",
      "url": "http://localhost:3000/mcp", // v2.0.0 - String. Server URL.
      // Required for sse/http type.
      // - Endpoint URL for HTTP or SSE connections
      // - Must be a valid URL with protocol
      // Examples:
      //   - "http://localhost:3000/mcp" (HTTP)
      //   - "http://localhost:3001/events" (SSE)
      "httpUrl": "http://localhost:3000", // v3.0.0 - String. Alternative HTTP URL.
      // - Secondary URL for HTTP-based servers
      // - Used for different endpoints or fallback
      // Example: "http://localhost:3000"
      "headers": {},
    },
    "sse-server": {
      "type": "sse",
      "url": "http://localhost:3001/events",
    },
    "kiro-server": {
      "type": "stdio",
      "command": ["node", "server.js"],
      "kiroAutoApprove": ["safe"], // v3.0.0 - Array. Kiro auto-approve tools.
      // - Tools that Kiro will automatically approve
      // - Kiro-specific configuration
      // Example: ["safe", "read", "search"]
      "kiroAutoBlock": ["dangerous"], // v3.0.0 - Array. Kiro auto-block tools.
      // - Tools that Kiro will automatically block/deny
      // - Kiro-specific configuration
      // Example: ["dangerous", "delete", "exec"]
    },
    "codex-server": {
      "type": "stdio",
      "command": "npx",
      "enabledTools": ["search"], // v4.0.0 - Array. Explicitly enabled tools.
      // - Whitelist of tools the AI can use (codexcli, opencode only)
      // - Only these tools will be available
      // Example: ["search", "find_symbol"]
      "disabledTools": ["delete"], // v4.0.0 - Array. Explicitly disabled tools.
      // - Blacklist of tools the AI cannot use (codexcli, opencode only)
      // - All other tools remain available
      // Example: ["delete", "write", "bash"]
    },
  },
}
```

---

## `.rulesync/.aiignore` or `.rulesyncignore`

Ignore patterns for AI processing (gitignore-style).

**Behavior:**

- **Preferred location**: `.rulesync/.aiignore` (recommended)
- **Legacy location**: `.rulesyncignore` (project root)
- **Precedence**: `.rulesync/.aiignore` wins if both exist
- **Default creation**: `.rulesync/.aiignore` (if neither exists)

**Rules:**

- Standard gitignore pattern syntax
- One pattern per line
- Blank lines and `#` comments ignored
- `!` negation supported
- `**` globstar supported

### Comprehensive Example

```ignore
# Dependencies (v1.0.0+)
node_modules/
vendor/
.pnpm-store/

# Build outputs
dist/
build/
out/
*.min.js
*.min.css

# Environment and secrets
.env
.env.*
.env.local
.env.production
credentials/
secrets/
*.key
*.pem

# Large/generated files
*.log
logs/
*.tmp
tmp/
temp/

# IDE and OS files
.vscode/
.idea/
.DS_Store
Thumbs.db

# Test artifacts
coverage/
.nyc_output/
*.lcov

# Documentation build
site/
docs/_build/

# Keep these even if parent is ignored
!important.md
!important.log
```

---

## `rulesync.jsonc`

Main configuration file for Rulesync CLI.

**Location:**

- Project root: `rulesync.jsonc`
- Local overrides: `rulesync.local.jsonc` (gitignored)

### Comprehensive Example

```jsonc
{
  "$schema": "https://rulesync.io/schemas/config.json", // v2.0.0 - JSON Schema URL

  // Target configuration (v1.0.0+)
  "targets": [
    // v1.0.0 - AI tools to generate configs for
    "agentsmd",
    "claudecode",
    "cursor",
    "copilot",
    "opencode",
  ],

  // Features (array format) - v1.0.0
  "features": ["rules", "commands", "subagents", "skills", "mcp", "hooks", "ignore"],

  // OR Features (per-target format) - v2.0.0
  // "features": {
  //   "copilot": ["commands"],
  //   "agentsmd": ["rules", "mcp", "commands", "subagents", "skills"],
  //   "cursor": ["rules", "subagents"],
  //   "claudecode": ["rules", "mcp", "subagents"],
  //   "opencode": ["rules", "mcp", "commands", "subagents"]
  // },

  // Base directories for generation - v1.0.0
  "baseDirs": ["."],

  // CLI behavior options - v1.0.0
  "verbose": false, // Enable verbose logging
  "delete": false, // Delete existing files before generating

  // Mode options - v2.0.0
  "global": false, // Use global (user-level) mode
  "silent": false, // Suppress non-error output

  // Simulation options - v2.5.0
  "simulateCommands": false, // Enable simulated slash commands
  "simulateSubagents": false, // Enable simulated subagents
  "simulateSkills": false, // Enable simulated skills

  // Validation options - v3.0.0
  "dryRun": false, // Preview changes without writing
  "check": false, // Check mode (validate without writing)

  // Declarative skill sources - v4.0.0
  "sources": [
    {
      "source": "owner/repo", // Repository reference (required)
      "skills": ["skill-one"], // Specific skills to fetch (optional)
    },
    {
      "source": "github:another-org/repo",
    },
    {
      "source": "https://github.com/user/repo/tree/main/.rulesync",
    },
  ],
}
```

### Tool-Specific Options Reference

**`$schema`** (v2.0.0): JSON Schema URL for editor support.

**`targets`** (v1.0.0): Array of AI tools to generate configs for. Default `["agentsmd"]`.

**`features`** (v1.0.0): Features to enable. Can be array (all targets) or per-target object. Default `["rules"]`.

**`baseDirs`** (v1.0.0): Array of base directories for generation. Default `["."]`.

**`verbose`** (v1.0.0): Boolean - Enable verbose logging.

**`delete`** (v1.0.0): Boolean - Delete existing files before generating.

**`global`** (v2.0.0): Boolean - Use global (user-level) mode.

**`silent`** (v2.0.0): Boolean - Suppress non-error output.

**`simulateCommands`** (v2.5.0): Boolean - Enable simulated slash commands.

**`simulateSubagents`** (v2.5.0): Boolean - Enable simulated subagents.

**`simulateSkills`** (v2.5.0): Boolean - Enable simulated skills.

**`dryRun`** (v3.0.0): Boolean - Preview changes without writing.

**`check`** (v3.0.0): Boolean - Check mode (validate without writing). Note: mutually exclusive with `dryRun`.

**`sources`** (v4.0.0): Array of declarative skill source entries.

**Source Entry Fields:**

**`source`** (v4.0.0): **Required.** Repository reference (owner/repo, github:owner/repo, or full URL).

**`skills`** (v4.0.0): Array - Specific skills to fetch (omit for all).

---

## Supported Tools

Rulesync supports **24 AI development tools**:

| Tool                 | Rules | Commands | Subagents | Skills | MCP | Hooks |
| -------------------- | ----- | -------- | --------- | ------ | --- | ----- |
| `agentsmd`           | ✓     | ✓        | ✓         | ✓      | -   | -     |
| `agentsskills`       | -     | -        | -         | ✓      | -   | -     |
| `antigravity`        | ✓     | ✓        | -         | ✓      | -   | -     |
| `augmentcode`        | ✓     | -        | -         | -      | -   | -     |
| `augmentcode-legacy` | ✓     | -        | -         | -      | -   | -     |
| `claudecode`         | ✓     | ✓        | ✓         | ✓      | ✓   | ✓     |
| `claudecode-legacy`  | ✓     | ✓        | ✓         | ✓      | ✓   | ✓     |
| `cline`              | ✓     | ✓        | -         | ✓      | ✓   | -     |
| `codexcli`           | ✓     | ✓        | ✓         | ✓      | ✓   | -     |
| `copilot`            | ✓     | ✓        | ✓         | ✓      | ✓   | ✓     |
| `cursor`             | ✓     | ✓        | -         | ✓      | ✓   | ✓     |
| `factorydroid`       | ✓     | ✓        | ✓         | ✓      | ✓   | ✓     |
| `geminicli`          | ✓     | ✓        | ✓         | ✓      | ✓   | -     |
| `goose`              | ✓     | -        | -         | -      | -   | -     |
| `junie`              | ✓     | -        | -         | -      | ✓   | -     |
| `kilo`               | ✓     | ✓        | -         | ✓      | ✓   | -     |
| `kiro`               | ✓     | ✓        | -         | ✓      | ✓   | -     |
| `opencode`           | ✓     | ✓        | ✓         | ✓      | ✓   | ✓     |
| `qwencode`           | ✓     | -        | -         | -      | -   | -     |
| `replit`             | ✓     | -        | -         | ✓      | -   | -     |
| `roo`                | ✓     | ✓        | ✓         | ✓      | ✓   | -     |
| `warp`               | ✓     | -        | -         | -      | -   | -     |
| `windsurf`           | ✓     | -        | -         | -      | -   | -     |
| `zed`                | ✓     | -        | -         | -      | -   | -     |

### Legacy Targets

These targets are **not** included in wildcard (`*`) expansion:

- `augmentcode-legacy` (v2.5.0)
- `claudecode-legacy` (v2.0.0)

### Conflicting Targets

These targets **cannot** be used together:

- `augmentcode` ↔ `augmentcode-legacy`
- `claudecode` ↔ `claudecode-legacy`

---

### v5.1.0 - v5.9.x

- Various bug fixes and improvements
- Dependency updates
- Documentation refinements

### v6.0.0

- Major refactoring of core architecture
- Performance improvements
- Enhanced error handling and validation

### v6.1.0 - v6.9.x

- Incremental improvements to existing features
- Bug fixes for edge cases in file generation
- Additional tool-specific optimizations
- Documentation improvements

### v7.0.0

- **Breaking Changes**: Restructured internal APIs
- Enhanced support for complex monorepo setups
- Improved conflict resolution in file generation

### v7.1.0 - v7.5.x

- v7.5.0: **Global support for Copilot and OpenCode rules** - Added global scope support for rules
- v7.5.0: **Subdirectory support for commands** - Added `supportsSubdirectory` flag for commands
- Various stability improvements

### v7.6.0

- **Goose rules support**: Added rules support for Goose AI coding tool (`.goosehints` / `.goose/memories`)
- **Codex CLI native subagent support**: Replaced simulated subagent with native TOML-based subagent support
- **Codex CLI local MCP mode**: Added local mode support for Codex CLI MCP configuration

### v7.7.0

- **Cline skill support**: Added Cline-specific skill configuration options
- **Goose ignore support**: Added `.gooseignore` support via `GooseIgnore` class
- Fixed: Preserve Cursor command frontmatter (description, handoffs)
- Fixed: Write Claude deny rules to `settings.json`

### v7.8.0

- **Enhanced error messages**: Include file path in frontmatter YAML parse error messages
- **New skill**: Added `git-worktree-runner` (gtr) skill
- Refactored: Renamed `clean-branches` command to `clean-git` with worktree prune support

### v7.9.0

- **Copilot hooks support**: GitHub Copilot now supports lifecycle hooks (previously hooks were only for Cursor, Claude Code, OpenCode)
- Major milestone: Full hooks support across all major AI tools

### v7.10.0 (Latest)

- **OpenCode JSONC support**: Added `opencode.jsonc` support with jsonc-parser (allows comments in JSON)
- Fixed: `--base-dir` flag was being silently ignored in generate command (now properly respected)
- Fixed: Import order and formatting in opencode-mcp.ts

### v7.10.1 (Planned)

- Planned patch release

---

## Complete Feature Matrix by Version

| Feature                    | Added In     | Tools Supported              |
| -------------------------- | ------------ | ---------------------------- |
| Basic rules                | v1.0.0       | agentsmd, claudecode, cursor |
| Hooks                      | v1.0.0       | claudecode, cursor           |
| Commands                   | v1.0.0       | agentsmd, cursor             |
| Subagents                  | v2.0.0       | claudecode, cursor           |
| Skills                     | v2.0.0       | agentsmd, claudecode         |
| MCP                        | v2.0.0       | claudecode, cursor           |
| Antigravity                | v2.0.0       | antigravity                  |
| OpenCode                   | v2.5.0       | opencode                     |
| Copilot                    | v3.0.0       | copilot                      |
| Gemini CLI                 | v3.0.0       | geminicli                    |
| Goose rules                | v3.0.0       | goose                        |
| Kiro                       | v3.0.0       | kiro                         |
| Junie, Qwen, Replit        | v4.0.0       | junie, qwencode, replit      |
| Factory Droid              | v5.0.0       | factorydroid                 |
| Copilot hooks              | v7.9.0       | copilot                      |
| Goose full support         | v7.6.0-7.7.0 | goose                        |
| Cline skills               | v7.7.0       | cline                        |
| Codex CLI native subagents | v7.6.0       | codexcli                     |
| OpenCode JSONC             | v7.10.0      | opencode                     |

---

## Version Reference

### v1.0.0

Initial release: `agentsmd`, `claudecode`, `cursor` support. Features: `rules`, `commands`, `hooks`.

### v2.0.0

Added: `localRoot`, `claudecode-legacy`, `augmentcode`, `factorydroid`, `kilo`, `roo`, `windsurf`, `subagents`, `skills`, `mcp`, `hooks.version`, `global`, `silent`, hook types, `antigravity`.

### v2.5.0

Added: `agentsmd.subprojectPath`, `codexcli.short-description`, MCP `description`/`targets`, `opencode`, simulation options.

### v3.0.0

Added: `claudecode.paths`, `copilot`, `copilot.excludeAgent`, `geminicli`, `goose`, `kiro`, `warp`, `zed`, MCP fields, `dryRun`, `check`, `loop_limit`.

### v4.0.0

Added: `opencode` subagent config, `cline`/`roo` skills, MCP `enabledTools`/`disabledTools`/`exposed`, `sources`, `junie`, `qwencode`, `replit`.

### v5.0.0

Added: `factorydroid` hooks support.

### v7.6.0

Added: Goose rules, Codex CLI native subagents.

### v7.7.0

Added: Cline skills, Goose ignore support.

### v7.8.0

Added: Enhanced error messages, git-worktree-runner skill.

### v7.9.0

Added: Copilot hooks support.

### v7.10.0

Added: OpenCode JSONC support.
