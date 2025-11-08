# Generating Photon MCPs Skill

**AI-powered skill for generating single-file, production-ready MCP servers.**

Build complete MCP servers in seconds—one file, zero boilerplate, pure business logic.

---

## What This Skill Does

Generate production-quality `.photon.ts` files from simple requests:

- ✅ **One file = Complete MCP** - Single TypeScript file becomes full MCP server
- ✅ **AI-friendly code** - Clear, readable structure optimized for AI understanding and modification
- ✅ **Auto-dependencies** - Dependencies auto-install via `@dependencies` tags (like `npx` or `uv`)
- ✅ **Fork-first design** - Easy to customize, copy, and modify
- ✅ **MCP-client agnostic** - Works with Claude Desktop, Cursor, Zed, Continue, Cline, and any MCP client
- ✅ **Security built-in** - Input validation, path traversal prevention, command injection protection

**From idea to working MCP server in under 60 seconds.**

---

## Installation

### Option 1: Plugin Marketplace (Recommended for Claude Code)

```bash
# Add the marketplace
/plugin marketplace add portel-dev/photon-skill

# Install the skill
/plugin install generating-photon-mcps@photon-skills
```

### Option 2: Manual Installation (Any MCP Client)

1. Download `generating-photon-mcps.zip` from [GitHub Releases](https://github.com/portel-dev/photon-skill/releases)
2. In your MCP client, upload the skill (consult client documentation for installation steps)

### Option 3: Build from Source

```bash
# Clone the repository
git clone https://github.com/portel-dev/photon-skill.git
cd photon-skill

# Package the skill (requires skill-creator tools)
python /path/to/skill-creator/scripts/package_skill.py ./generating-photon-mcps .

# Install the generated ZIP file in your MCP client
```

---

## Usage

Simply ask your AI assistant to create an MCP:

**Examples:**
- "Create a GitHub Issues MCP"
- "Build an MCP for file operations"
- "Make an MCP that interacts with PostgreSQL"
- "Generate an MCP for time and date utilities"

**The skill will:**
1. Ask clarifying questions if needed
2. Generate a complete `.photon.ts` file with proper structure
3. Include security helpers and error handling
4. Provide auto-installing dependencies
5. Give usage instructions for any MCP client

---

## Why Single-File MCPs?

### 🤖 AI-Friendly

AI agents can understand your entire MCP in one context:

```
"Read my github.photon.ts and explain how it works"
"Review this photon for security issues"
"Add error handling to this tool method"
```

Traditional MCPs scatter logic across files—Photons give complete context.

### 👤 Human-Friendly

- **Understand**: Read one file, understand the whole system
- **Review**: Code reviews are one file, complete story
- **Debug**: All logic in one place
- **Share**: One file to copy, fork, or send

### 🔧 Fork-First Philosophy

Every generated photon is designed to be customized:

```bash
# Generate base photon
"Create a Jira MCP"

# Save it
# Copy and customize
cp ~/.photon/jira.photon.ts ~/.photon/my-jira.photon.ts
# Edit my-jira.photon.ts to add company auth, custom fields, etc.

# Run your custom version
photon mcp my-jira
```

**No build configs. No package.json. Just code.**

---

## What Gets Generated

Each `.photon.ts` file includes:

### 1. File Header with Auto-Dependencies

```typescript
/**
 * GitHub Issues - Issue tracking operations
 *
 * Common use cases:
 * - Create issues: "Create a bug report"
 * - List issues: "Show open issues"
 * - Update issues: "Close issue #42"
 *
 * Configuration:
 * - token: GitHub personal access token (required)
 * - owner: Repository owner (required)
 * - repo: Repository name (required)
 *
 * Dependencies are auto-installed on first run (like npx or uv).
 *
 * @dependencies @octokit/rest@^19.0.0
 * @version 1.0.0
 * @author Your Name
 * @license MIT
 */
```

### 2. Smart Constructor with Defaults

```typescript
export default class GitHubIssues {
  constructor(
    private token: string,
    private owner: string,
    private repo: string
  ) {}
  // Maps to: GITHUB_ISSUES_TOKEN, GITHUB_ISSUES_OWNER, GITHUB_ISSUES_REPO
}
```

### 3. Security-First Patterns

- ✅ Input validation
- ✅ Directory traversal prevention
- ✅ Command injection protection
- ✅ File size limits
- ✅ Parameterized queries

### 4. Consistent Error Handling

```typescript
async createIssue(params: { title: string; body: string }) {
  try {
    const result = await this.octokit.issues.create({ ... });
    return { success: true, issue: result.data };
  } catch (error) {
    console.error('Failed to create issue:', error);
    return { success: false, error: error.message };
  }
}
```

### 5. Lifecycle Hooks

```typescript
async onInitialize() {
  // Called when photon loads
  this.octokit = new Octokit({ auth: this.token });
  console.error('GitHub Issues photon initialized');
}

async onShutdown() {
  // Called on shutdown
  console.error('GitHub Issues photon shutting down');
}
```

### 6. Well-Documented Tools

```typescript
/**
 * Create a new issue
 * @param title Issue title
 * @param body Issue description
 * @param labels Optional labels
 */
async createIssue(params: {
  title: string;
  body: string;
  labels?: string[];
}) {
  // Implementation...
}
```

---

## Included Patterns

The skill includes reference patterns for:

| Pattern | Use Cases |
|---------|-----------|
| **Filesystem** | Read, write, list, search, delete files |
| **HTTP/API** | GET, POST, download, timeout handling |
| **Database** | SQLite, PostgreSQL, MongoDB with transactions |
| **Shell/Git** | Command execution with security |
| **Security** | Input validation, injection prevention |
| **Error Handling** | Consistent error responses |
| **Configuration** | Smart defaults, environment variables |

---

## MCP Client Compatibility

Generated `.photon.ts` files work with **any MCP client**:

- **Claude Desktop** (Anthropic)
- **Cursor** (IDE)
- **Zed** (IDE)
- **Continue** (VS Code extension)
- **Cline** (VS Code extension)
- Any MCP-compatible client

### Running the Generated MCP

```bash
# Development mode (hot reload)
photon mcp github-issues --dev

# Production mode
photon mcp github-issues

# Get config for your MCP client
photon get github-issues --mcp
```

Add the configuration output to your MCP client's config file. Consult your client's documentation for specific setup instructions.

---

## Examples

### Example 1: Time Operations

**Request:** "Create an MCP for time operations"

**Result:** Complete `time.photon.ts` with:
- Timezone conversion
- Date formatting
- Relative time calculations
- Smart defaults (system timezone)
- Full documentation

### Example 2: Database Operations

**Request:** "Build a PostgreSQL MCP"

**Result:** Complete `postgres.photon.ts` with:
- Query execution
- Parameterized queries
- Transaction support
- Schema introspection
- Connection pooling
- Auto-dependency: `@dependencies pg@^8.11.0`

### Example 3: Custom Integration

**Request:** "Make an MCP for our company CRM API"

**Result:** Complete `crm.photon.ts` with:
- API authentication
- CRUD operations
- Error handling
- Rate limiting
- Custom business logic

---

## Fork and Customize Workflow

### 1. Generate Base Photon

```
You: "Create a Jira MCP"
AI: [Generates jira.photon.ts]
```

### 2. Test It

```bash
photon mcp jira --dev
# Test with your MCP client
```

### 3. Fork for Customization

```bash
cp ~/.photon/jira.photon.ts ~/.photon/company-jira.photon.ts
```

### 4. Modify

Edit `company-jira.photon.ts` to add:
- Company-specific authentication
- Custom field mappings
- Internal workflow logic
- Additional tools

### 5. Use Your Custom Version

```bash
photon mcp company-jira
```

**No build config changes needed. No package.json to update. Just code.**

---

## Development

To modify this skill:

1. **Edit instructions**: `generating-photon-mcps/SKILL.md`
2. **Update patterns**: `generating-photon-mcps/references/patterns.md`
3. **Test changes** locally
4. **Package for release**:
   ```bash
   python /path/to/skill-creator/scripts/package_skill.py ./generating-photon-mcps .
   # Create GitHub release and attach generating-photon-mcps.zip
   ```

**Note:** ZIP files are not committed to the repository (see `.gitignore`). They are distributed via GitHub Releases.

---

## Contributing

Contributions welcome! Submit issues and PRs at [github.com/portel-dev/photon-skill](https://github.com/portel-dev/photon-skill).

---

## Related Projects

- **[photon-mcp](https://github.com/portel-dev/photon-mcp)** - Photon runtime for MCP servers
- **[photons](https://github.com/portel-dev/photons)** - Official marketplace with 16+ production-ready photons

---

## License

MIT

---

**Build focused MCPs. One file at a time.**

Made with ⚛️ by [Portel](https://github.com/portel-dev)
