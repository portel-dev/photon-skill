# Generating Photon MCPs Skill

Claude skill for generating single-file TypeScript MCPs for the Photon runtime.

## Installation

### Option 1: Plugin Marketplace (Recommended for Claude Code)

```bash
# Add the marketplace
/plugin marketplace add portel-dev/photon-skill

# Install the skill
/plugin install generating-photon-mcps@photon-skills
```

### Option 2: Manual Installation (Claude Desktop)

1. Download `generating-photon-mcps.zip` from [GitHub Releases](https://github.com/portel-dev/photon-skill/releases)
2. In Claude Desktop, click "Upload skill" and select the ZIP file

### Option 3: Build from Source

```bash
# Clone the repository
git clone https://github.com/portel-dev/photon-skill.git
cd photon-skill

# Package the skill (requires skill-creator tools)
python /path/to/skill-creator/scripts/package_skill.py ./generating-photon-mcps .

# Install the generated ZIP
# Claude Desktop: Upload generating-photon-mcps.zip
# Claude Code: /skill install generating-photon-mcps.zip
```

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

1. Edit `generating-photon-mcps/SKILL.md` for main instructions
2. Update `generating-photon-mcps/references/patterns.md` for implementation patterns
3. Update `generating-photon-mcps/references/restler-integration.md` for multi-runtime usage
4. Test changes locally
5. Create a new release with packaged ZIP:
   ```bash
   # Package the skill
   python /path/to/skill-creator/scripts/package_skill.py ./generating-photon-mcps .

   # Create GitHub release and attach generating-photon-mcps.zip
   ```

**Note:** ZIP files are not committed to the repository (see `.gitignore`). They are distributed via GitHub Releases for manual installation.

## License

MIT
