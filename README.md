# niri-lay

**保存 niri 当前的分屏格局，一键还原。**（save & restore niri workspace layout presets）

Windows 有固定的分屏布局，niri 只有自动平铺——`lay` 补上"把手动摆好的格局存成预设、随时拉回来"这一块：

```bash
lay save 学习     # 把当前工作区（几列、每列几窗、各列多宽）存成预设
lay               # 列出所有预设
lay 学习          # 一键：自动切到空工作区 → 开终端 → 摆成预设的样子
lay add 学习       # 现场补齐：眼前终端当第一格，补开剩余终端并整层排布
lay rm 学习       # 删除预设
```

还原后只开终端、不执行命令——内容自己输入。

## 依赖

- [niri](https://github.com/niri-wm/niri)（用到 `niri msg` IPC 与 action：`spawn` / `consume-or-expel-window-left` / `set-column-width`）
- Python ≥ 3.10（仅标准库）
- 一个终端（默认 `kitty`，用 `LAY_SPAWN=foot lay <名字>` 换）

## 安装

```bash
git clone <this repo>
ln -s "$PWD/lay" ~/.local/bin/lay     # 或 cp 到 PATH 里
```

## 预设存哪

`~/.config/niri-lay/presets.json` —— 换机器拷这一个文件即可带走全部预设。

## 原理

1. **保存**：`niri msg -j windows` 读每个窗口的列位置（`pos_in_scrolling_layout`）与列宽，除以输出逻辑宽度存成比例。
2. **还原**：在空工作区 `spawn` 终端 → 同组窗口用 `consume-or-expel-window-left` 合并进同一列 → 逐列 `set-column-width` 按比例摆宽。

## 已知边界（v1）

- 当前工作区有窗口时**自动滑到下方新空工作区**再还原（找不到空的才报错）
- 还原对象是"格子"，不记忆每个格子里跑的是什么程序
- 多显示器、浮动窗口暂不支持

## License

MIT
