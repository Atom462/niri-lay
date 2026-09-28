# niri-lay

**保存 niri 当前的分屏格局，一键还原。**（save & restore niri workspace layout presets）

Windows 有固定的分屏布局，niri 只有自动平铺——`lay` 补上"把手动摆好的格局存成预设、随时拉回来"这一块：

```bash
lay save 学习     # 把当前工作区（几列、每列几窗、各列多宽）存成预设
lay               # 列出所有预设
lay 学习          # 一键：自动切到空工作区 → 开终端 → 摆成预设的样子
lay add 学习       # 现场补齐：眼前终端当第一格，补开剩余终端并整层排布
lay switch 学习 写代码  # 当前像 A 就切 B，像 B 就切 A（原地来回跳）
lay edit 学习      # 用 $EDITOR 直接改预设（给名字跳到那行）
lay rm 学习       # 删除预设
```

还原后只开终端、不执行命令——内容自己输入。

---

## 常用场景

**存一个固定工位**：手摆好窗口 → `lay save 写代码`。以后任何时候 `lay 写代码` 就还原。

**在已有工作区铺开**：你正在某个终端里，敲 `lay add 写代码`——**你敲命令的这个终端会成为第一格**，收工时焦点精确回到你手上这个终端。

**两个格局来回跳**：`lay switch 学习 写代码`。它比对当前工作区与两份预设的结构（每列窗数 + 宽度），像 A 就切到 B，像 B 就切到 A；两份都不像时会报错，不会乱动你的窗口。

**还原成原来的程序**：加 `--apps`，按存档里的程序清单开（终端以外的程序也开），某个程序不在 PATH 就回退成终端并提示。

---

## 标志

```bash
lay --json        # 输出 presets.json 原文，方便脚本处理
lay --apps 学习    # 还原时开存档里的程序，不只开终端
```

---

## 依赖

- [niri](https://github.com/niri-wm/niri)（用到 `niri msg` IPC 与 action：`spawn` / `consume-or-expel-window-left` / `set-column-width` / `focus-window`）
- Python ≥ 3.10（仅标准库）
- 一个终端（默认 `kitty`，用 `LAY_SPAWN=foot lay <名字>` 换）
- `lay edit` 需要 `$EDITOR`（默认 `nvim`）

---

## 安装

```bash
git clone <this repo> ~/.config/niri/niri-lay   # 与 config.kdl 同级，作 niri 插件
ln -s ~/.config/niri/niri-lay/lay ~/.local/bin/lay   # 或 cp 到 PATH 里
```

---

## 预设存哪

`~/.config/niri-lay/presets.json` —— 换机器拷这一个文件即可带走全部预设。

| 字段 | 用途 |
|---|---|
| `count` / `width` | 每列窗数、占输出宽度的比例 |
| `width_px` | 保存时的绝对像素宽——**同一显示器还原时用它，更精准**；换了分辨率才回退到比例 |
| `heights` | 列内每窗的高度比例 |
| `apps` | 每格跑的程序，`--apps` 用 |

---

## 原理

1. **保存**：`niri msg -j windows` 读每个窗口的列位置（`pos_in_scrolling_layout`）与列宽，除以输出逻辑宽度存成比例，同时留一份绝对像素。
2. **还原**：在空工作区 `spawn` 终端 → 同组窗口用 `consume-or-expel-window-left` 合并进同一列 → 逐列 `set-column-width` 摆宽（输出宽度与保存时一致就用像素，否则用比例）。
3. **切换**：比对当前工作区与两份预设的结构，确定目标后走补齐流程。

---

## 已知边界（v1）

- 当前工作区有窗口时**自动滑到下方新空工作区**再还原（找不到空的才报错）
- 还原对象是"格子"，不记忆每个格子里跑的是什么（`--apps` 只在还原时按清单开一次）
- `switch` 需要当前格局与两份预设之一吻合，否则报错不动手
- 多显示器、浮动窗口暂不支持

---

## License

MIT
