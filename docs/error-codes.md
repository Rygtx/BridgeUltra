# Bridge Ultra — Error codes (v1.1 Installer)

When the installer hits a recoverable problem during a deploy, scan,
or uninstall, it shows a three-section dialog:

```
[ What ]    user-facing label, e.g. "文件被占用，无法写入"
[ Why ]     concise root cause in plain English / Chinese
[ How to ]  one-line recovery action
[ Copy error details ]   builds a paste-ready report including the
                          Win32 error code, the offending path, the
                          plugin name, and the last 5 log lines
```

This page maps the Win32 error codes the installer recognises to the
canonical user-facing triplet. The runtime mapping lives in
`installer/src/ui/controller/error_presenter.cpp` (`forWin32` switch);
the visible strings live in `installer/src/ui/i18n/strings.h`. Keep the
two in sync — the unit test `i18n_resource_test` will refuse to compile
if a key referenced from `error_presenter` is missing in the i18n table.

| Error category | Win32 codes | i18n keys (zh / en) | Typical recovery |
|---|---|---|---|
| File locked | `ERROR_SHARING_VIOLATION` (32), `ERROR_LOCK_VIOLATION` (33) | `Error_FileLocked_*` | Close any DAW that may be loading the plug-in; temporarily disable antivirus real-time protection if needed. |
| Access denied | `ERROR_ACCESS_DENIED` (5) | `Error_AccessDenied_*` | Restart the installer as administrator. (v1.1 manifest requires this — should not occur in production.) |
| Path not found | `ERROR_PATH_NOT_FOUND` (3), `ERROR_FILE_NOT_FOUND` (2), `ERROR_INVALID_NAME` (123) | `Error_PathNotFound_*` | Go back and choose a valid path. |
| Disk full | `ERROR_DISK_FULL` (112), `ERROR_HANDLE_DISK_FULL` (39) | `Error_DiskFull_*` | Free up space or pick a different target volume. |
| Probe SEH crash | (synthetic — raised by host32 probe) | `Error_ProbeSeh_*` | Skip this plug-in; contact the author if it's important. |
| Cancelled | `ERROR_CANCELLED` (1223) | `Error_Cancelled_*` | Restart the installer to retry. |
| Other | any other code | `Error_Generic_*` | "Copy error details" and attach the log to a bug report. |

## Clipboard report format

Pressing **复制错误详情 / Copy error details** writes a single multi-line
string into the clipboard, ready for pasting into a bug report:

```
Bridge Ultra v1.1 — error report
====================================
Operation       : <operation>
Path            : <offending path>
Plugin          : <plugin friendly name>
Win32 code      : 32 (ERROR_SHARING_VIOLATION)
Win32 message   : The process cannot access the file because it is being used by another process.
Time (UTC)      : 2026-05-23T14:22:31Z
OS              : Windows 11 26100
DPI scale       : 1.50
Last log lines  :
  [2026-05-23 14:22:30.123] [installer] [warn] deploy: write failed pl=Classic Chorus
  [2026-05-23 14:22:31.052] [installer] [error] deploy: ERROR_SHARING_VIOLATION at C:\Program Files\Common Files\VST2\BridgeUltra\Classic Chorus.dll
  ...
```

Telling users to paste this verbatim is the cheapest way to get a
diagnosable bug report.
