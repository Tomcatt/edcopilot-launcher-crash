# EDCoPilot won't start: LaunchEDCoPilot.exe crashes

If your Start Menu shortcut for [EDCoPilot](https://www.razzafrag.com/)
does nothing (no window, no EDCoPilot process, no new `EDCoPilot.log`),
the launcher is probably crashing before it starts the app.

Tested on EDCoPilot 1.11.765, Windows 11 (build 26200), October 2026.

## Symptom

The Start Menu shortcuts run `C:\EDCoPilot\LaunchEDCoPilot.exe`, not the app
itself. The launcher exits and EDCoPilot never appears. Running
`C:\EDCoPilot\EDCoPilot.exe` directly works fine.

Look in `C:\EDCoPilot\logs\` for the newest `LaunchEDCoPilot.log.*.txt`.
It ends like this:

```
SetFocusOnElite: invoked
Other error occurred while trying to set focus: (0, 'SetForegroundWindow', 'No error message is available')
  File "<frozen LaunchEDCoPilot>", line 78, in SetFocusOnElite
pywintypes.error: (0, 'SetForegroundWindow', 'No error message is available')

During handling of the above exception, another exception occurred:
  File "<frozen LaunchEDCoPilot>", line 673, in <module>
  File "<frozen LaunchEDCoPilot>", line 498, in LaunchCoPilot
  File "<frozen LaunchEDCoPilot>", line 84, in SetFocusOnElite
NameError: name 'prefs' is not defined
```

## Cause

The launcher tries to bring Elite Dangerous to the front. Windows often
refuses `SetForegroundWindow`, for example when Elite isn't running or
another window has focus. That's normal and harmless.

The bug is in the launcher's error handler for that case: it refers to a
variable, `prefs`, that doesn't exist at that point. The resulting
`NameError` kills the launcher before it starts EDCoPilot.

The launcher (`LaunchEDCoPilot.exe`, dated 2025-06-03) is not replaced by
EDCoPilot's in-app updates, so updating EDCoPilot doesn't fix it.

## Workaround

Point the shortcuts at `EDCoPilot.exe` instead of the launcher. The
shortcuts live in the all-users Start Menu, so this needs an **elevated**
PowerShell:

```powershell
$sh = New-Object -ComObject WScript.Shell
Get-ChildItem 'C:\ProgramData\Microsoft\Windows\Start Menu\Programs' -Recurse -Filter *.lnk |
  Where-Object { $sh.CreateShortcut($_.FullName).TargetPath -like '*LaunchEDCoPilot.exe' } |
  ForEach-Object {
    $l = $sh.CreateShortcut($_.FullName)
    $l.TargetPath       = 'C:\EDCoPilot\EDCoPilot.exe'
    $l.WorkingDirectory = 'C:\EDCoPilot'
    $l.IconLocation     = 'C:\EDCoPilot\EDCoPilot.exe,0'
    $l.Save()
    "Fixed: $($_.FullName)"
  }
```

Adjust `C:\EDCoPilot` if you installed somewhere else. Reinstalling
EDCoPilot may put the old shortcuts back.

What you lose by skipping the launcher: its single-instance check and its
attempt to focus Elite after starting. EDCoPilot itself still runs
normally.

## Status

Reported to the developer in the EDCoPilot Discord (`#bug-reports`) on
2026-10-21. I'll update this page if a fixed launcher ships.

## License

MIT, see [LICENSE](LICENSE).
