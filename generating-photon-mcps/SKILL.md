---
name: generating-photon-mcps
description: Generate single-file TypeScript MCPs for any MCP client. Use when users request creating an MCP, building an MCP server, or implementing Model Context Protocol tools. Generates AI-friendly, production-ready code with security best practices, auto-installing dependencies, and fork-first design. Triggers on phrases like "create a GitHub MCP", "build an MCP for file operations", or "make an MCP that does X".
---

# Photon MCP Generator

## Overview

Generate production-quality `.photon.ts` files—single-file TypeScript classes that become full MCP servers when run with the Photon runtime.

**Key Principles:**
- **One file = Complete MCP** - Single file contains entire working MCP server
- **AI-friendly code** - Optimized for AI understanding and modification
- **Auto-dependencies** - Dependencies auto-install via `@dependencies` tags (like `npx` or `uv`)
- **Fork-first design** - Easy to copy, customize, and modify
- **MCP-client agnostic** - Works with Claude Desktop, Cursor, Zed, Continue, Cline, and any MCP client

**Photon uses convention over configuration:**
- One class → One MCP server
- Public async methods → MCP tools
- Constructor parameters → Environment variables
- JSDoc comments → Tool descriptions
- TypeScript types → JSON schemas
- `@dependencies` tags → Auto-installed packages

This skill enables rapid MCP creation: from idea to working MCP in under 60 seconds.

---

## When to Use This Skill

Use this skill when the user requests:
- "Create an MCP for [functionality]"
- "Build an MCP that does [task]"
- "Make a GitHub Issues MCP"
- "Generate an MCP for file operations"
- Any request to build Model Context Protocol tools

---

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
- **Database** - SQLite, PostgreSQL, MongoDB
- **Shell/Git** - Command execution, git operations
- **Custom** - Combine patterns as needed

### Step 3: Generate the File

Create the `.photon.ts` file with:

1. **File header** with comprehensive JSDoc and `@dependencies` tags
2. **Import statements** for required modules
3. **Default export class** with descriptive name
4. **Constructor** with smart defaults and configuration
5. **Lifecycle hooks** (`onInitialize`, `onShutdown` if needed)
6. **Public async tool methods** with JSDoc and types
7. **Private helper methods** for shared logic

### Step 4: Apply AI-Friendly Code Principles

Generate code optimized for AI understanding and modification:

**Clear Structure:**
- ✅ Consistent file organization (header → imports → class → methods)
- ✅ One file = complete context for AI
- ✅ No scattered dependencies or hidden complexity
- ✅ Explicit types throughout (no `any` unless necessary)

**Readable Naming:**
- ✅ **Concise method names** (avoid repeating photon name)
- ✅ **CLI-friendly** (reads naturally: `photon cli time current`)
- ✅ Clear variable names (avoid abbreviations)
- ✅ Method names map directly to user actions
- ✅ Use standard CRUD verbs (get, list, create, update, delete)

**Self-Documenting:**
- ✅ Comprehensive JSDoc for AI to understand purpose
- ✅ Parameter descriptions become tool metadata
- ✅ Examples in JSDoc when helpful
- ✅ Type signatures that document expected inputs/outputs

**Modular and Composable:**
- ✅ One tool method = one focused operation
- ✅ Extract private helpers for reused logic
- ✅ Keep methods under 50 lines
- ✅ Clear separation of concerns

### Step 5: Apply Security Best Practices

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

### Step 6: Add Auto-Installing Dependencies

**IMPORTANT**: Dependencies auto-install via `@dependencies` JSDoc tags.

Add `@dependencies` tags in the file header JSDoc:

```typescript
/**
 * GitHub MCP - GitHub API integration
 *
 * Dependencies are auto-installed on first run (like npx or uv).
 *
 * @dependencies @octokit/rest@^20.0.0
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

### Step 7: Enable Fork-First Customization

Design generated code for easy forking and customization:

**Modularity:**
- Clear separation between core logic and customizable parts
- Configuration in constructor (easy to modify)
- Well-documented extension points

**Documentation for Forking:**
Include in file header:
```typescript
/**
 * Customization:
 * To add company-specific authentication:
 *   1. Copy this file: cp jira.photon.ts company-jira.photon.ts
 *   2. Modify the constructor to add custom auth
 *   3. Run: photon mcp company-jira
 */
```

**Common Fork Scenarios:**
- Adding custom authentication
- Extending with company-specific fields
- Combining multiple photons
- Adding workflow-specific logic

### Step 8: Provide Usage Instructions

After generating the file, provide:

1. **How to run it:**
   ```bash
   # Development mode (hot reload)
   photon mcp <name> --dev

   # Production mode
   photon mcp <name>
   ```

2. **How to get MCP client config:**
   ```bash
   photon get <name> --mcp
   ```
   Add output to MCP client configuration. Works with:
   - Claude Desktop (Anthropic)
   - Cursor (IDE)
   - Zed (IDE)
   - Continue (VS Code extension)
   - Cline (VS Code extension)
   - Any MCP-compatible client

3. **Environment variables needed** (if any):
   ```bash
   export <MCP_NAME>_<PARAM>=value
   ```

4. **Dependencies** (if any):
   ```
   Dependencies auto-install on first run: axios@^1.6.0, @octokit/rest@^20.0.0
   ```

5. **Fork and customize workflow**:
   ```bash
   # Copy to customize
   cp ~/.photon/<name>.photon.ts ~/.photon/my-<name>.photon.ts
   # Edit my-<name>.photon.ts
   # Run customized version
   photon mcp my-<name>
   ```

6. **Publishing to marketplace** (optional):
   ```bash
   # Create a marketplace from your photons
   photon sync marketplace --claude-code

   # This enables distribution via:
   # - Photon CLI: photon marketplace add org/repo
   # - Claude Code: /plugin marketplace add org/repo
   ```

   The `--claude-code` flag generates both:
   - `.marketplace/photons.json` - Photon CLI marketplace manifest
   - `.claude-plugin/marketplace.json` - Claude Code plugin manifest

   Push to GitHub and users can install your photons via either method.

---

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
  ) {}

  async onInitialize() {
    this.octokit = new Octokit({ auth: this.token, baseUrl: this.baseUrl });
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
import { join } from 'path';
import { homedir } from 'os';

export default class FilesystemMCP {
  constructor(
    private workdir: string = join(homedir(), 'Documents'),
    private maxFileSize: number = 10485760 // 10MB
  ) {}

  async read(params: { path: string }) { }
  async write(params: { path: string; content: string }) { }
  async list(params: { path?: string; recursive?: boolean }) { }
  async search(params: { pattern: string; path?: string }) { }

  // Security helper
  private _resolvePath(userPath: string): string {
    const resolved = join(this.workdir, userPath);
    if (!resolved.startsWith(this.workdir)) {
      throw new Error('Path traversal attempt detected');
    }
    return resolved;
  }
}
```

### Database MCP

For SQLite, PostgreSQL, MongoDB:
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
    console.error('[sqlite] ✅ Connected to database');
  }

  async onShutdown() {
    this.db?.close();
    console.error('[sqlite] ✅ Connection closed');
  }

  async query(params: { sql: string; params?: any[] }) { }
  async execute(params: { sql: string; params?: any[] }) { }
  async transaction(params: { statements: Array<{ sql: string; params?: any[] }> }) { }
}
```

---

## MCP Client Compatibility

Generated `.photon.ts` files are pure TypeScript classes that work with **any MCP client**.

### Running as MCP Server

```bash
# Development mode (hot reload)
photon mcp github-issues --dev

# Production mode
photon mcp github-issues

# Get config for your MCP client
photon get github-issues --mcp
```

### Compatible MCP Clients

- **Claude Desktop** (Anthropic)
- **Cursor** (IDE)
- **Zed** (IDE)
- **Continue** (VS Code extension)
- **Cline** (VS Code extension)
- Any MCP-compatible client implementing the Model Context Protocol

### Guidelines for Portable Code

Generate clean, standard TypeScript to ensure compatibility across MCP clients:

- ✅ Use standard TypeScript patterns
- ✅ No runtime-specific code or decorators
- ✅ Pure business logic, no protocol handling
- ✅ Let Photon runtime handle MCP protocol details

---

## AI-Friendly Code Principles

Generate code optimized for AI agents to understand, analyze, and modify:

### Complete Context in One File

**Why it matters:**
- AI can load entire MCP in single context window
- No jumping between files to understand logic
- Full picture available for analysis and suggestions

**How to achieve:**
```typescript
// Single file contains everything:
// - Configuration (constructor)
// - Dependencies (@dependencies tags)
// - Business logic (methods)
// - Security helpers (private methods)
// - Documentation (JSDoc)
```

### Self-Documenting Code

**Comprehensive JSDoc:**
```typescript
/**
 * Create a new issue in the repository
 *
 * This tool creates GitHub issues with optional labels and assignees.
 * Requires repository write access.
 *
 * @param title Issue title (required)
 * @param body Issue description in markdown (optional)
 * @param labels Array of label names to apply (optional)
 * @param assignees Array of GitHub usernames to assign (optional)
 * @returns Issue creation result with issue number and URL
 */
async createIssue(params: {
  title: string;
  body?: string;
  labels?: string[];
  assignees?: string[];
}) {
  // Implementation
}
```

**Type Signatures as Documentation:**
```typescript
// Good: Types tell the story
async query(params: { sql: string; params?: any[] }): Promise<{ success: boolean; rows?: any[]; error?: string }>

// Bad: Unclear what's returned
async query(params: any): Promise<any>
```

### Consistent Patterns

**Error Handling:**
```typescript
// Always return { success: boolean, ... }
return { success: true, data: result };
return { success: false, error: 'User-friendly message' };
```

**Naming Conventions:**

Photons are used both as MCP servers and CLI tools. Method names should be **concise and CLI-friendly**.

**Core Principle: Avoid Redundancy**
The photon/class name provides context, so don't repeat it in method names:

```typescript
// ❌ BAD - Redundant with class name
export default class Time {
  async getCurrentTime() { }
  async convertTime() { }
  async listTimezones() { }
}
// CLI: photon cli time getCurrentTime  (verbose and redundant)

// ✅ GOOD - Concise, context is clear
export default class Time {
  async current() { }
  async convert() { }
  async timezones() { }
}
// CLI: photon cli time current  (clean and natural)
```

**Standard CRUD Verbs:**
- Create: `create`, `add`, `new`
- Read: `get`, `list`, `find`, `search`
- Update: `update`, `set`, `modify`
- Delete: `delete`, `remove`, `clear`

**Singular vs Plural:**
- Singular for single items: `user()`, `get()`, `delete()`
- Plural for collections: `users()`, `list()`, `clear()`

**Examples:**
```typescript
// Resource-based photons
export default class Docker {
  async containers() { }     // List all
  async images() { }          // List all
  async start() { }           // Start container
  async stop() { }            // Stop container
  async remove() { }          // Remove container
}

// Service-based photons
export default class Slack {
  async post() { }            // Post message
  async channels() { }        // List channels
  async history() { }         // Channel history
  async search() { }          // Search messages
}

// Utility photons
export default class Math {
  async calculate() { }       // Calculate expression
  async random() { }          // Random number
}
```

**Private Helpers:**
```typescript
// Private helpers: underscore prefix
private _validateInput() { }
private _resolvePath() { }
```

**Test Your Names:**
Say the CLI command out loud - does it sound natural?
- ✅ `photon cli time current --timezone "America/New_York"`
- ❌ `photon cli time getCurrentTime --timezone "America/New_York"`

### Modular Structure

Break complex operations into well-named helpers:

```typescript
async processLargeFile(params: { path: string }) {
  const content = await this._readFile(params.path);
  const validated = this._validateContent(content);
  const transformed = this._transformData(validated);
  return this._formatOutput(transformed);
}

// AI can easily understand each step
private async _readFile(path: string) { }
private _validateContent(content: string) { }
private _transformData(data: any) { }
private _formatOutput(result: any) { }
```

---

## Quality Checklist

Before delivering a generated MCP, verify:

- [ ] File header JSDoc is comprehensive with `@dependencies` tags if needed
- [ ] All public methods have JSDoc with `@param` tags
- [ ] Constructor parameters have sensible defaults
- [ ] Security helpers are included (e.g., `_resolvePath` for filesystem)
- [ ] Error handling returns `{ success: boolean, ... }` format
- [ ] Dependencies declared with `@dependencies` tags (prefer built-in Node modules)
- [ ] Lifecycle hooks (`onInitialize`, `onShutdown`) are used if needed
- [ ] Private methods start with `_` or use `private` keyword
- [ ] All tool methods are async
- [ ] TypeScript types are explicit and complete
- [ ] Code is AI-friendly (clear structure, self-documenting)
- [ ] Fork-first design (easy to customize)
- [ ] MCP-client agnostic language in comments/docs

---

## Resources

### references/patterns.md

Comprehensive patterns for common MCP operations:
- File header format with `@dependencies` tags
- Filesystem operations with security
- HTTP/API operations with timeout handling
- Database operations with connection management
- Shell/Git operations with command injection prevention
- Security patterns (directory traversal, input validation)
- Error handling best practices
- Configuration patterns with smart defaults

**When to use:** Always consult this file for implementation patterns.

---

## Anti-Patterns to Avoid

❌ **Don't use base classes** - No `extends PhotonMCP` or similar
❌ **Don't add framework code** - Keep files pure TypeScript
❌ **Don't use decorators** - Convention over configuration
❌ **Don't forget security** - Always validate paths and inputs
❌ **Don't throw from tools** - Return error objects instead
❌ **Don't hardcode paths** - Use `homedir()` and smart defaults
❌ **Don't skip JSDoc** - Documentation becomes MCP metadata
❌ **Don't ignore lifecycle** - Use `onShutdown` for cleanup
❌ **Don't scatter logic** - Keep everything in one file
❌ **Don't use `any` types** - Be explicit for AI understanding
❌ **Don't mention "Claude"** - Use "MCP client" or "AI assistant" instead

---

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
 * Customization:
 * To add custom date formats:
 *   1. Copy this file: cp time.photon.ts my-time.photon.ts
 *   2. Add new format methods
 *   3. Run: photon mcp my-time
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
    console.error(`[time] ✅ Initialized with timezone: ${this.defaultTimezone}`);
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
      console.error('[time] Conversion failed:', error);
      return { success: false, error: error.message };
    }
  }

  /**
   * Calculate relative time from now
   * @param timestamp ISO 8601 timestamp
   */
  async relative(params: { timestamp: string }) {
    try {
      const date = new Date(params.timestamp);
      const now = new Date();
      const diffMs = now.getTime() - date.getTime();

      const seconds = Math.floor(diffMs / 1000);
      const minutes = Math.floor(seconds / 60);
      const hours = Math.floor(minutes / 60);
      const days = Math.floor(hours / 24);

      return {
        success: true,
        relative: this._formatRelative(days, hours, minutes, seconds),
        exactDiffMs: diffMs,
      };
    } catch (error: any) {
      console.error('[time] Relative calculation failed:', error);
      return { success: false, error: error.message };
    }
  }

  // Private helper for formatting relative time
  private _formatRelative(days: number, hours: number, minutes: number, seconds: number): string {
    if (days > 0) return `${days} day${days > 1 ? 's' : ''} ago`;
    if (hours > 0) return `${hours} hour${hours > 1 ? 's' : ''} ago`;
    if (minutes > 0) return `${minutes} minute${minutes > 1 ? 's' : ''} ago`;
    return `${seconds} second${seconds !== 1 ? 's' : ''} ago`;
  }
}
```

**Usage instructions provided:**
```bash
# Run in development mode (hot reload)
photon mcp time --dev

# Run in production mode
photon mcp time

# Get MCP client configuration
photon get time --mcp
# Add output to your MCP client's config file

# Configure timezone (optional)
export TIME_DEFAULT_TIMEZONE="America/Los_Angeles"

# Fork and customize
cp ~/.photon/time.photon.ts ~/.photon/my-time.photon.ts
# Edit my-time.photon.ts to add custom formats
photon mcp my-time
```

**Dependencies:** None (uses built-in Intl API)

---

## Summary

Generate production-quality Photon MCPs by:
1. Understanding user requirements
2. Choosing appropriate patterns from `references/patterns.md`
3. Generating complete `.photon.ts` file with:
   - AI-friendly, self-documenting code
   - Security best practices and error handling
   - Auto-installing dependencies via `@dependencies`
   - Fork-first design for easy customization
   - MCP-client agnostic implementation
4. Providing clear usage instructions for any MCP client
5. Verifying against quality checklist

Keep files framework-agnostic, secure, well-documented, AI-friendly, and focused on single responsibilities.
