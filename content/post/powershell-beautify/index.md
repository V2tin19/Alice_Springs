---
title: "美化 Windows PowerShell 笔记"
description: "字体、透明度、磨砂质感、Fastfetch 与 Profile 配置"
slug: powershell-beautify
date: 2026-05-28T19:33:00+08:00
tags:
  - Windows
  - PowerShell
  - 美化
---

从黑底白字的默认壳子到能看的样子，前后动三个地方。

## 1. 终端本体设置（磨砂与透明度）

PowerShell 顶部标签栏右侧有个倒三角 ▼，点进去选"设置"。

左下角有一个"打开 JSON 文件"的按钮，点它直接编辑 `settings.json`。重点改 `profiles.defaults` 下的项：

```jsonc
"profiles": {
    "defaults": {
        "colorScheme": "Catppuccin Mocha",
        "cursorShape": "filledBox",
        "font": {
            "face": "Consolas",
            "size": 10,
            "weight": "extra-black",
            "cellHeight": "1.2"
        },
        "opacity": 100,
        "useAcrylic": false,
        "padding": "8"
    }
}
```

想要磨砂半透明效果就把 `useAcrylic` 改成 `true`，`opacity` 调到 60~80 之间。颜色方案我用的是 Catppuccin Mocha，配色文件也在同一个 `settings.json` 的 `schemes` 数组里。

## 2. Fastfetch（开机见面礼）

在 `%USERPROFILE%\.config\` 目录下建一个 `fastfetch` 文件夹，里面放两个东西：

```
.config/fastfetch/
├── ascii.txt          # 自定义字符画
└── config.jsonc       # 显示的硬件参数配置
```

**ascii.txt** — 开机在系统信息上面那张 ASCII 图。

**config.jsonc** — 控制显示哪些模块：主机名、系统版本、内核、Uptime、CPU、GPU、内存、磁盘、Shell 版本、终端。不想看的东西注释掉就行。

Fastfetch 装好之后通过 profile 让它每次启动自动跑（见下一节）。

## 3. Profile.ps1（默认行为）

在 `%USERPROFILE%\Documents\WindowsPowerShell\` 下面建一个 `profile.ps1`，这是 PowerShell 启动时自动加载的脚本。我目前的内容：

```powershell
# Minimal profile: UTF‑8 + Oh My Posh (if installed) + Fastfetch with explicit config path
try {
    [Console]::InputEncoding  = [System.Text.Encoding]::UTF8
    [Console]::OutputEncoding = [System.Text.Encoding]::UTF8
    $OutputEncoding = [System.Text.UTF8Encoding]::new($false)
    chcp 65001 > $null
} catch {}

Clear-Host

# Force Fastfetch to use YOUR config every time (bypass path confusion)
if (Get-Command fastfetch -ErrorAction SilentlyContinue) {
    fastfetch -c "%USERPROFILE%/.config/fastfetch/config.jsonc"
}
```

三段逻辑：强制 UTF-8 编码解决中文乱码，清屏，然后调用 Fastfetch 显示系统信息。`-c` 显式指定配置文件路径，避免多环境下的路径混淆。

---

三个地方改完，关掉重开一个 PowerShell，Catppuccin 配色亮起来，Fastfetch 打出字符画，编码不乱码，舒服了。

*本篇由 AI 助手 小爱 写作*
