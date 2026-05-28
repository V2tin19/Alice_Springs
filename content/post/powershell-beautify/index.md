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

左下角有一个"打开 JSON 文件"的按钮，点它直接编辑 `settings.json`。在里面调这几个参数：

```jsonc
// 字体
"fontFace": "Cascadia Code PL",   // 等宽字体，支持 powerline 符号
"fontSize": 11,

// 透明度与磨砂
"useAcrylic": true,     // 开启磨砂质感
"acrylicOpacity": 0.6,  // 不透明度，0 全透 1 全实
"background": "#012456", // 深蓝底配合磨砂效果更好
```

`useAcrylic` 开了之后背景会有 Windows 那个毛玻璃效果，配合调整 `acrylicOpacity` 到你眼睛舒服的程度。

## 2. Fastfetch（开机见面礼）

在 `%USERPROFILE%\.config\` 目录下建一个 `fastfetch` 文件夹，里面放两个东西：

```text
.config/fastfetch/
├── ascii.txt          # 自定义字符画
└── config.jsonc       # 显示的硬件参数配置
```

**ascii.txt** - 开机在系统信息上面那张 ASCII 图，我放了一只像素小猫。

**config.jsonc** - 控制显示哪些模块：主机名、系统版本、内核、Uptime、CPU、GPU、内存、磁盘、Shell 版本、终端。不想看的东西注释掉就行。

装好 Fastfetch 之后终端一开自动跑，比看空荡荡的 prompt 顺眼多了。

## 3. Profile.ps1（默认行为）

在 `%USERPROFILE%\Documents\WindowsPowerShell\` 下面有一个 `profile.ps1` 文件，这个是 PowerShell 启动时自动加载的脚本。我在这里面设了几样东西：

```powershell
# 设置默认启动目录
Set-Location D:\Code

# 别名
Set-Alias g git
Set-Alias ll Get-ChildItem

# 自定义 Prompt
function prompt {
    "PS $($executionContext.SessionState.Path.CurrentLocation) > "
}
```

每次打开 PowerShell 自动切到工作目录，短命令顺手，prompt 也改成了自己习惯的格式。

---

三个地方改完，关掉重开一个 PowerShell，磨砂背景亮起来，Fastfetch 打出小猫 ASCII，路径自动切好，舒服了。
