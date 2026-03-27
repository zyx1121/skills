---
name: daily-briefing
description: Use when user says "今天有什麼事", "日程簡報", "daily briefing", "今天行程", "今天待辦", "今天的信", "早安簡報", or wants a morning summary of their day.
---

# Daily Briefing

整合 Apple Calendar、Reminders、Mail，輸出今日摘要。

## Flow

1. 用 `calendar_list_events` 取今天的行事曆事件
2. 用 `reminders_get_lists` + `reminders_list` 取所有清單的未完成提醒
3. 用 `mail_count_unread` 取各帳號未讀數；用 `mail_list_messages` 取今天收到的信（最新 10 封）
4. 整合輸出

## Output Format

```
# 📅 今日簡報 — YYYY-MM-DD（週X）

## 行事曆
- HH:MM – HH:MM　事件名稱（行事曆名）
- 全天　　　　　　事件名稱
（無行程時：今天沒有行程）

## 待辦提醒
- [ ] 提醒名稱（截止：MM/DD）
- [ ] 提醒名稱
（無待辦時：沒有未完成的提醒）

## 未讀郵件
**總計 N 封未讀**
- 寄件人 — 主旨（時間）
- 寄件人 — 主旨（時間）
（列出今天收到的前 5 封，再補總未讀數）
```

## Rules

- 行事曆事件按開始時間排序；全天活動列在最前面
- 提醒只顯示未完成（`show_completed: false`），有截止日期的優先排列
- 郵件列今天收到的前 5 封；若今天無新信，改列最新 3 封未讀
- 若任何 MCP 工具失敗，跳過該區塊並標注「（無法取得）」，繼續輸出其他區塊
- 輸出完後，問：「有需要幫你處理什麼嗎？」
