# Bridge Ultra — Troubleshooting

## Where to look first

- **Logs**:
  `%LOCALAPPDATA%\BridgeUltra\logs\bridge.<YYYY-MM-DD>.log`
  Daily rotation. The `[proxy64]`, `[host32]`, and `[installer]`
  process tags help identify which side of the bridge a message came
  from. The `[lifecycle]`, `[ipc]`, `[audio]`, `[gui]`, `[installer]`
  category tags narrow it further.
- **Crash overlay**: when a plugin crashes its sub-process, the proxy
  paints a small "Plugin process crashed — Restart" widget over the
  editor. Click Restart to respawn.
- **Settings panel** in the GUI exposes the log level (`info` →
  `debug` is the typical first step when filing a bug).

## Common symptoms

### "Plugin not found" / sidecar missing

The proxy refused to load because its sibling `.bridge.json` is gone
or points at a moved DLL.

- Re-run the scan to regenerate sidecars.
- If you moved the original 32-bit DLL, update its path in the JSON
  (the file is plain text) or just delete + regenerate the proxy.

### DAW reports "0 plugins found" after install

- Confirm the target directory matches the directory your DAW scans.
  The default `C:\Program Files\Common Files\VST2\BridgeUltra\` works
  for most hosts, but Studio One and FL prefer `%ProgramFiles%\Steinberg\VstPlugins\`
  or a per-user directory.
- Some DAWs cache plugin scans aggressively. In Cubase, click *Update*
  in the plugin manager. In FL Studio, **Manage plugins → Find plugins**.

### Editor shows blank / black

- Check the log for `gui` lines. A common cause is a plugin that needs
  a specific Win32 message it didn't get during embedding.
- DPI: try toggling the host's plugin-window DPI mode. The Bridge
  Ultra sub-process is `Per-Monitor V2` aware; some hosts override.

### "Plugin process crashed" overlay appears immediately

The original 32-bit DLL crashed during `effOpen`. Most often:
- Missing dependency (other DLLs the plugin expects in its directory).
  Copy the entire 32-bit installation, not just the .dll.
- Encrypted / DRM plugins (e.g. iLok-protected) often check parent
  process hierarchy. Bridge Ultra cannot help with these — talk to the
  plugin vendor.

### High CPU usage

- Most of the 5–10 % overhead is unavoidable (cross-process audio
  buffer copy + scheduling). Heavy plugins should still feel native.
- The latency benchmark in `tests/integration/latency_bench.cpp`
  measures the additional latency on your CPU vs an in-process load.
- Try reducing the host's audio buffer size; smaller buffers mean
  fewer per-block bookkeeping costs relative to actual processing.

### Multiple instances refuse to run

The proxy spawns one `BridgeUltraHost32.exe` per instance. If the
target DAW limits sub-processes (rare), expect 16+ instances to fail.
Workaround: increase the DAW's plugin-process budget, or split the
project across DAW sessions.

## Reporting a bug

When opening an issue please attach:

1. The relevant `bridge.YYYY-MM-DD.log` snippet (around the failure).
2. The `Settings → About` versions of installer / proxy / host32.
3. The `<plugin>.bridge.json` sidecar of the affected plugin.
4. The DAW name + version.
5. A repro project file or step-by-step description.

The `installer/Settings → Open log folder` button takes you straight to
the logs.
