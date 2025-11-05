# Photon Skill - AI-Powered MCP Generator

A Claude skill for generating production-ready Photon MCP files from natural language descriptions.

## What This Skill Does

Enables Claude to generate complete `.photon.ts` files (Model Context Protocol servers) in seconds, based on natural language descriptions.

**Input:** "Create an MCP to manage my GitHub issues"
**Output:** Complete, production-ready `github-issues.photon.ts` file

## Installation

### For Claude Desktop

1. Copy this folder to your Claude skills directory:
```bash
cp -r photon-skill ~/.claude/plugins/marketplaces/anthropic-agent-skills/
```

2. Restart Claude Desktop

3. The skill activates automatically when you request MCP generation

### For Claude Code / CLI

The skill is automatically available when this repository is in your workspace.

## Usage Examples

### Example 1: Simple File Operations
```
User: Create an MCP to organize my Downloads folder
```

Claude generates a complete `downloads-organizer.photon.ts` with tools to:
- List files by type
- Move files to categorized folders
- Delete old files
- Search for files

### Example 2: API Integration
```
User: Create an MCP for managing Notion pages
```

Claude generates `notion-manager.photon.ts` with:
- Constructor requiring Notion API token
- Tools for CRUD operations on pages
- Database queries
- Content formatting

### Example 3: Automation
```
User: Build an MCP to backup my important folders daily
```

Claude generates `backup-manager.photon.ts` with:
- Configurable source directories
- Backup destination settings
- Compression options
- Scheduling utilities

## What Gets Generated

Every Photon MCP includes:

✅ **Complete header documentation**
- Description and use cases
- Configuration parameters
- Dependencies listing

✅ **Type-safe constructor**
- Smart defaults (~/Documents, API endpoints, etc.)
- Validation logic
- Environment variable mapping

✅ **Production-ready tools**
- JSDoc documentation
- TypeScript type safety
- Error handling
- Consistent return formats

✅ **Helper methods**
- Private methods (prefixed with `_`)
- Reusable utilities
- Security validation

✅ **Lifecycle hooks**
- `onInitialize()` for setup
- `onShutdown()` for cleanup

## Generated File Structure

```typescript
/**
 * [Name] - [Description]
 *
 * Configuration, use cases, dependencies documented here
 */

import { ... } from '...';

export default class MyTool {
  constructor(
    private config: string = defaultValue
  ) {
    // Validation
  }

  async onInitialize() {
    // Setup
  }

  /**
   * Tool description
   * @param input Description
   */
  async toolName(params: { input: string }) {
    // Implementation with error handling
    return { success: true, result: ... };
  }

  private _helper() {
    // Private utilities
  }
}
```

## Key Features

### 🎯 AI-Optimized Patterns
- Designed specifically for AI generation
- Consistent conventions
- Predictable structure
- Clear documentation requirements

### 🔒 Security Built-In
- Path validation templates
- Input sanitization patterns
- API key handling
- Whitelist-based command execution

### 📦 Framework-Agnostic
- No Photon-specific code in generated files
- Pure TypeScript classes
- Works with any runtime (MCP, CLI, REST, etc.)
- Reusable as npm packages

### ⚡ Instant Results
- Complete MCP in one response
- Copy-paste ready
- Test immediately with `npx photon [name] --dev`
- Deploy with `npx photon [name] --config`

## Common Patterns Included

The skill knows how to generate:

### File System Operations
- Path validation and security
- Recursive directory traversal
- File type detection
- Glob pattern matching

### HTTP/API Calls
- RESTful API interactions
- Authentication headers
- Error handling
- Response formatting

### Database Operations
- SQLite integration
- Connection management
- Prepared statements
- Transaction handling

### Shell Commands
- Safe command execution
- Output capture
- Working directory handling
- Async execution

## Quality Standards

Every generated file includes:

- ✅ Complete TypeScript typing
- ✅ JSDoc documentation on all public methods
- ✅ Error handling in try-catch blocks
- ✅ Security validation where needed
- ✅ Consistent return formats ({ success, ... })
- ✅ Smart default values in constructor
- ✅ Clear, actionable error messages

## Integration with Other Tools

### Photon MCP Runtime
```bash
npx photon my-tool --dev      # Development mode
npx photon my-tool --config   # Generate MCP config
```

### Restler-TS Framework
Generated .photon.ts classes can be loaded by restler-ts to expose as:
- REST APIs
- GraphQL endpoints
- JSON-RPC services
- WebSocket servers

### Direct Import
```typescript
import MyTool from './my-tool.photon.ts';

const tool = new MyTool({ config: 'value' });
const result = await tool.someMethod({ input: 'data' });
```

## Examples Gallery

See the `examples/` directory for complete, working Photon MCPs:

- `github-issues.photon.ts` - GitHub issue management
- `file-organizer.photon.ts` - Smart file organization
- `notion-tasks.photon.ts` - Notion database integration
- `backup-manager.photon.ts` - Automated backup system

## Development

This skill is part of the Photon ecosystem:

- **Photon Runtime**: [@portel/photon](https://github.com/portel-dev/photon-mcp)
- **Photon Hub**: Community MCP registry (coming soon)
- **Restler-TS**: Multi-protocol API framework

## Contributing

Contributions welcome! To improve the skill:

1. Edit `SKILL.md` to add new patterns or examples
2. Add example `.photon.ts` files to `examples/`
3. Update documentation in README.md
4. Test generation quality with various prompts

## License

MIT © Portel

---

**Built for the future of AI-powered development** 🚀
