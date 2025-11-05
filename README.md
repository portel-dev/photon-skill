# Generating Photon MCPs Skill

Claude skill for generating single-file TypeScript MCPs for the Photon runtime.

## Installation

1. Download `generating-photon-mcps.zip`
2. Install in Claude Desktop or Claude Code:
   - Claude Desktop: Click "Upload skill" and select the ZIP file
   - Claude Code: Use slash command `/skill install generating-photon-mcps.zip`

## What This Skill Does

Enables rapid generation of production-quality `.photon.ts` files:
- Creates complete MCP servers from simple requests
- Includes security best practices (input validation, path traversal prevention)
- Provides error handling patterns
- Generates comprehensive JSDoc documentation
- Uses smart platform-aware defaults

## Usage

Simply ask Claude to create an MCP:

- "Create a GitHub Issues MCP"
- "Build an MCP for file operations"
- "Make an MCP that interacts with SQLite"
- "Generate an MCP for time and date utilities"

Claude will:
1. Ask clarifying questions if needed
2. Generate a complete `.photon.ts` file with proper structure
3. Include security helpers and error handling
4. Provide usage instructions

## What Gets Generated

Each generated `.photon.ts` file includes:

- **Comprehensive file header** with JSDoc
- **Smart constructor defaults** (e.g., `~/Documents` for filesystem)
- **Environment variable configuration** (automatically mapped)
- **Security patterns** (directory traversal prevention, input validation)
- **Error handling** with `{ success: boolean, ... }` format
- **Lifecycle hooks** (`onInitialize`, `onShutdown`)
- **Well-documented tools** with JSDoc and TypeScript types

## Patterns Included

The skill includes reference patterns for:

- **Filesystem operations** - Read, write, list, search, delete
- **HTTP/API operations** - GET, POST, download, timeout handling
- **Database operations** - SQLite with transactions
- **Shell/Git operations** - Command execution with security
- **Security best practices** - Input validation, command injection prevention
- **Error handling** - Consistent error responses
- **Configuration** - Smart defaults, environment variables

## Framework-Agnostic Design

Generated `.photon.ts` files are pure TypeScript classes that work with:
- **Photon runtime** → MCP protocol (Claude Desktop, Cursor, Windsurf)
- **Restler-TS runtime** → REST/GraphQL/WebSocket APIs
- **Direct import** → Use as a library

## Example

**Request:** "Create an MCP for time operations"

**Result:** Complete `time.photon.ts` file with:
- Timezone conversion
- Date formatting
- Relative time calculations
- Smart defaults (system timezone)
- Full JSDoc documentation

## Development

To modify this skill:

1. Edit `photon/SKILL.md` for main instructions
2. Update `photon/references/patterns.md` for implementation patterns
3. Update `photon/references/restler-integration.md` for multi-runtime usage
4. Repackage with `package_skill.py`

## License

MIT
