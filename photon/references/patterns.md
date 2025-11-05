# Photon MCP Patterns Reference

Comprehensive patterns for common MCP operations. Use these as templates when generating `.photon.ts` files.

## Table of Contents

1. [File Header Format](#file-header-format)
2. [Filesystem Operations](#filesystem-operations)
3. [HTTP/API Operations](#http-api-operations)
4. [Database Operations](#database-operations)
5. [Shell/Git Operations](#shell-git-operations)
6. [Security Patterns](#security-patterns)
7. [Error Handling](#error-handling)
8. [Configuration Patterns](#configuration-patterns)

---

## File Header Format

Every `.photon.ts` file should start with a comprehensive JSDoc header:

```typescript
/**
 * MCP-Name - Brief description (one line)
 *
 * Detailed description: what the MCP provides and what problems it solves.
 * Multiple lines are fine.
 *
 * Common use cases:
 * - Use case 1: "Example user request"
 * - Use case 2: "Another example"
 * - Use case 3: "Third example"
 *
 * Example: toolName({ param: "value" }) → expected result
 *
 * Configuration:
 * - paramName: Description (default: value)
 * - paramName2: Description (default: value)
 *
 * Dependencies: package-name (or None if no external deps)
 *
 * @version 1.0.0
 * @author Portel
 * @license MIT
 */
```

---

## Filesystem Operations

### Pattern: Filesystem MCP

```typescript
import { readFile, writeFile, readdir, mkdir, rm, stat } from 'fs/promises';
import { join, resolve, relative } from 'path';
import { existsSync } from 'fs';
import { homedir } from 'os';

export default class Filesystem {
  constructor(
    private workdir: string = join(homedir(), 'Documents'),
    private maxFileSize: number = 10485760, // 10MB
    private allowHidden: boolean = false
  ) {
    if (!workdir || workdir.trim() === '') {
      throw new Error('Working directory must be a non-empty path');
    }
    if (!existsSync(workdir)) {
      throw new Error(`Working directory does not exist: ${workdir}`);
    }
  }

  async onInitialize() {
    console.error(`[filesystem] ✅ Initialized`);
    console.error(`[filesystem] Working directory: ${this.workdir}`);
  }

  /**
   * Read file contents
   * @param path Path to file (relative to working directory)
   * @param encoding File encoding (default: utf-8)
   */
  async read(params: { path: string; encoding?: 'utf-8' | 'base64' }) {
    try {
      const fullPath = this._resolvePath(params.path);
      const encoding = params.encoding || 'utf-8';
      const content = await readFile(fullPath, encoding as BufferEncoding);
      return { success: true, content, path: fullPath };
    } catch (error: any) {
      return { success: false, error: error.message };
    }
  }

  /**
   * Write content to file
   * @param path Path to file (relative to working directory)
   * @param content Content to write
   */
  async write(params: { path: string; content: string }) {
    try {
      const fullPath = this._resolvePath(params.path);
      await writeFile(fullPath, params.content, 'utf-8');
      return { success: true, message: `File written: ${fullPath}` };
    } catch (error: any) {
      return { success: false, error: error.message };
    }
  }

  /**
   * List files and directories
   * @param path Directory path (default: working directory root)
   * @param recursive List recursively (default: false)
   */
  async list(params: { path?: string; recursive?: boolean }) {
    try {
      const dirPath = this._resolvePath(params.path || '.');
      const entries = await this._listRecursive(dirPath, params.recursive || false);
      return { success: true, path: dirPath, entries };
    } catch (error: any) {
      return { success: false, error: error.message };
    }
  }

  /**
   * Delete file or directory
   * @param path Path to delete (relative to working directory)
   * @param recursive Delete directory recursively (default: false)
   */
  async delete(params: { path: string; recursive?: boolean }) {
    try {
      const fullPath = this._resolvePath(params.path);
      await rm(fullPath, { recursive: params.recursive || false, force: true });
      return { success: true, message: `Deleted: ${fullPath}` };
    } catch (error: any) {
      return { success: false, error: error.message };
    }
  }

  // Private helper: Prevent directory traversal attacks
  private _resolvePath(path: string): string {
    const resolved = resolve(this.workdir, path);
    const rel = relative(this.workdir, resolved);

    if (rel.startsWith('..') || resolve(rel) === rel) {
      throw new Error(`Access denied: path outside working directory`);
    }

    return resolved;
  }

  private async _listRecursive(dir: string, recursive: boolean): Promise<string[]> {
    const entries = await readdir(dir, { withFileTypes: true });
    const files: string[] = [];

    for (const entry of entries) {
      if (!this.allowHidden && entry.name.startsWith('.')) {
        continue;
      }

      const fullPath = join(dir, entry.name);
      const relativePath = relative(this.workdir, fullPath);

      if (entry.isDirectory()) {
        if (recursive) {
          const subFiles = await this._listRecursive(fullPath, true);
          files.push(...subFiles);
        } else {
          files.push(relativePath + '/');
        }
      } else {
        files.push(relativePath);
      }
    }

    return files;
  }
}
```

**Key patterns:**
- Use `join(homedir(), 'Documents')` for default working directory
- Always use `_resolvePath()` helper to prevent directory traversal
- Return `{ success: boolean, ... }` format consistently
- Use `onInitialize()` to log configuration

---

## HTTP/API Operations

### Pattern: HTTP Client MCP

```typescript
export default class Fetch {
  constructor(
    private timeout: number = 5000,
    private maxRedirects: number = 5,
    private userAgent?: string
  ) {}

  async onInitialize() {
    console.error(`[fetch] ✅ Initialized`);
    console.error(`[fetch] Timeout: ${this.timeout}ms`);
  }

  /**
   * Send GET request
   * @param url URL to fetch
   * @param headers Optional HTTP headers
   */
  async get(params: { url: string; headers?: Record<string, string> }) {
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), this.timeout);

    try {
      const headers = {
        'User-Agent': this.userAgent || 'Photon-MCP/1.0',
        ...params.headers,
      };

      const response = await fetch(params.url, {
        method: 'GET',
        headers,
        signal: controller.signal,
      });

      const data = await response.text();

      return {
        success: true,
        status: response.status,
        statusText: response.statusText,
        data,
        headers: Object.fromEntries(response.headers),
      };
    } catch (error: any) {
      if (error.name === 'AbortError') {
        return { success: false, error: `Request timeout after ${this.timeout}ms` };
      }
      return { success: false, error: error.message };
    } finally {
      clearTimeout(timeoutId);
    }
  }

  /**
   * Send POST request
   * @param url URL to post to
   * @param body Request body (string or object)
   * @param headers Optional HTTP headers
   */
  async post(params: {
    url: string;
    body: string | object;
    headers?: Record<string, string>;
  }) {
    const body = typeof params.body === 'string'
      ? params.body
      : JSON.stringify(params.body);

    const headers = {
      'Content-Type': 'application/json',
      'User-Agent': this.userAgent || 'Photon-MCP/1.0',
      ...params.headers,
    };

    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), this.timeout);

    try {
      const response = await fetch(params.url, {
        method: 'POST',
        headers,
        body,
        signal: controller.signal,
      });

      return {
        success: response.ok,
        status: response.status,
        data: await response.text(),
      };
    } catch (error: any) {
      if (error.name === 'AbortError') {
        return { success: false, error: `Request timeout after ${this.timeout}ms` };
      }
      return { success: false, error: error.message };
    } finally {
      clearTimeout(timeoutId);
    }
  }

  /**
   * Download file from URL
   * @param url URL to download from
   * @param outputPath Where to save the file
   */
  async download(params: { url: string; outputPath: string }) {
    try {
      const response = await fetch(params.url);
      if (!response.ok) {
        return {
          success: false,
          error: `HTTP ${response.status}: ${response.statusText}`,
        };
      }

      const buffer = await response.arrayBuffer();
      const { writeFile } = await import('fs/promises');
      await writeFile(params.outputPath, Buffer.from(buffer));

      return {
        success: true,
        path: params.outputPath,
        size: buffer.byteLength,
      };
    } catch (error: any) {
      return { success: false, error: error.message };
    }
  }
}
```

**Key patterns:**
- Use `AbortController` for timeout handling
- Always clear timeout in `finally` block
- Return detailed response info (status, headers, data)
- Handle both string and object request bodies

---

## Database Operations

### Pattern: SQLite Database MCP

```typescript
import Database from 'better-sqlite3';
import { join } from 'path';
import { homedir } from 'os';
import { existsSync, mkdirSync } from 'fs';
import { dirname } from 'path';

export default class Sqlite {
  private db?: Database.Database;

  constructor(
    private dbPath: string = join(homedir(), '.myapp', 'data.db'),
    private readonly: boolean = false
  ) {
    // Ensure directory exists
    const dir = dirname(dbPath);
    if (!existsSync(dir)) {
      mkdirSync(dir, { recursive: true });
    }
  }

  async onInitialize() {
    try {
      this.db = new Database(this.dbPath, {
        readonly: this.readonly,
      });
      console.error(`[sqlite] ✅ Database ready: ${this.dbPath}`);
    } catch (error: any) {
      console.error(`[sqlite] ❌ Failed to open database: ${error.message}`);
      throw error;
    }
  }

  async onShutdown() {
    if (this.db) {
      this.db.close();
      console.error('[sqlite] ✅ Database closed');
    }
  }

  /**
   * Execute SQL query and return results
   * @param sql SQL query to execute
   * @param params Query parameters (optional)
   */
  async query(params: { sql: string; params?: any[] }) {
    if (!this.db) {
      return { success: false, error: 'Database not initialized' };
    }

    try {
      const stmt = this.db.prepare(params.sql);
      const rows = params.params ? stmt.all(...params.params) : stmt.all();

      return {
        success: true,
        rows,
        count: rows.length,
      };
    } catch (error: any) {
      return { success: false, error: error.message };
    }
  }

  /**
   * Execute SQL statement (INSERT, UPDATE, DELETE)
   * @param sql SQL statement to execute
   * @param params Statement parameters (optional)
   */
  async execute(params: { sql: string; params?: any[] }) {
    if (!this.db) {
      return { success: false, error: 'Database not initialized' };
    }

    try {
      const stmt = this.db.prepare(params.sql);
      const result = params.params ? stmt.run(...params.params) : stmt.run();

      return {
        success: true,
        changes: result.changes,
        lastInsertRowid: result.lastInsertRowid,
      };
    } catch (error: any) {
      return { success: false, error: error.message };
    }
  }

  /**
   * Execute multiple statements in a transaction
   * @param statements Array of SQL statements with optional parameters
   */
  async transaction(params: { statements: Array<{ sql: string; params?: any[] }> }) {
    if (!this.db) {
      return { success: false, error: 'Database not initialized' };
    }

    try {
      const results = this.db.transaction(() => {
        return params.statements.map(stmt => {
          const prepared = this.db!.prepare(stmt.sql);
          return stmt.params ? prepared.run(...stmt.params) : prepared.run();
        });
      })();

      return {
        success: true,
        results: results.map(r => ({
          changes: r.changes,
          lastInsertRowid: r.lastInsertRowid,
        })),
      };
    } catch (error: any) {
      return { success: false, error: error.message };
    }
  }
}
```

**Key patterns:**
- Initialize database in `onInitialize()`
- Always close database in `onShutdown()`
- Use parameterized queries to prevent SQL injection
- Provide transaction support for atomic operations
- Create database directory if it doesn't exist

---

## Shell/Git Operations

### Pattern: Git Operations MCP

```typescript
import { exec } from 'child_process';
import { promisify } from 'util';

const execAsync = promisify(exec);

export default class Git {
  constructor(
    private repoPath: string = process.cwd(),
    private timeout: number = 10000
  ) {
    const { existsSync } = require('fs');
    const { join } = require('path');

    if (!existsSync(join(repoPath, '.git'))) {
      throw new Error(`Not a git repository: ${repoPath}`);
    }
  }

  async onInitialize() {
    console.error(`[git] ✅ Repository: ${this.repoPath}`);
  }

  /**
   * Get repository status
   */
  async status(params: {}) {
    return this._exec('git status --porcelain');
  }

  /**
   * Get commit history
   * @param count Number of commits to show (default: 10)
   * @param branch Branch name (default: current branch)
   */
  async log(params: { count?: number; branch?: string }) {
    const count = params.count || 10;
    const branch = params.branch || '';
    return this._exec(`git log ${branch} -n ${count} --oneline`);
  }

  /**
   * Show diff of changes
   * @param staged Show staged changes only (default: false)
   */
  async diff(params: { staged?: boolean }) {
    const cmd = params.staged ? 'git diff --cached' : 'git diff';
    return this._exec(cmd);
  }

  /**
   * Stage files for commit
   * @param files Files to stage (e.g., "." for all, or specific paths)
   */
  async add(params: { files: string }) {
    // Sanitize input to prevent command injection
    const files = params.files.replace(/[;&|`$()]/g, '');
    return this._exec(`git add ${files}`);
  }

  /**
   * Commit staged changes
   * @param message Commit message
   */
  async commit(params: { message: string }) {
    // Escape commit message properly
    const escaped = params.message.replace(/"/g, '\\"');
    return this._exec(`git commit -m "${escaped}"`);
  }

  /**
   * Create or switch branches
   * @param name Branch name
   * @param create Create new branch (default: false)
   */
  async branch(params: { name: string; create?: boolean }) {
    const flag = params.create ? '-b' : '';
    return this._exec(`git checkout ${flag} ${params.name}`);
  }

  // Private helper for executing git commands
  private async _exec(command: string) {
    try {
      const { stdout, stderr } = await execAsync(command, {
        cwd: this.repoPath,
        timeout: this.timeout,
      });

      return {
        success: true,
        output: stdout || stderr,
      };
    } catch (error: any) {
      return {
        success: false,
        error: error.message,
        output: error.stderr || '',
      };
    }
  }
}
```

**Key patterns:**
- Validate repository in constructor
- Use `promisify(exec)` for async shell commands
- Always set `cwd` and `timeout` in exec options
- Sanitize/escape user input to prevent command injection
- Return both stdout and stderr

---

## Security Patterns

### Directory Traversal Prevention

Always validate paths to prevent accessing files outside working directory:

```typescript
private _resolvePath(userPath: string): string {
  const resolved = resolve(this.workdir, userPath);
  const rel = relative(this.workdir, resolved);

  // Block paths that escape working directory
  if (rel.startsWith('..') || resolve(rel) === rel) {
    throw new Error(`Access denied: path outside working directory`);
  }

  return resolved;
}
```

### Command Injection Prevention

**Method 1: Sanitize input**
```typescript
const sanitized = userInput.replace(/[;&|`$()]/g, '');
await exec(`command ${sanitized}`);
```

**Method 2: Use spawn with args array (preferred)**
```typescript
import { spawn } from 'child_process';

const child = spawn('git', ['commit', '-m', userMessage]);
```

### Input Validation

Always validate user inputs:

```typescript
async process(params: { email: string; age: number }) {
  // Validate email format
  if (!params.email.includes('@')) {
    return { success: false, error: 'Invalid email format' };
  }

  // Validate age range
  if (params.age < 0 || params.age > 150) {
    return { success: false, error: 'Age must be between 0 and 150' };
  }

  // Process validated inputs
  return { success: true };
}
```

### File Size Limits

Prevent memory exhaustion:

```typescript
async read(params: { path: string }) {
  const fullPath = this._resolvePath(params.path);
  const stats = await stat(fullPath);

  if (stats.size > this.maxFileSize) {
    return {
      success: false,
      error: `File too large: ${stats.size} bytes (max: ${this.maxFileSize})`,
    };
  }

  const content = await readFile(fullPath, 'utf-8');
  return { success: true, content };
}
```

---

## Error Handling

### Standard Error Response Format

Use consistent error response format:

```typescript
try {
  // Operation
  return { success: true, result: data };
} catch (error: any) {
  return {
    success: false,
    error: error.message,
    code: error.code // Optional error code
  };
}
```

### Detailed Error Responses

Provide helpful error messages:

```typescript
async process(params: { input: string }) {
  // Validation error
  if (!params.input) {
    return {
      success: false,
      error: 'Input is required',
      code: 'MISSING_INPUT',
    };
  }

  try {
    // Process
    return { success: true, result: data };
  } catch (error: any) {
    // Runtime error
    return {
      success: false,
      error: `Processing failed: ${error.message}`,
      code: 'PROCESSING_ERROR',
    };
  }
}
```

### Graceful Degradation

Don't crash on errors:

```typescript
async fetch(params: { url: string }) {
  try {
    return await this._fetchWithRetry(params.url, 3);
  } catch (error: any) {
    // Log but don't throw
    console.error('[fetch] Error:', error);
    return {
      success: false,
      error: 'Service temporarily unavailable',
    };
  }
}
```

---

## Configuration Patterns

### Smart Defaults

Use platform-aware and sensible defaults:

```typescript
import { homedir } from 'os';
import { join } from 'path';

constructor(
  // Cross-platform user documents folder
  private workdir: string = join(homedir(), 'Documents'),

  // Reasonable size limits
  private maxFileSize: number = 10485760, // 10MB
  private timeout: number = 5000, // 5 seconds

  // Conservative security defaults
  private allowHidden: boolean = false,
  private strictMode: boolean = true
) {}
```

### Required Configuration

For required parameters, omit defaults and validate:

```typescript
constructor(
  private apiKey: string,
  private endpoint: string,
  private timeout: number = 5000
) {
  if (!apiKey || apiKey.trim() === '') {
    throw new Error('API key is required');
  }

  if (!endpoint || !endpoint.startsWith('http')) {
    throw new Error('Valid endpoint URL is required');
  }
}
```

### Configuration Logging

Log configuration in `onInitialize()`:

```typescript
async onInitialize() {
  console.error(`[mcp-name] ✅ Initialized`);
  console.error(`[mcp-name] Config:`);
  console.error(`  - Endpoint: ${this.endpoint}`);
  console.error(`  - Timeout: ${this.timeout}ms`);
  console.error(`  - Strict mode: ${this.strictMode}`);
}
```

---

## Additional Tips

1. **Use TypeScript built-in modules** when possible (fs, path, http) to minimize dependencies
2. **Keep tool methods focused** - one tool should do one thing well
3. **Document edge cases** in JSDoc (e.g., "Returns empty array if no files found")
4. **Use meaningful return values** - include metadata like timestamps, counts, paths
5. **Handle cleanup** in `onShutdown()` - close connections, flush buffers, delete temp files
6. **Test error paths** - ensure errors return helpful messages, not just stack traces
