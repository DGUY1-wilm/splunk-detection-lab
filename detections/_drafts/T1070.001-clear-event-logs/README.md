# T1070.001: Indicator Removal: Clear Windows Event Logs (DRAFT)

> **Status: draft, untested.** Nothing here has been run in the lab yet. No screenshots or findings exist for it.

| | |
|---|---|
| **Tactic** | Defense Evasion |
| **ATT&CK** | [T1070.001](https://attack.mitre.org/techniques/T1070/001/) |
| **Data sources** | Security Event 1102, Sysmon Event ID 1 |

## Planned simulation

**Take a snapshot first.** This wipes the local Security log.

```powershell
wevtutil cl Security
```

Or check `Invoke-AtomicTest T1070.001 -ShowDetails` for ready-made tests.

## Why it should work

Because the Universal Forwarder ships events off the machine as they happen, the clear action is recorded and the earlier history is already in Splunk. This is a good point to make in the final write-up, once confirmed.

## To validate

1. Snapshot the VM, run the simulation, and note the time.
2. Run the searches in [`detection.spl`](detection.spl) over Last 15 minutes.
3. Confirm event 1102 arrives in Splunk despite the local log being cleared.
4. Record false positives (admin maintenance) and limitations (an attacker could disable the forwarder or stop logging before clearing).
5. Revert the snapshot, and capture screenshots (`00-attack-run.png`, `01-security-1102.png`, `02-saved-alert.png`, `03-alert-triggered.png`).
