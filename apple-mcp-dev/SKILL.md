---
name: apple-mcp-dev
description: Use when building, debugging, or publishing Apple MCP servers (AppleScript/JXA → MCP tools for macOS apps). Triggers on "new MCP", "apple mcp", "AppleScript MCP", "build mcp server", "add mcp tool", or when working in any apple-*-mcp repo.
---

# Apple MCP Development — Patterns & Playbook

Build MCP servers that control macOS apps via AppleScript/JXA, following the `@zyx1121/apple-*-mcp` series conventions.

## Project Structure

```
apple-{app}-mcp/
├── src/
│   ├── index.ts              # Entry point — platform check + StdioServerTransport
│   ├── server.ts             # Create McpServer, register tools
│   ├── applescript.ts        # runAppleScript(), runJxa(), escapeForAppleScript()
│   ├── helpers.ts            # success(), error(), withErrorHandling()
│   └── tools/
│       └── {domain}.ts       # Tool definitions grouped by domain
├── package.json              # @zyx1121/{name}, bin, prepublishOnly: npm run build
├── tsconfig.json             # ES2022, Node16, strict
├── .gitignore                # node_modules/, dist/, .env*
├── LICENSE                   # MIT
└── README.md                 # Install, Prerequisites, Tools table, Examples, Limitations
```

## Tech Stack

- **Language**: TypeScript (ES2022 target, Node16 module)
- **MCP SDK**: `@modelcontextprotocol/sdk` (^1.12.1)
- **Validation**: `zod` (^3.25.0)
- **macOS automation**: `osascript` (AppleScript) + `osascript -l JavaScript` (JXA)
- **No other dependencies**

## Core Files

### `src/applescript.ts`

```typescript
import { execFile } from "node:child_process";

const TIMEOUT_MS = 60_000;

export class AppError extends Error {
  constructor(message: string) { super(message); this.name = "AppError"; }
}

// AppleScript execution
export async function runAppleScript(script: string): Promise<string> {
  return new Promise((resolve, reject) => {
    execFile("osascript", ["-e", script], { timeout: TIMEOUT_MS }, (err, stdout, stderr) => {
      if (err) {
        const msg = stderr || err.message;
        if (msg.includes("not running") || msg.includes("-600"))
          return reject(new AppError("App is not running."));
        if (msg.includes("not allowed") || msg.includes("not permitted"))
          return reject(new AppError("Permission denied. Grant in System Settings > Privacy > Automation."));
        if (msg.includes("timed out") || (err as NodeJS.ErrnoException).code === "ETIMEDOUT")
          return reject(new AppError("AppleScript execution timed out."));
        return reject(new AppError(msg.trim()));
      }
      resolve(stdout.trimEnd());
    });
  });
}

// JXA execution — use when AppleScript has keyword conflicts or needs better object introspection
export async function runJxa(script: string): Promise<string> {
  return new Promise((resolve, reject) => {
    execFile("osascript", ["-l", "JavaScript", "-e", script], { timeout: TIMEOUT_MS }, (err, stdout, stderr) => {
      if (err) return reject(new AppError((stderr || err.message).trim()));
      resolve(stdout.trimEnd());
    });
  });
}

export function escapeForAppleScript(str: string): string {
  return str.replace(/\\/g, "\\\\").replace(/"/g, '\\"');
}
```

### `src/helpers.ts`

```typescript
export function success(data: unknown) {
  return { content: [{ type: "text" as const, text: JSON.stringify(data, null, 2) }] };
}

export function error(message: string) {
  return { content: [{ type: "text" as const, text: JSON.stringify({ error: message }) }], isError: true as const };
}

export function withErrorHandling<T extends Record<string, unknown>>(
  fn: (args: T) => Promise<ReturnType<typeof success>>,
) {
  return async (args: T) => {
    try { return await fn(args); }
    catch (e) {
      if (e instanceof AppError) return error(e.message);
      if (e instanceof Error) return error(e.message);
      return error(String(e));
    }
  };
}
```

### `src/index.ts`

```typescript
#!/usr/bin/env node
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { createServer } from "./server.js";

if (process.platform !== "darwin") {
  console.error("This MCP server only runs on macOS.");
  process.exit(1);
}

const server = createServer();
const transport = new StdioServerTransport();
await server.connect(transport);
```

### Tool Registration Pattern

```typescript
server.registerTool(
  "tool_name",
  {
    description: "What it does",
    inputSchema: z.object({
      param: z.string().describe("Description"),
      optional_param: z.coerce.number().optional().describe("Description"),
    }),
  },
  withErrorHandling(async ({ param, optional_param }) => {
    const script = `tell application "App" to ...`;
    const raw = await runAppleScript(script);
    return success({ result: raw });
  }),
);
```

## AppleScript Gotchas

### 1. Reserved Keywords

AppleScript has reserved words (`and`, `or`, `not`, `in`, `of`, `the`, etc.) that break enum constants.

**Problem**: `slide layout title and content` → syntax error because `and` is a keyword.

**Solution**: Check the app's `.sdef` file for the real enum names:
```bash
find "$(osascript -e 'POSIX path of (path to application "AppName")')Contents/" -name "*.sdef"
grep -A2 "your_enum" /path/to/App.sdef
```

### 2. Newlines in Text

JSON sends `\n` as actual newline chars (0x0A). AppleScript can't have literal newlines inside strings.

**Solution**: Replace newlines with `" & return & "` in the AppleScript string:
```typescript
function prepareText(text: string): string {
  return escapeForAppleScript(text)
    .replace(/\n/g, '" & return & "')       // actual newlines
    .replace(/\\\\n/g, '" & return & "');   // literal \n after escape
}
```

### 3. Data Separators

Use ASCII 30 (Record Separator) for multi-record output, Tab for fields:
```typescript
const RS = "\u001e";  // record separator
const FS = "\u001f";  // field separator
// AppleScript: set output to output & field1 & "\\t" & field2 & "${RS}"
```

### 4. Sandbox Paths

Some macOS apps (PowerPoint, etc.) run in a sandbox. File writes to `/tmp/foo` actually land at `~/Library/Containers/com.microsoft.Powerpoint/Data/tmp/foo`.

**Solution**: Write to a known path, then `copyFileSync` from sandbox to real target:
```typescript
import { copyFileSync, existsSync } from "node:fs";
import { homedir } from "node:os";
import { join } from "node:path";

const SANDBOX = join(homedir(), "Library/Containers/com.app.identifier/Data");

function copySandboxFile(requestedPath: string, realTarget: string): string {
  const sandboxPath = join(SANDBOX, requestedPath);
  if (existsSync(sandboxPath)) { copyFileSync(sandboxPath, realTarget); return realTarget; }
  if (existsSync(requestedPath)) { copyFileSync(requestedPath, realTarget); return realTarget; }
  return sandboxPath;
}
```

### 5. AppleScript vs JXA — When to Use Which

| Use Case | AppleScript | JXA |
|----------|-------------|-----|
| Simple CRUD (create, read, set text) | ✅ Best | OK |
| Enum constants with reserved words | ❌ Breaks | ✅ Works |
| Bullet points, paragraph format | ❌ No API | ✅ Works |
| Object introspection / counting | Sometimes broken | ✅ More reliable |
| Complex nested `tell` blocks | ✅ Natural syntax | Verbose |

### 6. Observability Pattern

Always add a `preview` tool that exports to PDF/image so the AI agent can self-verify:
```typescript
server.registerTool("app_preview", {
  description: "Export current state to /tmp for visual inspection.",
  inputSchema: z.object({}),
}, withErrorHandling(async () => {
  // Export → copy from sandbox → return path
  return success({ path: "/tmp/app-preview.pdf", hint: "Read this PDF to see the result." });
}));
```

## Development Workflow

```bash
# 1. Scaffold
mkdir apple-{app}-mcp && cd apple-{app}-mcp
npm init -y
npm install @modelcontextprotocol/sdk zod
npm install -D @types/node typescript

# 2. Test AppleScript interactively
osascript -e 'tell application "App" to ...'     # AppleScript
osascript -l JavaScript -e 'var app = ...'        # JXA

# 3. Build & test locally
npm run build
claude mcp add apple-{app} -- node $(pwd)/dist/index.js   # local dev
# Restart Claude Code to load MCP → test tools

# 4. Iterate: edit src → npm run build → restart Claude → test
```

## Publishing Checklist

```bash
# 1. Ensure package.json has:
#    - "name": "@zyx1121/apple-{app}-mcp"
#    - "bin": { "apple-{app}-mcp": "dist/index.js" }
#    - "files": ["dist"]
#    - "prepublishOnly": "npm run build"

# 2. Git
git init && git add -A
git commit -m "feat: initial release — MCP server for {App} via AppleScript"

# 3. GitHub
gh repo create zyx1121/apple-{app}-mcp --public --source=. --push
# Or if repo exists: git remote add origin ... && git push -u origin main

# 4. npm
npm publish --access public

# 5. Install command for users:
# claude mcp add apple-{app} -- npx @zyx1121/apple-{app}-mcp
```

## Updating an Existing MCP

```bash
# 1. Edit source
# 2. Build
npm run build
# 3. Bump version
npm version patch  # or minor/major
# 4. Commit + push + publish
git add -A && git commit -m "fix: description" && git push
npm publish --access public
```

## README Template

````markdown
# @zyx1121/apple-{app}-mcp

MCP server for {App} — {one-line description} via Claude Code.

## Install

```bash
claude mcp add apple-{app} -- npx @zyx1121/apple-{app}-mcp
```

## Prerequisites

- macOS with {App} installed
- Node.js >= 18
- First run will prompt for Automation permission (System Settings > Privacy & Security > Automation)

## Tools

| Tool | Description |
|------|-------------|
| `tool_name` | Description |

## Limitations

- macOS only (uses AppleScript via `osascript`)
- {App} must be running

## License

MIT
````
