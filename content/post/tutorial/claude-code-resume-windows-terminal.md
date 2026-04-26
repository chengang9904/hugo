---
title: "Windows 下 Claude Code /resume 显示不全的处理方法"
date: 2026-04-26T00:00:00+08:00
draft: false
tags:
  - Claude Code
  - Windows Terminal
  - PowerShell
  - Hugo
  - Troubleshooting
categories:
  - 教程
---

如果你在 Windows 上使用 Claude Code，可能会遇到这样一个现象：

- 正常开发对话时，内容是一点一点输出的，显示完全正常。
- 但使用 `/resume` 或 `claude -c` 恢复上一次会话时，内容会一次性灌入终端。
- 这时候终端里只能看到后半段，前面的内容像被“吃掉”了一样。

例如你原本的对话内容可以抽象成 `abcd`：

- 正常实时输出时，`a b c d` 都能看到。
- 但恢复会话时，可能只看到 `Claude Code UI 头部 + bcd`，最前面的 `a` 不显示。

很多人第一反应会去调大 Windows Terminal 的 scrollback 行数，但这个问题通常**不是单纯的显示行数不够**，而更像是 **Claude Code 在 Windows 下恢复长会话时的 TUI 渲染或 scrollback 兼容问题**。

## 适用环境

本文适用于类似下面的环境：

- Windows Terminal 最新版
- PowerShell 7 最新版
- 已经手动调整过终端最大显示行数
- 问题主要出现在 Claude Code 的 `/resume` 或 `claude -c`

## 现象为什么会发生

从现象上看，这不是“Claude Code 没有恢复完整上下文”，而更像是：

1. **实时流式输出** 是一行一行刷新的，终端更容易正常显示。
2. **恢复旧会话** 时，Claude Code 会一次性重建较长的历史内容。
3. 在 Windows Terminal + PowerShell 的组合下，这段恢复出来的内容不一定能完整落进你当前可见的 scrollback 区域。
4. 结果就是：你能看到界面头部和后半段内容，但更早的部分看不到。

所以，问题重点往往不在“终端最多能存多少行”，而在“恢复时这段内容是怎么被渲染和挂接到滚动缓冲区里的”。

## 最有效的解决办法

### 1. 把 Claude Code 切到 fullscreen renderer

先编辑 Claude Code 的用户配置文件：

```text
C:\Users\Win\.claude\settings.json
```

加入下面这段：

```json
{
  "tui": "fullscreen"
}
```

这是最关键的一项。

`fullscreen` 模式本身就是为了更稳定地处理长内容显示、滚动和重绘问题。对于 `/resume` 或 `claude -c` 这种一次性恢复长历史的场景，通常比默认模式更稳。

### 2. 恢复后不要只靠终端滚轮回看历史

即便切了 `fullscreen`，也不建议只靠鼠标滚轮去赌终端有没有把所有历史都保住。

更稳的做法是：

1. 恢复会话后按 `Ctrl+O` 打开 transcript viewer。
2. 在 transcript viewer 里按 `[`，把完整会话写回到终端原生 scrollback。
3. 如果还想更稳，按 `v`，把会话导出到临时文件，然后用外部编辑器查看。

这个方式通常比直接在主界面里往上翻可靠得多。

### 3. 会话特别大时，优先用 summary 恢复

如果是很长、很老的会话，Claude Code 往往会提供从 summary 恢复的方式。

如果你的目标是继续工作，而不是逐字重新阅读全部历史，那么 summary 模式通常更稳定，也更不容易触发这类显示问题。

### 4. 局部没刷出来时按 `Ctrl+L`

如果你怀疑只是界面没完整重绘，可以按：

```text
Ctrl+L
```

这个操作会强制重绘界面，有时候能把“只显示一部分”的情况刷回来。

## 那另外两个可选参数是做什么的

除了 `tui: "fullscreen"`，还可以加两个辅助项：

```json
{
  "tui": "fullscreen",
  "viewMode": "verbose",
  "prefersReducedMotion": true
}
```

下面分别解释。

### `viewMode: "verbose"`

作用是让 Claude Code 的界面更偏向“完整展示”，少一些折叠和精简。

它的价值在于：

- 恢复会话时，显示出来的信息更充分。
- 更容易判断是“内容真的没恢复”，还是“其实恢复了，只是 UI 没给你完整展开”。

它**不是根治项**，更像一个辅助观察和减少摘要展示的选项。

### `prefersReducedMotion: true`

这个配置会尽量减少动画、过渡效果和一些动态重绘。

它的价值在于：

- 在 Windows Terminal 上，频繁动画和重绘有时会放大显示异常。
- 开启后，界面更静态，某些闪烁、局部没刷出来的问题可能会减轻。

它同样**不是核心修复项**，只是一个辅助稳定项。

## 推荐配置

如果你只想做最小改动，推荐先用：

```json
{
  "tui": "fullscreen"
}
```

如果你希望顺便把显示稳定性再往上拉一点，可以改成：

```json
{
  "tui": "fullscreen",
  "viewMode": "verbose",
  "prefersReducedMotion": true
}
```

## 建议的排查顺序

可以按下面顺序排查：

### 方案一：先做最小改动

1. 升级 Claude Code 到最新版本。
2. 在 `settings.json` 里只加：

```json
{
  "tui": "fullscreen"
}
```

3. 重新打开 Claude Code。
4. 再试一次 `/resume` 或 `claude -c`。

### 方案二：如果仍然偶发显示不全

1. 继续保留 `fullscreen`。
2. 增加：

```json
{
  "viewMode": "verbose",
  "prefersReducedMotion": true
}
```

3. 恢复会话后立即按 `Ctrl+O`。
4. 使用 `[` 或 `v` 获取完整 transcript。

## 总结

如果你在 Windows Terminal 里已经把最大显示行数调大，但 Claude Code 的 `/resume` 或 `claude -c` 仍然显示不全，那么大概率**不是终端行数本身不够**，而是 **恢复长会话时的渲染和 scrollback 兼容问题**。

最值得先试的方案只有一句话：

```json
{
  "tui": "fullscreen"
}
```

如果还想更稳，就再加上：

```json
{
  "viewMode": "verbose",
  "prefersReducedMotion": true
}
```

另外，恢复之后尽量用 `Ctrl+O` 打开 transcript viewer，再通过 `[` 或 `v` 查看完整内容，不要完全依赖终端原生回滚。
