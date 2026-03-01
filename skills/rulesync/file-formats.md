# File Formats

Complete reference for all Rulesync file formats.

---

## `rulesync/rules/*.md`

Rules define AI coding guidelines and constraints.

### Comprehensive Example

```yaml
---
# Canonical options (v1.0.0+)
root: true # v1.0.0 - Is this the root overview file
localRoot: false # v2.0.0 - Project-specific local rules
targets: # v1.0.0 - Target AI tools ("*" = all)
  - "*"
description: "Project overview" # v1.0.0 - Rule description
globs: # v1.0.0 - File patterns this rule applies to
  - "**/*"
  - "!node_modules/**"
  - "!.git/**"

# agentsmd specific (v2.5.0+)
agentsmd:
  subprojectPath: "packages/frontend" # v2.5.0 - Path to subproject for nested AGENTS.md

# claudecode specific (v3.0.0+)
claudecode:
  paths: # v3.0.0 - Glob patterns for conditional rules
    - "src/**/*.ts"
    - "tests/**/*.test.ts"

# cursor specific (v1.0.0+)
cursor:
  alwaysApply: true # v1.0.0 - Always apply this rule in Cursor
  description: "Cursor-specific" # v1.0.0 - Description shown in UI
  globs: # v1.0.0 - Cursor-specific file patterns
    - "*.ts"
    - "*.tsx"

# copilot specific (v4.0.0+)
copilot:
  excludeAgent: "code-review" # v4.0.0 - Exclude from "code-review" or "coding-agent"

# antigravity specific (v2.0.0+)
antigravity:
  trigger: "glob" # v2.0.0 - Trigger: "always_on", "glob", "manual", "model_decision"
  globs: # v2.0.0 - File patterns when trigger is "glob"
    - "src/**/*.rs"
    - "Cargo.toml"
  description: "Apply for Rust" # v2.0.0 - Description for "model_decision" trigger
---
# Rule body content goes here
```

### Tool-Specific Options Reference

**`root`** (v1.0.0): Whether this is the root/overview rule file. Only one root file should exist.

**`localRoot`** (v2.0.0): Project-specific local rules. For Claude Code: generates `CLAUDE.local.md`. For others: appends to root file.

**`targets`** (v1.0.0): Array of target AI tools. Use `"*"` for all tools or specify tool names.

**`description`** (v1.0.0): Human-readable description of the rule.

**`globs`** (v1.0.0): File patterns this rule applies to (e.g., `["**/*.ts", "!tests/**"]`).

**`agentsmd.subprojectPath`** (v2.5.0): Path to subproject for nested AGENTS.md support. Only valid when `root: false`.

**`claudecode.paths`** (v3.0.0): Array of glob patterns for conditional rules. Takes precedence over `globs`.

**`cursor.alwaysApply`** (v1.0.0): Boolean - Whether rule always applies in Cursor.

**`cursor.description`** (v1.0.0): Description shown in Cursor's rule UI.

**`cursor.globs`** (v1.0.0): Cursor-specific file patterns.

**`copilot.excludeAgent`** (v4.0.0): String - Exclude from `"code-review"` or `"coding-agent"`.

**`antigravity.trigger`** (v2.0.0): Trigger mode: `"always_on"`, `"glob"`, `"manual"`, `"model_decision"`.

**`antigravity.globs`** (v2.0.0): Required when trigger is `"glob"`.

**`antigravity.description`** (v2.0.0): Used with `"model_decision"` trigger.

---

## `.rulesync/hooks.json`

Lifecycle hooks run scripts at specific events.

Events use **canonical camelCase** (e.g., `sessionStart`). Tool mappings:

- Cursor: uses camelCase as-is
- Claude Code: converts to PascalCase (e.g., `SessionStart`)
- OpenCode: maps to dot notation (e.g., `session.created`)
- Copilot: maps canonical events to tool-specific names (e.g., `afterSubmitPrompt` → `userPromptSubmitted`, `afterError` → `errorOccurred`)
- Factory Droid: converts to PascalCase (e.g., `sessionStart` → `SessionStart`)

### Supported Events by Tool

| Event                  | Cursor | Claude Code | OpenCode | Copilot | Factory Droid |
| ---------------------- | ------ | ----------- | -------- | ------- | ------------- |
| `sessionStart`         | ✓      | ✓           | ✓        | ✓       | ✓             |
| `sessionEnd`           | ✓      | ✓           | -        | ✓       | ✓             |
| `preToolUse`           | ✓      | ✓           | ✓        | ✓       | ✓             |
| `postToolUse`          | ✓      | ✓           | ✓        | ✓       | ✓             |
| `stop`                 | ✓      | ✓           | ✓        | -       | ✓             |
| `beforeSubmitPrompt`   | ✓      | ✓           | -        | -       | ✓             |
| `subagentStop`         | ✓      | ✓           | -        | -       | ✓             |
| `preCompact`           | ✓      | ✓           | -        | -       | ✓             |
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

### Comprehensive Example

```jsonc
{
  "version": 1, // v2.0.0 - Schema version
  "hooks": {
    // v1.0.0 - Canonical hook definitions
    "sessionStart": [
      {
        "type": "command", // v2.0.0 - "command" or "prompt"
        "command": ".rulesync/hooks/start.sh", // v1.0.0 - Command to execute
        "timeout": 30, // v2.5.0 - Timeout in seconds
      },
    ],
    "preToolUse": [
      {
        "type": "command",
        "command": ".rulesync/hooks/pre-tool.sh",
        "matcher": "Write|Edit", // v2.0.0 - Pattern to match tool name
        "timeout": 10,
      },
    ],
    "postToolUse": [
      {
        "type": "prompt", // v2.0.0 - Prompt-type hook
        "prompt": "Review output", // v2.0.0 - Prompt text
        "matcher": "Bash",
      },
    ],
    "stop": [
      {
        "type": "command",
        "command": ".rulesync/hooks/audit.sh",
        "loop_limit": 5, // v5.0.0 - Maximum iterations
      },
    ],
  },
  // Tool-specific overrides (v2.0.0+)
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

### Tool-Specific Options Reference

**`version`** (v2.0.0): Schema version number.

**`hooks`** (v1.0.0): Object mapping event names to arrays of hook definitions.

**Hook Definition Fields:**

**`type`** (v2.0.0): `"command"` or `"prompt"`. Default is `"command"`.

**`command`** (v1.0.0): Command to execute. Required when `type: "command"`. Cannot contain newlines.

**`prompt`** (v2.0.0): Prompt text. Required when `type: "prompt"`.

**`matcher`** (v2.0.0): Pattern to match (e.g., tool name like `"Write"` or `"Write|Edit"`). Cannot contain newlines.

**`timeout`** (v2.5.0): Timeout in seconds.

**`loop_limit`** (v5.0.0): Maximum loop iterations (null for unlimited).

**Tool Override Keys** (v2.0.0+): `cursor.hooks`, `claudecode.hooks`, `copilot.hooks`, `opencode.hooks`, `factorydroid.hooks` - Tool-specific event overrides.

---

## `rulesync/commands/*.md`

Custom slash commands for AI tools.

### Comprehensive Example

```yaml
---
# Canonical options (v1.0.0+)
targets: # v1.0.0 - Target AI tools
  - "*"
description: "Review a PR" # v1.0.0 - Command description (required)

# antigravity specific (v2.0.0+)
antigravity:
  trigger: "/review" # v2.0.0 - Command trigger path
  turbo: true # v2.0.0 - Append // turbo for auto-execution

# copilot specific (v3.0.0+)
copilot:
  description: "Review PR" # v3.0.0 - Copilot-specific description
---
# Command body content
target_pr = $ARGUMENTS

If target_pr is not provided, use the PR of the current branch.
```

### Tool-Specific Options Reference

**`targets`** (v1.0.0): Array of target AI tools. Use `"*"` for all.

**`description`** (v1.0.0): **Required.** Command description.

**`antigravity.trigger`** (v2.0.0): Command trigger path (e.g., `/review`).

**`antigravity.turbo`** (v2.0.0): Boolean - Append `// turbo` for auto-execution. Default `true`.

**`copilot.description`** (v3.0.0): Copilot-specific description.

---

## `rulesync/subagents/*.md`

Subagent definitions for specialized AI agents.

### Comprehensive Example

```yaml
---
# Canonical options (v1.0.0+)
targets: # v1.0.0 - Target AI tools
  - "*"
name: planner # v1.0.0 - Subagent name (required)
description: "General planner" # v1.0.0 - Description (required)

# claudecode specific (v2.0.0+)
claudecode:
  model: "inherit" # v2.0.0 - "inherit", "opus", "sonnet", "haiku"

# copilot specific (v3.0.0+)
copilot:
  tools: # v3.0.0 - Additional tools
    - "web/fetch"
    - "github/repo"

# opencode specific (v4.0.0+)
opencode:
  mode: "subagent" # v4.0.0 - Agent mode
  model: "anthropic/claude-sonnet-4-20250514" # v4.0.0 - Model identifier
  temperature: 0.1 # v4.0.0 - Temperature (0-1)
  tools: # v4.0.0 - Tool permissions
    write: false
    edit: false
    bash: false
    read: true
    glob: true
    grep: true
  permission: # v4.0.0 - Permission overrides
    bash:
      "git diff": allow
      "npm test": allow
---
# Subagent body content
You are the planner for any tasks.
```

### Tool-Specific Options Reference

**`targets`** (v1.0.0): Array of target AI tools. Use `"*"` for all.

**`name`** (v1.0.0): **Required.** Subagent name (kebab-case recommended).

**`description`** (v1.0.0): **Required.** Subagent description.

**`claudecode.model`** (v2.0.0): Model selection: `"inherit"`, `"opus"`, `"sonnet"`, `"haiku"`.

**`copilot.tools`** (v3.0.0): Array of additional tools. `agent/runSubagent` is auto-included.

**`opencode.mode`** (v4.0.0): OpenCode agent mode (default `"subagent"`).

**`opencode.model`** (v4.0.0): Model identifier (e.g., `"anthropic/claude-sonnet-4-20250514"`).

**`opencode.temperature`** (v4.0.0): Temperature value (0-1).

**`opencode.tools`** (v4.0.0): Object with tool permissions (`write`, `edit`, `bash`, `read`, `glob`, `grep`).

**`opencode.permission`** (v4.0.0): Object with permission overrides for commands.

---

## `.rulesync/skills/*/SKILL.md`

Reusable skills with directory-based organization.

### Comprehensive Example

```yaml
---
# Canonical options (v1.0.0+)
name: example-skill # v1.0.0 - Skill name (required)
description: "Sample skill" # v1.0.0 - Description (required)
targets: # v1.0.0 - Target AI tools
  - "*"

# claudecode specific (v2.0.0+)
claudecode:
  allowed-tools: # v2.0.0 - Allowed tools for Claude Code
    - "Bash"
    - "Read"
    - "Write"
    - "Grep"
    - "Glob"

# codexcli specific (v2.5.0+)
codexcli:
  short-description: "Quick skill" # v2.5.0 - Brief user-facing description

# opencode specific (v3.0.0+)
opencode:
  allowed-tools: # v3.0.0 - Allowed tools for OpenCode
    - "bash"
    - "read"
    - "write"

# copilot specific (v3.0.0+)
copilot:
  license: "MIT" # v3.0.0 - License for Copilot

# cline specific (v4.0.0+)
cline: {}

# roo specific (v4.0.0+)
roo: {}
---
# Skill body content
This skill can perform step-by-step operations.
```

### Tool-Specific Options Reference

**`name`** (v1.0.0): **Required.** Skill name (kebab-case recommended).

**`description`** (v1.0.0): **Required.** Skill description.

**`targets`** (v1.0.0): Array of target AI tools. Use `"*"` for all.

**`claudecode.allowed-tools`** (v2.0.0): Array of allowed tools for Claude Code.

**`codexcli.short-description`** (v2.5.0): Brief user-facing description.

**`opencode.allowed-tools`** (v3.0.0): Array of allowed tools for OpenCode.

**`copilot.license`** (v3.0.0): License string for Copilot.

**`cline`** (v4.0.0): Cline-specific options (empty object for now).

**`roo`** (v4.0.0): Roo-specific options (empty object for now).

---

## `.rulesync/mcp.json`

MCP (Model Context Protocol) server configuration.

### Comprehensive Example

```jsonc
{
  "mcpServers": {
    "serena": {
      "description": "Code analysis server", // v2.5.0 - Human-readable description
      "type": "stdio", // v2.0.0 - "stdio", "sse", "http"
      "transport": "stdio", // v3.0.0 - Transport alias for type
      "command": "uvx", // v2.0.0 - Command to run
      "args": ["--from", "git+url"], // v2.0.0 - Command arguments
      "env": {}, // v2.0.0 - Environment variables
      "targets": ["*"], // v2.5.0 - Target AI tools
      "exposed": true, // v4.0.0 - Exposed to AI
      "disabled": false, // v3.0.0 - Whether disabled
      "cwd": "/path/to/server", // v3.0.0 - Working directory
      "timeout": 30000, // v3.0.0 - General timeout (ms)
      "networkTimeout": 10000, // v3.0.0 - Network timeout (ms)
      "trust": true, // v3.0.0 - Trust the server
      "alwaysAllow": ["search"], // v3.0.0 - Tools to always allow
      "tools": ["search", "list"], // v3.0.0 - Available tools
      "headers": {
        // v3.0.0 - HTTP headers for sse/http
        "Authorization": "Bearer token",
      },
    },
    "http-server": {
      "type": "http",
      "url": "http://localhost:3000/mcp", // v2.0.0 - Server URL
      "httpUrl": "http://localhost:3000", // v3.0.0 - Alternative HTTP URL
      "headers": {},
    },
    "sse-server": {
      "type": "sse",
      "url": "http://localhost:3001/events",
    },
    "kiro-server": {
      "type": "stdio",
      "command": ["node", "server.js"],
      "kiroAutoApprove": ["safe"], // v3.0.0 - Kiro auto-approve
      "kiroAutoBlock": ["dangerous"], // v3.0.0 - Kiro auto-block
    },
    "codex-server": {
      "type": "stdio",
      "command": "npx",
      "enabledTools": ["search"], // v4.0.0 - Explicitly enabled (codexcli, opencode)
      "disabledTools": ["delete"], // v4.0.0 - Explicitly disabled (codexcli, opencode)
    },
  },
}
```

### Tool-Specific Options Reference

**`type`** (v2.0.0): Connection type: `"stdio"`, `"sse"`, `"http"`. Default `"stdio"`.

**`transport`** (v3.0.0): Transport type (alias for `type`).

**`command`** (v2.0.0): Command to run (string or array). Required for `stdio` type.

**`args`** (v2.0.0): Array of command arguments.

**`url`** (v2.0.0): Server URL. Required for `sse`/`http` type.

**`httpUrl`** (v3.0.0): Alternative HTTP URL.

**`env`** (v2.0.0): Object of environment variables.

**`cwd`** (v3.0.0): Working directory for the server.

**`disabled`** (v3.0.0): Boolean - Whether server is disabled.

**`description`** (v2.5.0): Human-readable description.

**`targets`** (v2.5.0): Array of target AI tools. Default `["*"]`.

**`exposed`** (v4.0.0): Boolean - Whether server is exposed to AI. Default `true`.

**`timeout`** (v3.0.0): General timeout in milliseconds.

**`networkTimeout`** (v3.0.0): Network timeout in milliseconds.

**`trust`** (v3.0.0): Boolean - Whether to trust the server.

**`alwaysAllow`** (v3.0.0): Array of tools to always allow.

**`tools`** (v3.0.0): Array of available tools.

**`kiroAutoApprove`** (v3.0.0): Array of Kiro auto-approve tools.

**`kiroAutoBlock`** (v3.0.0): Array of Kiro auto-block tools.

**`headers`** (v3.0.0): Object of HTTP headers for sse/http.

**`enabledTools`** (v4.0.0): Array - Explicitly enabled tools (codexcli, opencode only).

**`disabledTools`** (v4.0.0): Array - Explicitly disabled tools (codexcli, opencode only).

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
