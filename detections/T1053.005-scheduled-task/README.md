# T1053.005: Scheduled Task/Job: Scheduled Task

| | |
|---|---|
| **Tactic** | Persistence, Privilege Escalation, Execution |
| **ATT&CK** | [T1053.005](https://attack.mitre.org/techniques/T1053/005/) |
| **Data sources** | Sysmon Event ID 1 (Security 4698 optional, see below) |
| **Severity (suggested)** | Low to Medium (depends on what the task runs and as whom) |
| **Status** | Validated against lab data. Scheduled alert not yet saved. |

## Summary

Scheduled tasks let an attacker run something again at a set time, at logon, or at startup. This detection flags new tasks created with `schtasks.exe`.

## Simulation

A manual task that opens Notepad every day at 09:00:

```powershell
schtasks /create /tn LabTask /tr notepad.exe /sc daily /st 09:00
```

The Atomic Red Team T1053.005 tests were reviewed but the evidence here comes from the manual task above. For reference, Atomic test 1 registers `T1053_005_OnLogon` and `T1053_005_OnStartup` tasks that launch `calc.exe`.

<!-- TO CAPTURE: the schtasks command running in PowerShell -->

## Evidence

Sysmon recorded `schtasks.exe` with the full `/create` command line, launched from PowerShell by the local user:

![Sysmon schtasks create](01-sysmon-schtasks-create.png)

## Detection logic

See [`detection.spl`](detection.spl) and [`rule.yml`](rule.yml).

The validated search matches `schtasks.exe` with `/create`. The untested "risk" variant adds two flags that make a task more suspicious: running as SYSTEM, and triggering at logon or startup.

## False positives

None observed beyond the test. In practice, software installers and updaters create tasks constantly, so alerting on every `/create` is noisy. Treat this as a hunting or enrichment signal unless it is combined with context such as an unusual parent process, a task running from a temp directory, or SYSTEM-level execution.

## Limitations and evasion

- Tasks can be created without `schtasks.exe`, using the Task Scheduler COM API, PowerShell's `Register-ScheduledTask`, or by writing task XML directly. None of those run `schtasks.exe`, so this search misses them.
- Sysmon records the command line, but not what the task later does. A second detection on the task's execution would be needed for full coverage.
- Security event 4698 (task created) is a more complete signal but only exists if the audit subcategory is enabled. It was not captured in this lab.

## Tuning ideas

- Alert on tasks that run from user-writable paths (`%TEMP%`, `AppData`, `Downloads`).
- Raise severity for `/ru system`, `/sc onlogon`, and `/sc onstart`.
- Allowlist known installer parents.

## Saved alert

Settings are in [`../ALERTS.md`](../ALERTS.md).

<!-- TO CAPTURE -->

## Cleanup

```powershell
schtasks /delete /tn LabTask /f
```

<!-- TO CAPTURE -->
