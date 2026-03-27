---
name: kitty-migration
description: 从 wezterm 迁移到 kitty 终端的项目背景
type: project
---

**目标**: 将终端模拟器从 wezterm 切换到 kitty，编写符合用户习惯的 kitty.conf 配置。

**背景**: 用户之前使用 wezterm，但存在一些无法解决的 bug，现决定切换到 kitty。

---

## 迁移进度

| 模块 | 状态 | 备注 |
|------|------|------|
| 字体配置 | ✅ 完成 | Monaspace Argon + Symbols Nerd Font |
| 主题 | ✅ 完成 | everforest.conf |
| 复制粘贴 | ✅ 完成 | 混合方案: Kitty 鼠标 + Tmux 键盘 |
| 滚动 | ✅ 完成 | 非 tmux 场景的基础滚动 |
| tmux 配置 | ✅ 完成 | 按键绑定已修复 |
| 测试验证 | ✅ 完成 | 快捷键已验证正常 |

**当前状态**: 迁移完成。

---

## 文件结构

```
~/.config/kitty/
├── kitty.conf          # 主配置
├── everforest.conf     # 主题配色
└── memory/             # 记忆目录

~/.config/tmux/
└── tmux.conf           # tmux 配置
```

---

## 已确定方案

### 复制粘贴与滚动 — 混合实现

**Kitty 层**:
- 鼠标选择即复制到剪贴板
- 快捷键: Ctrl+Shift+C/V 或 Shift+Alt+C/V
- 滚动: Ctrl+Shift+方向键/PageUp/PageDown

**Tmux 层**:
- Copy 模式: Shift+Alt+X 进入
- vim 风格键绑定
- Wayland 剪贴板集成 (wl-clipboard)

### Tab 和 Pane

由 tmux 实现，kitty 不配置。

---

## Tmux 快捷键速查

### Window (Tab)
| 快捷键 | 功能 |
|--------|------|
| `Shift+Alt+T` | 新建窗口 |
| `Alt+H` | 上一个窗口 |
| `Alt+L` | 下一个窗口 |

### Pane
| 快捷键 | 功能 |
|--------|------|
| `Shift+Alt+P` | 右侧分割 |
| `Shift+Alt+D` | 下方分割 |
| `Shift+Alt+H/J/K/L` | 切换 pane |
| `Shift+Alt+方向键` | 调整大小 |

### Session
| 快捷键 | 功能 |
|--------|------|
| `Shift+Alt+O` | 进入 session 模式 |
| `s` | 列出 sessions |
| `d` | detach |
| `n` | 新建 session（输入名称） |
| `Escape` | 退出模式 |

### Copy Mode
| 快捷键 | 功能 |
|--------|------|
| `Shift+Alt+X` | 进入 copy 模式 |
| `v` | 开始选择 |
| `V` | 选择行 |
| `Ctrl+V` | 块选择 |
| `y` | 复制并退出 |

### 状态栏与标题
| 配置项 | 值 |
|--------|-----|
| 状态栏位置 | 顶部 (`status-position top`) |
| 窗口标题 | session name (`set-titles-string "#{session_name}"`) |

---

## 重要修复记录

### tmux 按键绑定语法
**问题**: `S-M-h` 语法不匹配 Shift+Alt+H 发送的序列
**原因**: kitty 发送 `^[H` (ESC + 大写H)，tmux 需要用 `M-H` 而不是 `S-M-h`
**解决**: 所有 Shift+Alt+字母 绑定使用 `M-大写字母` 格式

```bash
# 错误
bind -n S-M-h select-pane -L

# 正确
bind -n M-H select-pane -L
```

### 调试终端按键的方法

**1. 检查终端发送的原始序列**
```bash
cat -v
# 然后按目标键，观察输出
# 例如 Shift+Alt+H 显示 ^[H
```

**2. kitty 键盘协议调试**
```bash
kitty +kitten show_key -m kitty  # kitty 扩展协议
kitty +kitten show_key -m normal # 传统模式
```

**3. 验证 tmux 绑定**
```bash
tmux list-keys -T root | grep "M-H"
```

### tmux 绑定语法对照表

| 按键 | 终端发送 | tmux 语法 |
|------|----------|-----------|
| Alt+h | `^[h` | `M-h` |
| Alt+H (Shift+Alt+h) | `^[H` | `M-H` |
| Alt+Left | `^[[1;3D` | `M-Left` |
| Shift+Alt+Left | `^[[1;4D` | `M-S-Left` |

**规则**:
- 字母键：`Shift+Alt+字母` → tmux 用 `M-大写字母`
- 方向键：`Shift+Alt+方向` → tmux 用 `M-S-方向`

### 配置兼容性

**tmux 终端设置需匹配 kitty**:
```bash
set -g default-terminal "xterm-kitty"
set -ag terminal-overrides ",xterm-kitty:RGB"
set -s extended-keys on
```

### 常见错误

| 错误 | 原因 | 解决 |
|------|------|------|
| `paste-command` 无效 | tmux 3.4 无此选项 | 用 `bind v run "wl-paste \| tmux load-buffer - ; tmux paste-buffer"` |
| 按键无响应 | 绑定语法错误 | 用 `cat -v` 确认序列，修正语法 |
| 绑定不加载 | tmux 未重载 | `tmux source-file ~/.config/tmux/tmux.conf` |

---

## 软硬件环境

```
CPU:  2 x Intel(R) Xeon(R) E5-2650 v2 (32) @ 3.40 GHz
GPU:  AMD Radeon RX 580 2048SP [Discrete]
OS:   Kubuntu 24.04.4 LTS (Noble Numbat) x86_64
内核: Linux 6.8.0-106-generic
Shell: bash 5.2.21
DE:   KDE Plasma 5.27.12
显示: KWin (Wayland)
终端: kitty 0.46.2
输入法: Fcitx5 5.1.7
tmux: 3.4
```

**注意事项**:
- Wayland 环境 → tmux 剪贴板使用 wl-clipboard
- Fcitx5 输入法 → Alt 键可能需要确认不冲突

---

## 待办事项

1. 用户测试所有快捷键
2. 验证 Wayland 剪贴板功能
3. 确认 Fcitx5 与 Alt 键无冲突
4. 根据测试结果调整配置