# skills

Personal AI skills for Claude Code and other AI assistants.

## Install

```bash
bunx skills add zyx1121/skills
```

## Skills

### `daily-briefing`

整合 Apple Calendar、Reminders、Mail 的每日晨間簡報。

**觸發方式：** 說「今天有什麼事」、「日程簡報」、「daily briefing」

**需要的 MCP servers：**
- [`@zyx1121/apple-calendar-mcp`](https://github.com/zyx1121/apple-calendar-mcp)
- [`@zyx1121/apple-reminders-mcp`](https://github.com/zyx1121/apple-reminders-mcp)
- [`@zyx1121/apple-mail-mcp`](https://github.com/zyx1121/apple-mail-mcp)

**功能：**
- 今日行事曆事件
- 未完成提醒事項
- 未讀郵件分析（讀取內容後分類為 🔴 需要處理 / 🟡 值得注意 / ⚪ 可略過）
- 已讀垃圾信建議刪除

---

### `regression`

定期執行自訂任務的 stateful 排程 runner，支援報告生成與歷史查詢。

**觸發方式：** 說「regression」、「run regression」、「排程任務」

**功能：**
- 首次執行：引導建立 `~/.config/regression/config.md` 任務設定檔
- 後續執行：依序執行所有任務，儲存報告至 `~/.config/regression/reports/`
- 查詢報告：「regression 報告」、「上次結果」、「兩週比較」

## MCP Servers

搭配使用的 Apple 系列 MCP servers（均需 macOS）：

| Package | 說明 |
|---------|------|
| [`@zyx1121/apple-calendar-mcp`](https://github.com/zyx1121/apple-calendar-mcp) | Apple Calendar — 讀取、建立、更新、刪除行事曆事件 |
| [`@zyx1121/apple-reminders-mcp`](https://github.com/zyx1121/apple-reminders-mcp) | Apple Reminders — 管理提醒事項 |
| [`@zyx1121/apple-mail-mcp`](https://github.com/zyx1121/apple-mail-mcp) | Apple Mail — 讀取、搜尋、刪除信件 |
| [`@zyx1121/apple-music-mcp`](https://github.com/zyx1121/apple-music-mcp) | Apple Music — 播放控制 |
| [`@zyx1121/apple-contacts-mcp`](https://github.com/zyx1121/apple-contacts-mcp) | Apple Contacts — 聯絡人管理 |

安裝範例：

```bash
claude mcp add apple-mail -s user -- npx @zyx1121/apple-mail-mcp
claude mcp add apple-calendar -s user -- npx @zyx1121/apple-calendar-mcp
claude mcp add apple-reminders -s user -- npx @zyx1121/apple-reminders-mcp
claude mcp add apple-music -s user -- npx @zyx1121/apple-music-mcp
claude mcp add apple-contacts -s user -- npx @zyx1121/apple-contacts-mcp
```
