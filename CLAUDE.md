# Kitty 终端配置

## 项目概述

这是 kitty 终端模拟器的配置目录，从 wezterm 迁移而来。原 wezterm 配置位于 `/home/miter/.config/wezterm/`。

## 目录结构

```
~/.config/kitty/
├── kitty.conf        # 主配置文件
├── everforest.conf   # 主题配色文件
└── memory/           # Claude 记忆目录
    ├── MEMORY.md
    └── kitty-migration.md
```

## 迁移背景

用户因 wezterm 存在无法解决的 bug 而切换到 kitty。需要保持原有的操作习惯和键绑定风格。

## 配置特点

- **主题**: Everforest Dark
- **字体**: Monaspace Argon/Xenon + 中文回退
- **键绑定风格**: Vim 风格 (h/j/k/l 导航)，Leader 模式
- **Shell**: bash

## Wezterm → Kitty 功能对照

| Wezterm | Kitty | 备注 |
|---------|-------|------|
| unix_domains | 需配合 tmux | kitty 无内置多路复用 |
| leader + mode | map 过滤器 | kitty 无模式系统 |
| workspace | tmux session | 需 tmux 替代 |
| pane split | layout splits | kitty 原生支持 |

## 迁移原则

1. 保持用户操作习惯一致
2. 逐步迁移，每个模块确认后再进行下一步
3. kitty 不支持的功能需提出替代方案供用户选择