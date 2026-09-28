# niri-lay

把 niri 的工作区布局存成预设，随时拉回来。

我用的桌面是 Arch Linux 上的 niri（配 DMS）。每次坐下来干活，都要把同样的几个终端重新
摆成同样的形状——同样的列、同样的宽度、同样的叠放。懒得每次重新规划布局，就让 AI 写了
这么个小脚本：先摆一次存下来，以后一键还原。

就这么多。它**不是**会话管理器——不会快照你的桌面、也不会重启你的程序，只管"形状"。

```bash
lay save study     # 把当前工作区的布局存成预设 "study"
lay                # 列出所有预设
lay study          # 切到空工作区，按预设重建
lay add study      # 在当前工作区补齐——你敲命令的这个终端是第一格
lay switch study code   # 当前像 study 就切成 code，像 code 就切成 study
lay edit study     # 用 $EDITOR 改预设 JSON
lay rm study       # 删除预设
```

还原只开终端，不在里面执行任何命令——内容自己输入。

## 依赖

- [niri](https://github.com/niri-wm/niri)：用到 `niri msg` IPC 与这些 action——
  `spawn`、`consume-or-expel-window-left`、`set-column-width`、`focus-column`、`focus-window`
- Python ≥ 3.10（只用标准库，无第三方依赖）
- 一个终端（默认 `kitty`，可用 `LAY_SPAWN=foot lay study` 换）
- `lay edit` 需要 `$EDITOR`（默认 `nvim`）

在 Arch Linux + niri 26.04 + kitty 上测过；只要 niri 版本提供上述 action，应该都能跑。

## 安装

```bash
git clone <本仓库> ~/.config/niri/niri-lay     # 与 config.kdl 同级，当作 niri 插件
ln -s ~/.config/niri/niri-lay/lay ~/.local/bin/lay
```

## 行为说明

- **`lay save <名字>`**：读当前焦点工作区每个窗口的列位置与尺寸；列宽同时存比例与绝对像素，
  另外存列内各窗高度比例和每格的 `app_id`。
- **`lay <名字>`**：自动滑到下方空工作区（niri 会新建），开终端、按预设并成列、设宽度。
  当前输出宽度与存档一致就用像素（更准），不一致回退百分比，换显示器也能看。
- **`lay add <名字>`**：不换工作区。现有窗口填第一列，缺的终端补开，收工时焦点回到你敲
  命令的那个终端。
- **`lay switch A B`**：比对当前布局与两份预设（列数 + 宽度）。像 A 就给 B，像 B 就给 A；
  两份都不像就报错，什么都不动。
- **`--apps`**：还原时按存档的程序清单开（程序不在 `PATH` 就回退成终端）。
- **`--json`**：原样输出 `presets.json`，方便脚本处理。

预设存在 `~/.config/niri-lay/presets.json`。就一个文件，换机拷走即可。

| 字段 | 含义 |
| --- | --- |
| `count`、`width` | 每列窗数、宽度占输出的比例 |
| `width_px` | 保存时的绝对像素宽——输出宽度一致时用它 |
| `heights` | 列内每窗的高度比例 |
| `apps` | 每格跑的程序，`--apps` 用 |

## 已知边界

- 只开终端，不记每个格子里**跑过什么**（`--apps` 只是按清单开一次程序）。
- 不支持浮动窗口与多显示器。
- 不支持标签页式/分组列，也没有窗口规则。
- 还原会开好几个终端，所以会先滑到空工作区，不打扰你当前的工作；`add` 与 `switch` 原地
  操作，装不下时会拒绝而不是硬来。
- `switch` 要求当前布局与两份预设之一吻合，没有模糊匹配。

## 免责声明

**本项目由 AI 按我的要求写成。** 我是学生，这是我发布的第一个工具：想法、需求和测试是
我的，代码主要是 AI 的——请把它当作学习项目，而不是成熟软件。

- 就是一个 Python 脚本（约 300 行，只用标准库）。**运行前请自己读一遍。**
- 它只通过 IPC 与 niri 通信：不改你的 `config.kdl`，也不删文件。
- 只在作者本人的环境（Arch Linux、niri 26.04、kitty）上测过，粗糙之处在所难免。
- 无任何担保，见 [LICENSE](LICENSE)，使用风险自负。

欢迎 issue 与 PR——指出哪里写错了、写得不好，对我真的有帮助，我是来学的。

英文版：[README.md](README.md)

## 许可

[MIT](LICENSE)
