# Photon + Restler-TS Integration

How Photon MCP files work with the Restler-TS framework for multi-protocol deployment.

## Overview

**Photon** creates single-file TypeScript utilities that work as MCP servers.
**Restler-TS** is a multi-protocol API framework that can expose these utilities as REST/GraphQL/JSON-RPC/WebSocket APIs.

Together: **Write once (Photon), deploy everywhere (Restler).**

## Architecture

```
┌─────────────────────────┐
│  .photon.ts File        │
│  (Pure TypeScript)      │
└────────┬────────────────┘
         │
         ├──────────────────┬──────────────────┬───────────────┐
         ↓                  ↓                  ↓               ↓
    [@portel/photon]   [Restler-TS]     [Direct Import]  [NPM Package]
    (MCP Server)       (REST/GraphQL)   (Node.js App)    (Published)
```

## Use Case 1: Photon for MCPs, Restler for APIs

### Step 1: Create Photon MCP
```typescript
// filesystem.photon.ts
export default class Filesystem {
  constructor(
    private workdir: string = join(homedir(), 'Documents')
  ) {}

  async list(params: { path?: string }) {
    // Implementation
    return { success: true, files: [...] };
  }

  async read(params: { path: string }) {
    // Implementation
    return { success: true, content: '...' };
  }
}
```

### Step 2: Use with Photon Runtime (MCP)
```bash
npx photon filesystem --dev
# → MCP server for Claude Desktop
```

### Step 3: Use with Restler (REST API)
```typescript
// server.ts
import { Restler } from 'restler-ts';
import Filesystem from './filesystem.photon.ts';

const app = new Restler();

// Register Photon class as API
app.register(Filesystem);

app.listen(3000);
```

**Result:**
```bash
POST /filesystem/list
POST /filesystem/read
# Same class, now as REST endpoints!
```

## Use Case 2: Restler API → Photon MCP

You can also go the other direction: take a Restler API and expose it as MCP.

### Restler API Class
```typescript
// users.api.ts
import { Get, Post, Path, Body } from 'restler-ts';

export class UsersApi {
  @Get(':id')
  async get(@Path('id') id: string) {
    return { id, name: 'John' };
  }

  @Post()
  async create(@Body() data: { name: string }) {
    return { id: '123', ...data };
  }
}
```

### Wrap for Photon
```typescript
// users.photon.ts
import { UsersApi } from './users.api';

export default class Users {
  private api = new UsersApi();

  /**
   * Get user by ID
   * @param id User ID
   */
  async get(params: { id: string }) {
    return this.api.get(params.id);
  }

  /**
   * Create new user
   * @param name User name
   */
  async create(params: { name: string }) {
    return this.api.create(params);
  }
}
```

## Comparison

| Feature | Photon Alone | Restler Alone | Photon + Restler |
|---------|--------------|---------------|------------------|
| **MCP Support** | ✅ Native | ⚠️ Via module | ✅ Best of both |
| **REST API** | ❌ | ✅ Native | ✅ Best of both |
| **GraphQL** | ❌ | ✅ Native | ✅ Via Restler |
| **Decorators** | ❌ Plain classes | ✅ @Get, @Post | 🔄 Optional |
| **File Count** | 1️⃣ Single file | 📦 Multiple | 1️⃣ or 📦 Your choice |
| **Learning Curve** | 🟢 Easy | 🟡 Moderate | 🟢 Start easy, grow |

## Pattern Recommendations

### When to use Photon Only
- ✅ Building MCP servers
- ✅ Quick prototypes
- ✅ Personal automation tools
- ✅ AI agent utilities

### When to use Restler Only
- ✅ Building production APIs
- ✅ Need GraphQL/WebSocket
- ✅ Complex routing
- ✅ Middleware pipelines

### When to use Both
- ✅ Multi-protocol deployment
- ✅ Start simple (MCP), scale later (API)
- ✅ Team has both AI agents and web clients
- ✅ Maximum flexibility

## Example: Full-Stack Photon File

A single `.photon.ts` file that works with both runtimes:

```typescript
/**
 * Tasks - Task management system
 *
 * Works with:
 * - Photon: npx photon tasks --dev
 * - Restler: app.register(Tasks)
 * - Direct: import Tasks from './tasks.photon.ts'
 *
 * Dependencies: None
 */

interface Task {
  id: string;
  title: string;
  completed: boolean;
}

export default class Tasks {
  private tasks: Map<string, Task> = new Map();

  constructor(
    private storePath?: string
  ) {}

  /**
   * List all tasks
   * @param completed Filter by completion status
   */
  async list(params: { completed?: boolean }) {
    let tasks = Array.from(this.tasks.values());

    if (params.completed !== undefined) {
      tasks = tasks.filter(t => t.completed === params.completed);
    }

    return { success: true, tasks, count: tasks.length };
  }

  /**
   * Create a new task
   * @param title Task title
   */
  async create(params: { title: string }) {
    const task: Task = {
      id: `task-${Date.now()}`,
      title: params.title,
      completed: false
    };

    this.tasks.set(task.id, task);

    return { success: true, task };
  }

  /**
   * Mark task as complete
   * @param id Task ID
   */
  async complete(params: { id: string }) {
    const task = this.tasks.get(params.id);

    if (!task) {
      return { success: false, error: 'Task not found' };
    }

    task.completed = true;

    return { success: true, task };
  }

  /**
   * Delete a task
   * @param id Task ID
   */
  async delete(params: { id: string }) {
    const deleted = this.tasks.delete(params.id);

    if (!deleted) {
      return { success: false, error: 'Task not found' };
    }

    return { success: true, message: 'Task deleted' };
  }
}
```

### Usage with Photon
```bash
npx photon tasks --dev
# MCP server running
```

### Usage with Restler
```typescript
import { Restler } from 'restler-ts';
import Tasks from './tasks.photon.ts';

const app = new Restler();
app.register(Tasks);
app.listen(3000);

// Now available as:
// POST /tasks/list
// POST /tasks/create
// POST /tasks/complete
// POST /tasks/delete
```

### Usage Direct
```typescript
import Tasks from './tasks.photon.ts';

const tasks = new Tasks();
await tasks.create({ title: 'Buy milk' });
const all = await tasks.list({});
console.log(all.tasks);
```

## Best Practices

### 1. Keep Classes Pure
Don't add framework-specific code:

❌ **Bad:**
```typescript
export default class MyTool {
  private _photonConfigError: string;  // Framework-specific!
  private _restlerContext: any;        // Framework-specific!
}
```

✅ **Good:**
```typescript
export default class MyTool {
  // Pure TypeScript, no framework coupling
  constructor(private config: string) {}

  async doSomething(params: { input: string }) {
    return { success: true, result: '...' };
  }
}
```

### 2. Use Standard Return Formats
Both frameworks understand this:

```typescript
// Success
return { success: true, data: result };

// Error
return { success: false, error: 'Message' };
```

### 3. Document Everything
JSDoc works for both:

```typescript
/**
 * Create a new record
 * @param name Record name
 * @param data Record data
 */
async create(params: { name: string; data: any }) {
  // Photon: auto-generates MCP schema
  // Restler: auto-generates OpenAPI docs
}
```

### 4. Type Everything
TypeScript types work for both:

```typescript
async fetch(params: {
  url: string;           // Required
  method?: string;       // Optional
  headers?: Record<string, string>;  // Complex type
}) {
  // Both frameworks validate at runtime
}
```

## Migration Paths

### Photon → Restler
1. Start with Photon MCP for quick prototyping
2. Add `@Get()`, `@Post()` decorators when ready
3. Register with Restler for API deployment
4. Original MCP still works!

### Restler → Photon
1. Start with Restler API for web services
2. Extract business logic to .photon.ts
3. Use Photon runtime for MCP server
4. Original API still works!

## Conclusion

**Photon** = Convention for single-file TypeScript utilities
**Restler** = Framework for multi-protocol APIs

Together = **Maximum flexibility with minimum coupling**

Write once, deploy everywhere:
- ✅ MCP servers (AI agents)
- ✅ REST APIs (web/mobile)
- ✅ GraphQL (modern frontends)
- ✅ WebSocket (real-time)
- ✅ JSON-RPC (services)
- ✅ Direct import (Node.js apps)
- ✅ NPM packages (reusable)

---

**The future is multi-protocol** 🚀
