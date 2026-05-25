# Bridge Ultra — Known Limitations

## Out of scope (won't ship)

These were explicitly excluded from the v1 scope and are unlikely to
change:

- **VST3, CLAP, AU, AAX bridging** — only VST 2.4 is supported.
  Architecture is extension-friendly so a future major version could
  add VST3, but no work is planned.
- **64-bit → 32-bit reverse bridging** — running 64-bit plugins in old
  32-bit hosts. Not a goal.
- **macOS / Linux** — Windows-only. Many APIs we rely on (Job Object,
  per-monitor DPI v2, named events with PIPE_TYPE_MESSAGE) have no
  exact equivalents on other OSes.
- **Network / remote bridging** — Vienna Ensemble Pro–style LAN
  hosting. Not a goal.
- **Custom plugin GUIs** — Bridge Ultra forwards the plugin's own
  editor; it does not synthesise an alternative parameter UI.
- **DRM / iLok bypass** — many copy-protected plugins inspect the
  parent process tree. They will see `BridgeUltraHost32.exe` instead
  of the DAW. If the plugin refuses to load, talk to the plugin
  vendor — Bridge Ultra cannot circumvent any protection.

## Soft limitations (might improve)

These could be lifted in a future release if there is demand:

- **Plugin GUI embedding stability** — v0.1.0 successfully shows the
  plugin GUI on first open in most DAWs we tested. However, repeated
  open/close cycles can cause black-rendered or unresponsive GUIs in
  some hosts (Studio One 6 in particular). Audio, parameter automation,
  and state save/load are unaffected. v1.x will adopt the
  carla-bridge-win32 architecture (full main-thread Win32 GUI shell in
  the 32-bit child process); see `docs/carla-reference.md` for the
  detailed roadmap. Workaround: configure plugin parameters in the
  original 32-bit host, save the chunk/state, then load the bridged
  proxy in the 64-bit DAW which will restore that state automatically.
- **Single-shot plugin entry** — the proxy assumes the plugin's
  `VSTPluginMain` is idempotent. Some weird plugins instantiate one
  global per process; with one sub-process per instance we accidentally
  fix some of those, but cross-instance state (e.g. shared license
  files) is the plugin's responsibility.
- **Sample-rate / block-size changes during playback** — supported,
  but we re-allocate the audio shm region on a block-size increase.
  The first such block will under-run; subsequent blocks recover.
- **Heartbeat / dedicated pipe** — the heartbeat thread shares the
  control pipe; under heavy dispatcher load a heartbeat may queue
  behind a slow dispatcher response. The proxy uses generous timeouts
  to avoid spurious "crashed" detection but a dedicated heartbeat
  pipe is on the roadmap.

## Plugin-specific notes

| Plugin                  | Note                                                       |
|-------------------------|------------------------------------------------------------|
| BT Tempo Delay 3D       | Reports `numPrograms=5`; works as expected.                |
| Classic Chorus          | `programChunks=false`; param round-trip handles all state. |
| Delay R3                | Reports empty effect/vendor strings. Cosmetic only.        |
| Native Reverb Plus      | `programChunks=true`; chunk get/set must be exercised.     |
| Surround R3             | Reports empty strings (cosmetic only).                     |

## Compatibility matrix

Tested at least once each release on the hosts in
`tests/e2e/checklist.md`. **Release sign-off requires ≥ 4 hosts pass.**

| Host           | Status                                                          |
|----------------|-----------------------------------------------------------------|
| Reaper 7       | Primary. Should always pass.                                    |
| FL Studio 21   | Plugin scan caches aggressively — re-scan after install.        |
| Cubase 12      | Use *Information* → *Update* in plugin manager after install.   |
| Studio One 6   | Per-user VST2 paths preferred; defaults work otherwise.         |
| Ableton Live 11| Plugin folder must be in *Preferences* → *VST Plug-in Custom Folder*. |
| Bitwig 5       | First scan slow; subsequent scans cache.                        |

## Filing limitation requests

If you hit a behaviour that isn't documented here, please open an
issue with:

- The exact plugin (vendor + version + source).
- The DAW name + version.
- The Bridge Ultra logs at `info` and `debug` levels around the failure.
- Whether the plugin works in `BridgeUltraHost32.exe --probe <dll> <out>`
  (gives a quick yes/no on plugin loadability before any audio routing
  is involved).
