# T1136.001: Create Account: Local Account

| | |
|---|---|
| **Tactic** | Persistence |
| **ATT&CK** | [T1136.001](https://attack.mitre.org/techniques/T1136/001/) |
| **Data sources** | Sysmon Event ID 1 (process creation), Windows Security Event 4720 (account created) |
| **Severity (suggested)** | Medium |
| **Status** | Validated against lab data. Scheduled alert not yet saved. |

## Summary

An attacker who gains access often creates a local account to keep a way back in. This detection watches for the two traces of that action: the process that ran (`net user /add`) and the Windows event recording the new account (4720).

## Simulation

Run on the Windows VM with Atomic Red Team. The test creates local accounts from a command prompt (`net user /add`) and from PowerShell (`New-LocalUser`).

```powershell
Invoke-AtomicTest T1136.001 -ShowDetails
Invoke-AtomicTest T1136.001
```

Accounts created during testing included `T1136.001_CMD`, `T1136.001_PowerShell`, and `T1136.001_Admin`.

![Atomic test details](../../screenshots/Detection%25201/01-atomic-test-details.png)

<!-- TO CAPTURE: the attack actually running in PowerShell -->

## Evidence

**Process creation (Sysmon EID 1).** `cmd.exe` launches `net.exe`, which launches `net1.exe`, both carrying the `/add` command line:

![Sysmon net user /add](../../screenshots/Detection%25202-sysmon-net-user-add.png)

**Account created (Security 4720).** The new accounts and who created them:

![Security 4720](../../screenshots/Detection%25203-security-4720-account-created.png)

## Detection logic

The saved searches live in [`detection.spl`](detection.spl), and the Sigma version is in [`rule.yml`](rule.yml).

- **Search A** matches `net.exe` or `net1.exe` with `user` and `/add` in the command line. This catches the command-line method regardless of which account name is used.
- **Search B** reads Security 4720. This is the more reliable signal because it fires for any local account creation, including PowerShell `New-LocalUser`, which Search A does not see.

Using both gives a process view (how and from where) and a result view (that an account now exists).

## False positives observed in the lab

1. **Forwarder install.** Search A also returned `net localgroup "Performance Monitor Users" "NT SERVICE\SplunkForwarder" /add`, run by `NT AUTHORITY\SYSTEM` at install time. The wildcard `*user*` matched the word "Users". A `NOT CommandLine="*localgroup*"` filter (and the Sigma `filter_localgroup`) removes it.
2. **System-created accounts.** Search B also listed `defaultuser0`, `WDAGUtilityAccount`, and the `windows` account, which Windows created itself during setup, with machine-account creators such as `DESKTOP-KF26PU8$`. Excluding creators that end in `$` removes them.
3. **Odd timestamps.** A few of these rows carry times that do not fit the test window. They date from the early setup period, when the VM clock and time zone were still being corrected.

In production the biggest source of false positives would be help desk and provisioning activity, so a real deployment needs an allowlist of approved admin accounts and change windows.

## Limitations and evasion

- Search A only sees `net.exe` and `net1.exe`. Accounts created with PowerShell, WMI, or direct API calls evade it, which is why Search B matters.
- Search B only covers local accounts. Domain account creation (4720 on a domain controller) needs its own collection.
- An attacker with admin rights could clear logs afterward (see the draft T1070.001 detection).
- The check is event-based, so alert latency depends on the schedule.

## Tuning ideas

- Allowlist known admin accounts and approved change windows.
- Alert only on non-business-hours account creation.
- Combine with "account added to Administrators" (4732) for higher severity.

## Saved alert

Settings are in [`../ALERTS.md`](../ALERTS.md). Scheduled alerts require the trial license, so capture these before it ends.

<!-- TO CAPTURE -->

## Cleanup

```powershell
Invoke-AtomicTest T1136.001 -Cleanup
```

<!-- TO CAPTURE -->

