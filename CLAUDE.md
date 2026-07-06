# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

考研加油站 — 单文件 HTML 页面（`index.html`），为考研学生（2026年12月19日）设计的学习管理工具。纯前端，无构建工具，无框架，部署到 GitHub Pages。

## 常用命令

```bash
# 本地预览
open index.html

# 部署到 GitHub Pages
git add index.html && git commit -m "描述" && git push
# GitHub Pages 部署经常卡住，推送后需手动触发：
# Settings → Pages → Branch 切 None→Save→切回 main→Save

# 查看提交历史
git log --oneline -5
```

## 架构

单文件 ~3000 行，分三块：CSS（~700行） → HTML（~300行） → JS（~2000行）。

### 数据层

`localStorage` 键名 `kaoyan_study_data`，核心数据结构：

```
data = {
  totalMinutes, todayDate, todayMinutes, todayPomodoros,
  streak, lastStudyDate, weeklyCheckins: {date: bool},
  goals: [{id, name, type, target, unit, emoji, current, completedAt, completedCount}],
  logs: [{id, subject, subjectEmoji, minutes, date, time, note, auto}],
  achievements: {achId: timestamp},
  // 隐藏成就追踪字段: earlyBird, nightOwl, midnightOil, marathonDone,
  //   weekendWarrior, luckyNumber, doubleDay, silenceLover, mascotClicks,
  //   perfectPomodoros, morningStreak, goalsCompleted, timerModesUsed, etc.
}
```

### 面板系统

5 个面板通过 `.panel.active` 切换，导航按钮用 `data-panel` 属性关联：

- `panel-dashboard` — 倒计时、语录、日历、成就、数据管理
- `panel-pomodoro` — 番茄钟（4种倒计时 + 正向计时）
- `panel-goals` — 目标 CRUD（每日/每周/每月/单次）
- `panel-log` — 学习记录
- `panel-achievements` — 完整成就列表

### 日历模式

日历在仪表盘中，3 种模式通过 `switchCalMode()` 切换：
- `month` — 月视图，格子显示日期+学习时长
- `heat` — 热力图，6 个月色块（灰→蓝→绿→黄→红→勃艮第红），每格 24x24px
- `chart` — SVG 饼图（科目分布）+ 柱状图（本月每日+上周均值虚线）

### 番茄钟模式

预设按钮在 `.timer-presets`，`data-countup="true"` 标记正向计时。正向模式往上累计，手动停止记录；倒计时模式自动切换工作/休息。

### 成就系统

`ACHIEVEMENTS` 数组，每个成就含 `trophy`（bronze/silver/gold/platinum）、可选的 `hidden: true`。`check()` 函数检测达成条件。白金奖杯需全部 41 个成就解锁。

### 关键函数

- `refreshDashboard()` — 刷新仪表盘所有数据，包括 streak 重算
- `recalcStreak()` — 从 `data.logs` 直接计算连续打卡天数
- `recordStudySession(minutes)` — 记录一次学习（倒计时结束或正向计时停止）
- `renderGoals()` — 目标面板渲染，分「进行中」和「已完成」
- `checkDateChange()` — 日期变更时重置每日/每周/每月目标，重算 streak

## 注意事项

- 所有数据在浏览器 localStorage，换设备/浏览器会丢失，仪表盘底部有导出/导入按钮
- 每日/每周/每月目标会在日期变更、周一、1号自动重置 current 为 0，单次目标不重置
- 空间热键：空格键启动/暂停番茄钟
- GitHub Pages 部署不稳定，经常需要手动到 Settings → Pages 重触发
