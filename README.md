# skills

Personal AI skills for Claude Code and other AI assistants.

## Install

```bash
bunx skills add zyx1121/skills
```

## Skills

### `daily-briefing`

Morning briefing that integrates Apple Calendar, Reminders, and Mail.

**Triggers:** "今天有什麼事", "daily briefing", "今天行程", "今天的信", or any morning summary request.

**Required MCP servers:**
- [`@zyx1121/apple-calendar-mcp`](https://github.com/zyx1121/apple-calendar-mcp)
- [`@zyx1121/apple-reminders-mcp`](https://github.com/zyx1121/apple-reminders-mcp)
- [`@zyx1121/apple-mail-mcp`](https://github.com/zyx1121/apple-mail-mcp)

**What it does:**
- Today's calendar events
- Pending reminders
- Unread email analysis — reads each message and classifies as 🔴 action needed / 🟡 worth noting / ⚪ skip
- Suggests deleting low-value read emails

---

### `regression`

Stateful recurring task runner with report generation and history queries.

**Triggers:** "regression", "run regression", "排程任務", or any recurring automation request.

**What it does:**
- First run: guides you through creating `~/.config/regression/config.md` with your tasks
- Subsequent runs: executes all tasks in order, saves reports to `~/.config/regression/reports/`
- Report queries: "regression report", "last run results", "compare last two weeks"

### `apple-mcp-dev`

Playbook for building, debugging, and publishing Apple MCP servers.

**Triggers:** "new MCP", "apple mcp", "AppleScript MCP", "build mcp server", or when working in any `apple-*-mcp` repo.

**What it does:**
- Complete project structure and boilerplate
- AppleScript/JXA gotchas (reserved keywords, newlines, sandbox paths, data separators)
- When to use AppleScript vs JXA
- Observability pattern (preview tool for self-verification)
- Development workflow and publishing checklist

---

## MCP Servers

Apple MCP servers that pair well with these skills (macOS only):

| Package | Description |
|---------|-------------|
| [`@zyx1121/apple-calendar-mcp`](https://github.com/zyx1121/apple-calendar-mcp) | Apple Calendar — read, create, update, delete events |
| [`@zyx1121/apple-reminders-mcp`](https://github.com/zyx1121/apple-reminders-mcp) | Apple Reminders — manage reminder lists and items |
| [`@zyx1121/apple-mail-mcp`](https://github.com/zyx1121/apple-mail-mcp) | Apple Mail — read, search, send, delete messages |
| [`@zyx1121/apple-music-mcp`](https://github.com/zyx1121/apple-music-mcp) | Apple Music — playback control |
| [`@zyx1121/apple-contacts-mcp`](https://github.com/zyx1121/apple-contacts-mcp) | Apple Contacts — manage contacts |
| [`@zyx1121/apple-powerpoint-mcp`](https://github.com/zyx1121/apple-powerpoint-mcp) | Microsoft PowerPoint — create, edit, export presentations |

Install all as user-scoped MCP servers:

```bash
claude mcp add apple-calendar -s user -- npx @zyx1121/apple-calendar-mcp
claude mcp add apple-reminders -s user -- npx @zyx1121/apple-reminders-mcp
claude mcp add apple-mail -s user -- npx @zyx1121/apple-mail-mcp
claude mcp add apple-music -s user -- npx @zyx1121/apple-music-mcp
claude mcp add apple-contacts -s user -- npx @zyx1121/apple-contacts-mcp
claude mcp add apple-powerpoint -s user -- npx @zyx1121/apple-powerpoint-mcp
```
