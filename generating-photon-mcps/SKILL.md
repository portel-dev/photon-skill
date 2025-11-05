---
name: generating-photon-mcps
description: Generate single-file TypeScript MCPs for the Photon runtime. Use when users request creating an MCP, building an MCP server, or implementing Model Context Protocol tools. Triggers on phrases like "create a GitHub MCP", "build an MCP for file operations", or "make an MCP that does X".
---

# Photon MCP Generator

## Overview

Generate production-quality `.photon.ts` files—single-file TypeScript classes that become full MCP servers when run with the Photon runtime.

Photon uses **convention over configuration**:
- One class → One MCP server
- Public async methods → MCP tools
- Constructor parameters → Environment variables
- JSDoc comments → Tool descriptions
- TypeScript types → JSON schemas

This skill enables rapid MCP creation: from idea to working MCP in under 60 seconds.

## When to Use This Skill

Use this skill when the user requests:
- "Create an MCP for [functionality]"
- "Build an MCP that does [task]"
- "Make a GitHub Issues MCP"
- "Generate an MCP for file operations"
- Any request to build Model Context Protocol tools

## Generation Workflow

### Step 1: Understand Requirements

Ask clarifying questions if needed:
- What functionality should the MCP provide?
- What tools/operations are needed?
- Are there any specific APIs or services to integrate?
- What configuration is required (API keys, paths, etc.)?

### Step 2: Choose Pattern

Select the appropriate pattern from `references/patterns.md`:
- **Filesystem** - File/directory operations
- **HTTP/API** - Web requests, API integration
- **Database** - SQLite or other databases
- **Shell/Git** - Command execution, git operations
- **Custom** - Combine patterns as needed

### Step 3: Generate the File

Create the `.photon.ts` file with:

1. **File header** with comprehensive JSDoc
2. **Import statements** for required modules
3. **Default export class** with descriptive name
4. **Constructor** with smart defaults and configuration
5. **Lifecycle hooks** (`onInitialize`, `onShutdown` if needed)
6. **Public async tool methods** with JSDoc and types
7. **Private helper methods** for shared logic

### Step 4: Apply Best Practices

Ensure the generated file follows these principles:

**Security:**
- ✅ Validate all user inputs
- ✅ Prevent directory traversal (use `_resolvePath` helper)
- ✅ Escape shell commands or use spawn with args array
- ✅ Enforce file size limits
- ✅ Use parameterized database queries

**Error Handling:**
- ✅ Return `{ success: boolean, ... }` format
- ✅ Provide helpful error messages
- ✅ Never throw errors from tool methods
- ✅ Log errors to stderr, not stdout

**Configuration:**
- ✅ Use smart platform-aware defaults
- ✅ Constructor parameters map to env vars
- ✅ Validate required configuration
- ✅ Log configuration in `onInitialize()`

**Documentation:**
- ✅ Comprehensive file header JSDoc
- ✅ JSDoc for every public method
- ✅ Parameter descriptions with `@param`
- ✅ Return value documentation

**Code Quality:**
- ✅ One tool method = one focused operation
- ✅ Keep methods under 50 lines
- ✅ Use descriptive variable names
- ✅ Extract helpers for repeated logic

### Step 5: Add Dependencies (If Needed)

If the MCP requires external npm packages, add `@dependencies` JSDoc tags:

```typescript
/**
 * GitHub MCP - GitHub API integration
 *
 * Dependencies are auto-installed on first run.
 *
 * @dependencies @octokit/rest@^20.0.0
 * @dependencies axios@^1.6.0, date-fns@^3.0.0
 */
```

**How auto-install works:**
- Photon parses `@dependencies` tags from JSDoc
- Auto-installs to `~/.cache/photon-mcp/dependencies/{mcp-name}/`
- Works like `npx` or Python's `uv` - zero manual setup
- Cached per MCP, isolated from other MCPs
- Only installs once, reuses on subsequent runs

**Formats:**
- Single: `@dependencies axios@^1.0.0`
- Multiple (one line): `@dependencies axios@^1.0.0, date-fns@^2.0.0`
- Multiple (separate tags): `@dependencies axios@^1.0.0` + `@dependencies date-fns@^2.0.0`
- Scoped packages: `@dependencies @octokit/rest@^3.1.0`

**Prefer built-in modules when possible:**
- Use `fs/promises` instead of external file libraries
- Use `node:fetch` instead of axios (for simple requests)
- Use `child_process` instead of external shell libraries
- Only add external dependencies when they provide significant value

### Step 6: Provide Usage Instructions

After generating the file, provide:

1. **How to run it:**
   ```bash
   photon <mcp-name> --dev
   ```

2. **How to configure it:**
   ```bash
   photon <mcp-name> --config
   ```

3. **Environment variables needed** (if any):
   ```bash
   export MCP_NAME_PARAM_NAME="value"
   ```

4. **Dependencies** (if any):
   ```
   Dependencies auto-install on first run: axios@^1.6.0, @octokit/rest@^20.0.0
   ```

## Common MCP Types

### API Integration MCP

For GitHub, Jira, Slack, etc.:
- Use HTTP pattern from `references/patterns.md`
- Constructor takes API key, base URL
- Methods for common operations (list, create, update, delete)
- Include rate limiting if needed

**Example structure:**
```typescript
/**
 * @dependencies @octokit/rest@^20.0.0
 */
import { Octokit } from '@octokit/rest';

export default class GitHubMCP {
  private octokit: Octokit;

  constructor(
    private token: string,
    private baseUrl: string = 'https://api.github.com'
  ) {
    this.octokit = new Octokit({ auth: token, baseUrl });
  }

  async listIssues(params: { repo: string; state?: string }) { }
  async createIssue(params: { repo: string; title: string; body: string }) { }
  async updateIssue(params: { repo: string; number: number; ... }) { }
}
```

### Data Processing MCP

For text manipulation, calculations, transformations:
- Minimal or no constructor parameters
- Pure TypeScript (no external dependencies)
- Fast, synchronous-friendly operations

**Example structure:**
```typescript
export default class TextMCP {
  async slugify(params: { text: string }) { }
  async wordCount(params: { text: string }) { }
  async titleCase(params: { text: string }) { }
}
```

### File Operations MCP

For reading, writing, searching files:
- Use filesystem pattern from `references/patterns.md`
- Constructor takes working directory, size limits
- Include security helpers (`_resolvePath`)
- Support both single files and directories

**Example structure:**
```typescript
export default class FilesystemMCP {
  constructor(
    private workdir: string = join(homedir(), 'Documents'),
    private maxFileSize: number = 10485760
  ) {}

  async read(params: { path: string }) { }
  async write(params: { path: string; content: string }) { }
  async list(params: { path?: string; recursive?: boolean }) { }
  async search(params: { pattern: string; path?: string }) { }
}
```

### Database MCP

For SQLite or other databases:
- Use database pattern from `references/patterns.md`
- Initialize connection in `onInitialize()`
- Close connection in `onShutdown()`
- Support transactions for atomic operations

**Example structure:**
```typescript
/**
 * @dependencies better-sqlite3@^11.0.0
 */
import Database from 'better-sqlite3';
import { join } from 'path';
import { homedir } from 'os';

export default class SqliteMCP {
  private db?: Database.Database;

  constructor(private dbPath: string = join(homedir(), 'data.db')) {}

  async onInitialize() {
    this.db = new Database(this.dbPath);
  }

  async onShutdown() {
    this.db?.close();
  }

  async query(params: { sql: string; params?: any[] }) { }
  async execute(params: { sql: string; params?: any[] }) { }
}
```

## Framework-Agnostic Design

Photon files are pure TypeScript classes with **no framework dependencies**. This enables deployment flexibility:

- **Photon runtime** → MCP protocol (Claude Desktop, Cursor, Windsurf)
- **Restler-TS runtime** → REST/GraphQL/WebSocket APIs
- **Direct import** → Use as a library

See `references/restler-integration.md` for details on multi-protocol deployment.

**Guidelines:**
- No Photon-specific code in `.photon.ts` files
- Use standard TypeScript patterns
- Keep classes reusable across runtimes

## Quality Checklist

Before delivering a generated MCP, verify:

- [ ] File header JSDoc is comprehensive with @dependencies tags if needed
- [ ] All public methods have JSDoc with `@param` tags
- [ ] Constructor parameters have sensible defaults
- [ ] Security helpers are included (e.g., `_resolvePath` for filesystem)
- [ ] Error handling returns `{ success: boolean, ... }` format
- [ ] Dependencies declared with @dependencies tags (prefer built-in Node modules)
- [ ] Lifecycle hooks (`onInitialize`, `onShutdown`) are used if needed
- [ ] Private methods start with `_` or use `private` keyword
- [ ] All tool methods are async
- [ ] TypeScript types are explicit and complete

## Resources

### references/patterns.md

Comprehensive patterns for common MCP operations:
- File header format
- Filesystem operations with security
- HTTP/API operations with timeout handling
- Database operations with connection management
- Shell/Git operations with command injection prevention
- Security patterns (directory traversal, input validation)
- Error handling best practices
- Configuration patterns with smart defaults

**When to use:** Always consult this file for implementation patterns.

### references/restler-integration.md

Guide for multi-protocol deployment with Restler-TS framework:
- How to use Photon files as REST APIs
- GraphQL integration
- WebSocket support
- Direct library usage
- Benefits of framework-agnostic design

**When to use:** When user asks about using MCP in other ways or exposing as an API.

## Anti-Patterns to Avoid

❌ **Don't use base classes** - No `extends PhotonMCP` or similar
❌ **Don't add framework code** - Keep files pure TypeScript
❌ **Don't use decorators** - Convention over configuration
❌ **Don't forget security** - Always validate paths and inputs
❌ **Don't throw from tools** - Return error objects instead
❌ **Don't hardcode paths** - Use `homedir()` and smart defaults
❌ **Don't skip JSDoc** - Documentation becomes MCP metadata
❌ **Don't ignore lifecycle** - Use `onShutdown` for cleanup

## Example Generation

**User request:** "Create an MCP for time and date operations"

**Generated file:** `time.photon.ts`

```typescript
/**
 * Time - Date and time utilities
 *
 * Provides timezone-aware date/time operations using built-in Intl API.
 * No external dependencies required.
 *
 * Common use cases:
 * - Convert timezones: "What time is it in Tokyo?"
 * - Format dates: "Format this date as MM/DD/YYYY"
 * - Calculate relative time: "How long ago was this?"
 *
 * Configuration:
 * - defaultTimezone: Default timezone (default: system timezone)
 *
 * Dependencies: None (uses built-in Intl API)
 *
 * @version 1.0.0
 * @author Portel
 * @license MIT
 */

export default class Time {
  constructor(
    private defaultTimezone: string = Intl.DateTimeFormat().resolvedOptions().timeZone
  ) {}

  async onInitialize() {
    console.error(`[time] ✅ Initialized`);
    console.error(`[time] Default timezone: ${this.defaultTimezone}`);
  }

  /**
   * Get current time in specified timezone
   * @param timezone IANA timezone name (e.g., "America/New_York")
   */
  async now(params: { timezone?: string }) {
    const tz = params.timezone || this.defaultTimezone;
    const date = new Date();

    return {
      success: true,
      timestamp: date.toISOString(),
      formatted: date.toLocaleString('en-US', { timeZone: tz }),
      timezone: tz,
    };
  }

  /**
   * Convert timestamp to different timezone
   * @param timestamp ISO 8601 timestamp
   * @param timezone Target timezone
   */
  async convert(params: { timestamp: string; timezone: string }) {
    try {
      const date = new Date(params.timestamp);

      if (isNaN(date.getTime())) {
        return { success: false, error: 'Invalid timestamp format' };
      }

      return {
        success: true,
        formatted: date.toLocaleString('en-US', { timeZone: params.timezone }),
        timezone: params.timezone,
      };
    } catch (error: any) {
      return { success: false, error: error.message };
    }
  }

  /**
   * Format timestamp with custom format
   * @param timestamp ISO 8601 timestamp
   * @param format Format string (e.g., "YYYY-MM-DD")
   */
  async format(params: { timestamp: string; format: string }) {
    // Implementation here
  }
}
```

**Usage instructions provided:**
```bash
# Run in dev mode
photon time --dev

# Generate config for Claude Desktop
photon time --config

# Configure timezone (optional)
export TIME_DEFAULT_TIMEZONE="America/Los_Angeles"
```

## Summary

Generate production-quality Photon MCPs by:
1. Understanding user requirements
2. Choosing appropriate patterns from `references/patterns.md`
3. Generating complete `.photon.ts` file with security and error handling
4. Providing clear usage instructions
5. Verifying against quality checklist

Keep files framework-agnostic, secure, well-documented, and focused on single responsibilities.
