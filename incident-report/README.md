# Incident report INC-LAB-001 (simulated)

> **Read this first.** This is a write-up of **simulated** activity in my own lab. The four events below were run as separate tests by me, in sequence, on a VM I own. They were not a single automated attack chain, and no real system was involved. The report is written the way a SOC ticket would be, built only from evidence the lab produced.

| | |
|---|---|
| **Ticket** | INC-LAB-001 |
| **Title** | Suspected hands-on-keyboard activity on DESKTOP-KF26PU8 |
| **Host** | DESKTOP-KF26PU8 (Windows VM) |
| **User** | `DESKTOP-KF26PU8\windows` (local account) |
| **Source of alerts** | Splunk, from Sysmon and Windows Security / PowerShell logs |
| **Suggested severity** | High (persistence plus credential-guessing on one host) |
| **Analyst** | Daniel Guy |
| **Times** | As shown in Splunk, which appears to display UTC. Windows local time (US Eastern) is 4 hours earlier. |

## Summary

Over just over an hour, one local user on a single Windows host created new local accounts, ran an encoded PowerShell command, created a scheduled task, and then produced a burst of failed logons. Taken together, that sequence looks like an intruder establishing persistence and then testing credentials.

## Timeline

| Time (Splunk) | Event | ATT&CK | Evidence |
|---|---|---|---|
| ~02:15:20 to 02:15:27 | Local accounts created: `T1136.001_CMD`, `T1136.001_PowerShell`, `T1136.001_Admin`, and `NewLocalUser`, all by `windows`. Sysmon shows `cmd.exe` launching `net.exe` and `net1.exe` with `user /add`. | T1136.001 Persistence | Security 4720, Sysmon EID 1 |
| 03:07:53 | `powershell.exe` launched with `-encodedcommand` by `windows`, from another PowerShell session. | T1059.001 Execution | Sysmon EID 1 |
| 03:07:54 | Script block log records the decoded content: `whoami`. | T1059.001 | PowerShell 4104 |
| 03:13:32 | `schtasks /create /tn LabTask /tr notepad.exe /sc daily /st 09:00`, launched from PowerShell. | T1053.005 Persistence | Sysmon EID 1 |
| 03:20 to 03:25 window | Burst of 6 failed logons (4625) on the host. | T1110.001 Credential Access | Security 4625 |

<!-- Verify the 02:15 times against your screenshots before publishing. -->

## Analysis

- **Persistence came first and twice.** New accounts (T1136.001) and a scheduled task (T1053.005) are both ways to keep access.
- **The encoded command is a mild signal on its own.** The decoded content was only `whoami`, but the use of encoding is what a defender would want flagged, and the 4104 log is what reveals the real command.
- **The failed-logon burst follows account activity.** Six failures inside a 5-minute window crossed the 5-attempt threshold.
- **One host, one user.** No evidence in this data of movement to other machines (the lab has only one endpoint).

## Scope

Single host, single user account. Whether other hosts were touched cannot be determined from this lab.

## Recommended response (what a real analyst would do)

1. **Contain:** isolate the host from the network.
2. **Remove persistence:** delete the rogue local accounts and the `LabTask` scheduled task.
3. **Credentials:** reset passwords for the affected user and any account that logged in on the host.
4. **Investigate:** determine how the user session was obtained, and review the account's other activity.
5. **Hunt:** search other hosts for the same patterns (encoded PowerShell, new accounts, new scheduled tasks, failed-logon bursts).

<!-- TODO: state whether the accounts and the task were actually cleaned up in the lab (cleanup screenshots), and link them below. -->

## Detections that fired or would fire

| Detection | Folder |
|---|---|
| Local account created | [T1136.001](../detections/T1136.001-create-account/) |
| Encoded PowerShell | [T1059.001](../detections/T1059.001-encoded-powershell/) |
| Scheduled task created | [T1053.005](../detections/T1053.005-scheduled-task/) |
| Failed-logon burst | [T1110.001](../detections/T1110-brute-force/) |

Scheduled alerts have not yet been saved, so "would fire" is accurate until they are captured.

## Gaps and lessons

- No detection yet for log clearing (draft T1070.001) or registry persistence (draft T1547.001), which an intruder here might also use.
- The scheduled-task and failed-logon searches are noisy in real environments and would need context before paging anyone.
- The local-account search initially flagged the forwarder's own install activity. See the T1136.001 README.

## Evidence

<!-- Reuse the screenshots from the detection folders; adjust paths if you rename them. -->

![Accounts created](../detections/T1136.001-create-account/03-security-4720-account-created.png)

![Encoded PowerShell](../detections/T1059.001-encoded-powershell/01-sysmon-encodedcommand.png)

![Scheduled task](../detections/T1053.005-scheduled-task/01-sysmon-schtasks-create.png)

![Failed logon burst](../detections/T1110-brute-force/03-threshold-search-4625.png)
