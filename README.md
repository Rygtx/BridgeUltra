# Bridge Ultra

[![Release](https://img.shields.io/github/v/release/GeekASMR/BridgeUltra)](https://github.com/GeekASMR/BridgeUltra/releases)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](#license)
[![Platform](https://img.shields.io/badge/platform-Windows%2010%20%7C%2011-lightgrey)](#requirements)

A modern, low-latency Windows bridge that lets 64-bit DAW hosts load
32-bit VST2 plugins. Designed as a maintained replacement for JBridge.

让 64 位宿主 DAW 能在 Windows 上加载 32 位 VST2 插件的现代低延迟桥接器，
作为 JBridge 的可维护替代方案。

---

## Highlights · 特性

- **Per-plugin sub-process isolation** — Each plugin instance runs in its
  own 32-bit `BridgeUltraHost32.exe`. A plugin crash never takes down the
  DAW; the proxy paints a "Restart" overlay and respawns the host process
  on demand.

  每个插件实例运行在独立的 32 位子进程里，插件崩溃不会拖垮 DAW，代理会
  画出"重启"覆盖层，按需重新拉起子进程。

- **Zero-copy audio path** — Shared memory ring + named events, target
  end-to-end overhead under 1 ms at 48 kHz / 256 sample blocks.

  共享内存环形缓冲 + 命名事件，48 kHz / 256 采样块下端到端开销目标 < 1 ms。

- **Cross-process embedded GUI** — The plugin's own editor renders inside
  the DAW's plugin panel via the JBridge-style 64-bit container + 32-bit
  child window pattern. Full Per-Monitor V2 DPI awareness.

  跨进程嵌入式 GUI：插件原生编辑器直接显示在 DAW 的插件面板内，使用
  JBridge 风格的 64 位容器 + 32 位子窗口模式，完整支持 Per-Monitor V2 DPI。

- **Bilingual native installer** — Win32 + Direct2D + DirectWrite UI.
  Bilingual zh/en, system theme aware (light / dark / high-contrast),
  fully keyboard-traversable.

  双语原生安装器：Win32 + Direct2D + DirectWrite，中英双语，自动跟随
  系统主题（浅色 / 深色 / 高对比度），全键盘可达。

- **Single-file setup** — One `BridgeUltra-Setup-<version>.exe` carries
  the GUI, the proxy DLL template, and the 32-bit sub-process host as a
  single LZMA-compressed payload. No CRT redistributable, no separate
  driver install.

  单文件安装包：一个 setup.exe 内嵌 GUI、代理 DLL 模板、32 位宿主，
  LZMA 压缩，不需要单独的 CRT 运行库或驱动。

---

## Requirements · 系统要求

- Windows 10 1809+ / Windows 11 (64-bit)
- A 64-bit VST2 host (Studio One, Cubase, Reaper, FL Studio, Live, Bitwig …)
- Administrator privileges (the installer creates files under
  `%ProgramFiles%`)

---

## Quick start · 快速开始

1. Download `BridgeUltra-Setup-<version>.exe` from the
   [Releases](https://github.com/GeekASMR/BridgeUltra/releases) page.

   从 Releases 页面下载安装包。

2. Run the setup; UAC prompts for elevation.

   双击运行，授予管理员权限。

3. After installation, launch **Bridge Ultra** from the Start Menu and
   point it at:
   - **Source** — a directory containing 32-bit VST2 `.dll` files
   - **Target** — a directory your 64-bit DAW scans for VSTs

   安装完成后，从开始菜单启动 Bridge Ultra：
   - **来源** — 包含 32 位 VST2 `.dll` 的目录
   - **目标** — 64 位 DAW 扫描 VST 的目录

4. Click **Add plugins**, review the scan, and deploy. The DAW will
   discover the new 64-bit proxy DLLs on its next scan.

   点"添加插件"完成扫描和部署。下次 DAW 扫描时即可识别 64 位代理 DLL。

---

## How it works · 工作原理

```
┌──────────────────────────┐
│      64-bit DAW host     │
│   (Studio One, Cubase…)  │
└────────────┬─────────────┘
             │ VST2 dispatcher
┌────────────▼─────────────────┐
│  BridgeUltraProxy64.dll      │  64-bit proxy DLL — what the DAW sees
│  (per-plugin instance)       │
└────────────┬─────────────────┘
             │ named pipes + shared memory + events
┌────────────▼─────────────────┐
│  BridgeUltraHost32.exe       │  32-bit sub-process — loads the real plugin
│  + the actual 32-bit VST2    │
└──────────────────────────────┘
```

Each proxy DLL is a small loader that, on first non-trivial dispatcher
call, lazy-spawns a 32-bit `BridgeUltraHost32.exe` and wires up:

- **Audio** — single-writer/single-reader shared-memory ring with
  zero-copy float/double sample buffers.
- **Control** — named pipe carrying VST2 dispatcher requests/responses,
  with the proxy as server and the host process as client.
- **Host callback** — reverse named pipe so the plugin can call back
  into the host for tempo, transport, automation events.
- **Editor** — 64-bit `WS_CHILD` container under the DAW panel; the
  plugin's 32-bit GUI is attached cross-process via standard Win32
  `CreateWindowEx(parent=…)` semantics. HWNDs are 32-bit kernel handles
  on every Windows architecture, which is why the cross-process attach
  works at all.

每个代理 DLL 是个小型加载器，首次实际 dispatcher 调用时延迟拉起 32 位
host32 子进程，把音频（共享内存零拷贝）/ 控制（命名管道 dispatcher）/
host 回调（反向管道）/ 编辑器（跨进程 HWND 嵌入）全部接通。

---

## What's new in v1.1.0

Compared to the v1.0 baseline:

- **New native installer GUI** — Win32 + Direct2D + DirectWrite + DComp,
  Mica backdrop on Win11, full bilingual zh/en, system theme aware.
- **Robust overwrite install** — uses MoveFileEx rename-aside +
  delete-on-reboot so the installer can replace `BridgeUltra.exe` even
  while a previous instance is running.
- **Fix: Studio One save latency** — `effSetProgram` / `effGetProgram`
  fast-fail on dead IPC instead of burning a 5 s timeout per plugin.
- **Fix: GUI stability** — JBridge-style stash-parent pattern keeps the
  cross-process plugin child window alive across DAW panel close /
  re-open cycles, which fixes "GUI doesn't appear on second open" for
  Cakewalk-style skin engines (Delay R3, Native Reverb Plus, Surround R3).
- **Fix: host32 zombie processes** — control pipe IO loop now clears
  the running flag on every exit path, so the 32-bit sub-process exits
  cleanly when the DAW unloads the plugin instead of lingering until
  the parent dies.
- **Fix: minimise → restore on Win11** — installer GUI uses
  `SWP_FRAMECHANGED` on `SIZE_RESTORED` to force DWM to re-apply our
  custom NCCALCSIZE rule, fixing the "transparent / blank window after
  minimise" symptom on Win11 22H2+.
- **Fix: theme detection under elevation** — installer reads
  `AppsUseLightTheme` from the launching user's HKEY_USERS hive
  (resolved via the active console session SID) rather than
  HKEY_CURRENT_USER, so the elevated installer no longer paints dark
  over a user's light theme.

---

## Known limitations · 已知限制

- **DRM / iLok-protected plugins** — many copy-protected plugins inspect
  the parent process tree and will see `BridgeUltraHost32.exe` instead
  of the DAW. This may break licence checks. No workaround is planned.

  受 iLok 等版权保护的插件可能会拒绝在子进程下加载，无解。

- **Bridge-on-bridge** — bridging a plugin that itself bridges another
  plugin (32→64→32) is not supported.

  桥中桥（32 → 64 → 32）不支持。

- **Repeated open / close of plugin GUI** — extremely rapid double-click
  cycles on the plugin slot can occasionally leave the editor blank in
  some hosts (Studio One). The mitigations in v1.1 cover the common
  patterns; the v1.x → v2.0 roadmap is to adopt the carla-bridge-win32
  full GUI shell model for complete robustness.

  极快速反复双击插件槽偶发 GUI 不显示。v1.1 已覆盖常见模式；v2.0 路线图
  计划改用 carla-bridge-win32 完整 GUI shell 模型彻底稳定。

See [docs/known-limitations.md](docs/known-limitations.md) for the full
list and [docs/troubleshooting.md](docs/troubleshooting.md) for diagnosis
hints.

---

## License

[MIT](LICENSE).

The VST2 ABI compatibility header used internally is `vestige.h`
(clean-room, BSD-style), not the Steinberg VST 2.4 SDK. Bridge Ultra has
no dependency on the Steinberg SDK and ships no Steinberg-licensed code.

内部使用 `vestige.h`（干净室实现，BSD 协议）作为 VST2 ABI 兼容头，
不依赖也不分发任何 Steinberg VST 2.4 SDK 代码。

---

## Acknowledgements · 致谢

Architectural inspiration drawn from JBridge (closed-source, the original
in this space) and Carla (GPL, open-source reference for the
sub-process bridging pattern). Bridge Ultra is an independent
implementation; no code is shared with either project.

架构思路参考 JBridge（闭源，业界先驱）与 Carla（GPL 开源，子进程桥接
模式参考）。Bridge Ultra 为独立实现，与上述项目无代码共享。
