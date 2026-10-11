# T1547.001: Registry Run Keys / Startup Folder (DRAFT)

> **Status: draft, untested.** Nothing here has been run in the lab yet. No screenshots or findings exist for it.

| | |
|---|---|
| **Tactic** | Persistence (also Privilege Escalation) |
| **ATT&CK** | [T1547.001](https://attack.mitre.org/techniques/T1547/001/) |
| **Data source** | Sysmon Event ID 13 (registry value set) |

## Planned simulation

On the Windows VM, from a snapshot:

```powershell
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v LabTest /t REG_SZ /d "C:\Windows\System32\notepad.exe" /f
```

Cleanup:

```powershell
reg delete "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v LabTest /f
```

You can also check `Invoke-AtomicTest T1547.001 -ShowDetails` for ready-made tests.

## To validate

1. Run the simulation and note the time.
2. Run the search in [`detection.spl`](detection.spl) over Last 15 minutes.
3. Confirm `TargetObject` shows the Run key and `Image` shows what changed it. If nothing returns, check that the Sysmon config logs registry events.
4. Record false positives (installers often write Run keys) and limitations (for example, other persistence keys and the Startup folder are not covered).
5. Capture screenshots (`00-attack-run.png`, `01-sysmon-run-key-set.png`, `02-saved-alert.png`, `03-alert-triggered.png`, `04-cleanup.png`).
