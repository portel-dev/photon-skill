# Restler-TS Integration Reference

How `.photon.ts` files can be deployed across multiple runtimes using the Restler-TS framework.

## Overview

Photon files are **framework-agnostic TypeScript classes**. This means the same `.photon.ts` file can be:
1. Run as an MCP server with Photon
2. Exposed as a REST API with Restler-TS
3. Used as a GraphQL API with Restler-TS
4. Imported directly in any TypeScript project

This "write once, deploy everywhere" pattern maximizes code reuse.

## What is Restler-TS?

Restler-TS is a multi-protocol framework that automatically exposes TypeScript classes as:
- REST APIs (HTTP endpoints)
- GraphQL APIs (queries/mutations)
- WebSocket APIs (real-time)
- Direct imports (library usage)

## Integration Pattern

### Same Class, Multiple Runtimes

```typescript
// calculator.photon.ts
export default class Calculator {
  /**
   * Add two numbers
   * @param a First number
   * @param b Second number
   */
  async add(params: { a: number; b: number }) {
    return params.a + params.b;
  }
}
```

**Usage 1: MCP Server**
```bash
photon calculator --dev
# Exposed via MCP protocol for Claude Desktop
```

**Usage 2: REST API (Restler-TS)**
```typescript
// server.ts
import { RestlerApp } from '@restler/core';
import Calculator from './calculator.photon.ts';

const app = new RestlerApp();
app.registerService(Calculator);
app.listen(3000);

// Creates endpoint: POST /calculator/add
// Request: { "a": 5, "b": 3 }
// Response: 8
```

**Usage 3: GraphQL API (Restler-TS)**
```typescript
import { GraphQLApp } from '@restler/graphql';
import Calculator from './calculator.photon.ts';

const app = new GraphQLApp();
app.registerService(Calculator);

// Creates GraphQL mutation:
// mutation { add(a: 5, b: 3) }
```

**Usage 4: Direct Import**
```typescript
import Calculator from './calculator.photon.ts';

const calc = new Calculator();
const result = await calc.add({ a: 5, b: 3 });
```

## When to Mention Restler-TS

**Mention it when:**
- User asks about using Photon files in other ways
- User wants to expose MCP functionality as an API
- User is building a web service
- User mentions multi-protocol deployment

**Don't mention it when:**
- User just wants a simple MCP
- No indication of API/web service needs
- Focused solely on Claude Desktop integration

## Example: Multi-Protocol Deployment

```typescript
// github-issues.photon.ts
export default class GitHubIssues {
  constructor(
    private token: string,
    private repo: string
  ) {}

  async list(params: { state?: 'open' | 'closed' }) {
    // Implementation
  }

  async create(params: { title: string; body: string }) {
    // Implementation
  }
}
```

**Deployment Options:**

1. **MCP (Photon)**
   ```bash
   photon github-issues
   # Used by Claude Desktop via MCP protocol
   ```

2. **REST API (Restler-TS)**
   ```bash
   restler serve github-issues.photon.ts
   # Endpoints:
   # GET  /github-issues/list?state=open
   # POST /github-issues/create
   ```

3. **GraphQL (Restler-TS)**
   ```graphql
   query {
     list(state: "open") {
       title
       number
     }
   }

   mutation {
     create(title: "Bug", body: "Description")
   }
   ```

## Benefits of Framework-Agnostic Design

1. **Code Reuse** - Write once, deploy multiple ways
2. **Flexibility** - Change deployment without changing code
3. **Testing** - Test as plain TypeScript class
4. **Migration** - Easy to move between protocols
5. **Composition** - Combine services in larger systems

## Guidelines for Photon Files

To ensure compatibility across runtimes:

1. **Pure TypeScript** - No framework-specific code
2. **Single Params Object** - `async method(params: { ... })`
3. **Async Methods** - All public methods should be async
4. **Standard Returns** - Simple values or `{ success, ... }` format
5. **Constructor Config** - Use constructor parameters for configuration

## When NOT to Use This Pattern

- MCP-specific features (resources, prompts) - These only work in Photon
- Real-time streaming - May need runtime-specific handling
- Binary data - May need different serialization per protocol
- Complex authentication - May need per-runtime auth strategies

## Summary

Photon files are **just TypeScript classes**. This simplicity enables deployment flexibility:
- Photon runtime → MCP protocol
- Restler-TS runtime → REST/GraphQL/WebSocket
- Direct import → Library usage

Keep files framework-agnostic to maximize reusability across all runtimes.
