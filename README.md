# osu4k-player

一个在 Windows 上**离线运行**的 osu!mania 4K 谱面播放器（Rust + egui +
[rosu-renderer](https://github.com/Glubus/rosu-renderer) 分支），可直接读取
`.osz` 谱面包并游玩：谱面库、难度选择、ScoreV1 判定、长条、暂停/继续、
评分结算、皮肤导入。

> Unofficial project, not affiliated with ppy. **No beatmaps, music or skins are
> included** — supply your own legally obtained `.osz` files. osu! is a
> trademark of ppy Pty Ltd.

## 功能

- **单文件离线程序**：`.osz` 运行时通过文件对话框或拖放加载；无需安装 Rust、
  无需 VC++ 运行库（静态 CRT），不需要网络。
- **谱面库**：导入多个 `.osz`，列出谱面包与每种难度（键数 / OD / 音符数 /
  星级），双击难度直接开始。
- **4K 游玩**：`D F J K`（5K–8K 依次为 `D F 空格 J K`、`S D F J K L`、
  `S D F 空格 J K L`、`A S D F J K L ;`）。
- **判定**：ScoreV1 口径，`300 ±(64−3·OD)` / `100 ±(97−3·OD)` /
  `50 ±(127−3·OD)` ms，OD 钳制 0–10；长条头判一次、按住到尾 300、提前松开按
  尾偏差 100/50/Miss；连击、准确率、1,000,000 归一化分数、评级。
- **暂停**：`Esc` 暂停/继续，音频停止、画面冻结、半透明覆盖层；从原位置恢复。
- **时钟**：以 rodio 输出的**采样计数**为权威时钟（rodio 0.17 没有
  `Sink::get_pos()`），画面与判定共用同一时间值。
- **设置**：下落时间、偏移、音量、自定义 OD、打击音效、自动演示、
  过快/过慢提示条、皮肤。
- **星级**：使用 [rosu-pp](https://github.com/MaxOhn/rosu-pp) 3.1.0 计算。
- **皮肤**：支持 osu!mania 皮肤的 `mania-note*`（含 `H/L/T`）、
  `mania-hit*`、`lightingN/L-<列>`、`mania-stage-hint` 与 `[Mania]` 列颜色；
  缺图自动回退内置样式（见下文）。
- **内嵌中文字体**：Noto Sans SC 子集（SIL OFL 1.1），不读取系统字体。

## 环境要求

| 项目 | 要求 |
|---|---|
| 系统 | Windows 10 x64 |
| 显卡 | 支持 OpenGL 2.0+（普通显卡驱动即可） |
| 构建 | Rust 1.88（MSVC 工具链） |
| 运行库 | 静态 CRT，无需 VC++ Redistributable |

## 构建

```bash
# 首次需要联网拉取依赖（Cargo.lock 已锁定版本）
cargo build --release
# 依赖就绪后可以完全离线构建
cargo build --release --offline
```

产物：`target/release/osu4k-player.exe`（单文件，字体与依赖均已编入）。

- `third_party/rosu-renderer` 是 rosu-renderer 1.2.3 的本地分支（MIT），
  通过 `Cargo.toml` 的 `[patch.crates-io]` 使用；改动说明见
  [third_party/rosu-renderer/PATCHES.md](third_party/rosu-renderer/PATCHES.md)。
- `.cargo/config.toml` 只保留静态 CRT 的 `rustflags`；不使用任何第三方镜像。

## 运行

1. 双击 `osu4k-player.exe`。
2. 「导入 .osz」或把 `.osz` 拖进窗口。
3. 左侧选谱面包，右侧选难度，「开始游玩」或双击难度。
4. `Esc` 暂停/继续；结算界面可重试或返回选曲。

设置在主界面底部（默认展开）。谱面解压缓存在 `%TEMP%\osu4k-player\`，
配置写在 exe 同目录的 `osu4k_player.ini`（目录不可写时写到 `%APPDATA%`）。

## 皮肤

「设置 → 导入皮肤文件夹」选择**解压后**的皮肤目录（含 `skin.ini`；`.osk`
请先解压）。已支持：

- `mania-note{1,2,3,4,5,S}` 及同名 `H`（头）/`L`（身）/`T`（尾），
  4K 按 osu! 的镜像规则 `1,2,2,1` 取图，优先 `@2x`；
- `mania-hit{300,300g,100,50,0}` 打击特效；
- `lightingN-<列>` / `lightingL-<列>` 按键光效；
- `mania-stage-hint` 判定线；
- `skin.ini` 的 `Colour1..N` 列颜色与 `ColourColumnLine`。

未提供的元素回退到内置样式。暂不支持：`mania-key*` 键帽、
`mania-stage-left/right/bottom`、combo/score 数字贴图、动画帧。

## 自检

```bash
cargo test
osu4k-player.exe --selftest "某谱面.osz"                 # 解压/解析/判定/音频
osu4k-player.exe --selftest --skin "皮肤目录"            # 皮肤图片加载情况
```

## 技术说明

- rosu-renderer 1.2.3 没有“外部时钟”入口，且音符映射会偏移一个
  scroll_time；本项目 fork 后新增 `render_at_time(ui, time_ms, pos, pressed,
  holding)` 并修正映射，详见 PATCHES。
- rodio 0.17.3 没有 `Sink::get_pos()`；在解码源内统计被设备取走的采样数
  作为时钟（暂停时自然冻结），并用小幅锁相平滑。
- `.osz` 用 `zip` crate 自行解压，再把解压后的 `.osu` 路径交给 `rosu-map`。
- 完整中文说明（设置、判定细节、离线构建、故障排查等）：
  [README.txt](README.txt)。

## 已知限制

- 仅支持 osu!mania（4K 为主，5–8K 基本兼容）；不支持 osu!/taiko/catch。
- 不做血量/失败判定；不支持 mods、replay、成绩上传。
- 没有音频 seek（从音频开头播放）。
- 皮肤为子集支持（见上）；不支持 `.osk` 直接导入。
- 渲染后端为 OpenGL；无显卡驱动的虚拟机无法启动（会弹出错误提示）。

## 许可

- 本项目代码：[MIT](LICENSE)
- 第三方组件：[THIRD_PARTY.md](THIRD_PARTY.md)
- 内嵌字体子集：SIL OFL 1.1（[assets/OFL-NotoSansSC.txt](assets/OFL-NotoSansSC.txt)），
  由 `tools/make_font_subset.py` 从 Noto Sans SC 生成
- vendored `rosu-renderer`：MIT（[LICENSE](third_party/rosu-renderer/LICENSE)）
