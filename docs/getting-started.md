# Bridge Ultra — Getting Started

Bridge Ultra lets your 64-bit DAW load 32-bit VST2 plugins by
generating a small "proxy" DLL for each one. The proxy is what your DAW
sees; behind the scenes, the real 32-bit plugin runs in its own
sub-process.

## Install

1. Download `BridgeUltra-<version>.msi` from the releases page.
2. Double-click it. Click through Welcome → Install → Finish.
3. Bridge Ultra installs to `C:\Program Files\BridgeUltra\` and adds a
   Start Menu shortcut.

## Launch (UAC)

Starting with **v1.1**, `BridgeUltra.exe` always launches elevated. On
first launch (and every subsequent launch) Windows shows the standard
**User Account Control** prompt — click **Yes**. The window's title
bar is suffixed with "(管理员)" / "(Administrator)" to make this
visible. This change is required so the installer can write proxy DLLs
into directories like `C:\Program Files\Common Files\VST2\` without
prompting per file.

## Generate proxies (5-step Wizard, v1.1)

The Home view has a single primary button: **添加插件 / Add Plug-ins**.
It launches a five-step wizard. The bar at the top of the window shows
the current step.

1. **Step 1 — 选择源目录**: where your 32-bit plugins live, e.g.
   `C:\VST32\` or `C:\Program Files (x86)\VstPlugins\`. The textbox
   pre-fills from your last run, or you can pick from the recent
   history list.
2. **Step 2 — 选择目标目录**: where the 64-bit proxies should go. The
   default `C:\Program Files\Common Files\VST2\BridgeUltra\` is a safe
   choice. The wizard inline-validates the path: empty input is
   refused, source==target is refused, paths your DAWs already treat
   as 32-bit VST roots get a soft warning (you can still proceed),
   and a non-existent directory is offered to be created on Next.
3. **Step 3 — 扫描并选插件**: the scanner runs in the background and
   reports progress. Results are grouped by issue category — *Ready*,
   *Possible conflict / 64-bit duplicate exists*, *Path conflict
   needs review*, etc. Default selection rules pick the safe items.
4. **Step 4 — 部署中**: the deploy queue runs each generation job in
   sequence. Per-row status updates live; the overall progress bar
   advances. **停止后续 / Stop after current** safely cancels pending
   items without interrupting the in-flight job.
5. **Step 5 — 完成**: summary with success / failed / skipped counts,
   plus shortcuts to open the target folder, open the log folder,
   copy a deploy report, or finish back to Home. Source / target
   are recorded in the MRU history only after a successful run.

## Use

Bridged plugins behave like native VST2:

- Insert on a track.
- Open the editor — the original plugin's GUI appears, embedded in your
  DAW's plugin window.
- Save / restore: project state round-trips.
- Multiple instances: each runs in its own sub-process, so a crash in
  one plugin won't affect any others.

## Manage

The **管理已部署 / Manage Deployed** view (linked from Home) lists every
proxy currently installed in the target directory:

- **Search**: type to narrow by name, vendor, or uniqueID.
- **Multi-select**: click + Ctrl-click + Shift-click + Ctrl-A.
- **Uninstall**: removes the proxy DLL and its sidecar JSON for every
  selected row. The original 32-bit DLLs are left alone.
- **Re-probe**: refreshes the *stale* marker by re-running ProxyManager
  against the target directory.

## Settings

The **⚙ 设置** entry on Home opens the Settings dialog. Fields:

- **Interface language**: 中文 / English. Takes effect immediately.
- **Log level**: trace / debug / info / warn / error. Takes effect on
  next launch.
- **Scan worker count**: number of threads the Step 3 scanner uses.
  Takes effect on next scan.
- **Custom 32-bit DAW VST paths**: extra directories the wizard's
  Step 2 path-conflict detection will consider. Use this if your DAW
  scans a non-standard 32-bit folder.
- **Clear history**: empties the source / target MRU history.

## CLI / unattended

```powershell
BridgeUltra.exe --scan "C:\VST32" "C:\VST64\Bridge"
```

Recursively scans the source directory and overwrites all proxies in
the target directory. Useful for headless setup scripts.

## Next steps

- [Troubleshooting](troubleshooting.md) when things misbehave.
- [Architecture](architecture.md) if you want to know how it works.
- [Known limitations](known-limitations.md) before filing a bug.
