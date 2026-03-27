---
name: tmux-key-bindings
description: tmux 按键绑定的语法规则和调试方法
type: reference
---

## tmux 绑定语法关键规则

**核心发现**: `Shift+Alt+字母` 发送 `ESC + 大写字母`，tmux 必须用 `M-大写字母`

```bash
# Shift+Alt+H → ^[H → M-H
bind -n M-H select-pane -L

# Shift+Alt+P → ^[P → M-P
bind -n M-P split-window -h
```

**Why**: kitty 发送 `^[H` (ESC + 大写H)，tmux 的 `M-` 前缀表示 Alt，大写字母隐含 Shift。`S-M-h` 语法在 tmux 中不存在或行为不同。

**How to apply**: 所有 Shift+Alt+字母 绑定统一用 `M-大写字母`

## 调试流程

1. `cat -v` + 按键 → 查看原始序列
2. `tmux list-keys -T root` → 验证绑定是否存在
3. `tmux source-file` → 重载配置

## 方向键例外

方向键保持 `M-S-` 语法：
- `Shift+Alt+Left` → `M-S-Left`
- 原因：方向键发送的是 CSI 序列（如 `^[[1;4D`），tmux 有特殊处理