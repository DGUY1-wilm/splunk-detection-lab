# T1110.001: Brute Force: Password Guessing

| | |
|---|---|
| **Tactic** | Credential Access |
| **ATT&CK** | [T1110.001](https://attack.mitre.org/techniques/T1110/001/) |
| **Data sources** | Windows Security Event 4625 (failed logon) |
| **Severity (suggested)** | Medium |
| **Status** | Validated against lab data using a **manual simulation**. Scheduled alert not yet saved. |

## Summary

Repeated failed logons in a short time often mean someone is guessing passwords. This detection counts failed logons (event 4625) per host in 5-minute windows and flags bursts of five or more.

## Simulation (manual, not Atomic Red Team)

`Invoke-AtomicTest T1110` fails because T1110 is a parent technique with no atomics of its own. The T1110.001 atomics exist, but they brute force **Active Directory domain accounts** (over SMB against `%logonserver%`, and over LDAP against a domain controller). This lab VM is a standalone workgroup machine with no domain, so those tests do not apply.

![Atomic tests target Active Directory](../../screenshots/Detection%25206/01-atomic-ad-prereq-note.png)

Instead, I ran a harmless manual simulation: six failed network logons against the local machine as a nonexistent user.

```powershell
1..6 | ForEach-Object { net use \\127.0.0.1\IPC$ /user:labfake BadPass$_ }
```

Each attempt returned `System error 1326: The user name or password is incorrect`.

![Failed logon simulation](../../screenshots/Detection%2520602-failed-logon-simulation.png)

## Evidence

The threshold search found one 5-minute bucket with a count of 6, matching the six attempts:

<!-- OPTIONAL RECAPTURE: set the time range to "Last 15 minutes" instead of "All time" so it matches how the alert will run. -->
![Threshold search](../../screenshots/Detection%2520603-threshold-search-4625.png)

<!-- TO CAPTURE: the raw 4625 events showing the "labfake" account name (see Search B in detection.spl) -->

The search scanned 11 failed-logon events in total, and 6 fell in the burst bucket, matching the six simulated attempts. The other 5 fell outside that bucket.

<!-- TODO: if you investigated the other 5 events (screenshot), describe what they were here. Until then this README does not claim a cause. -->

## Detection logic

See [`detection.spl`](detection.spl) and [`rule.yml`](rule.yml). The Sigma file uses the Sigma correlation format (a base rule plus an `event_count` correlation), so it needs a recent sigma-cli and a backend that supports correlations. It converts to Splunk syntax without errors in sigma-cli 3.1.0, but the converted query has not been run in the lab.

## False positives

Mistyped passwords, expired credentials, and services or scheduled tasks using stale passwords all generate 4625 events. A threshold of 5 in 5 minutes is a starting point, not a tuned value.

## Limitations and evasion

- **Low and slow attacks.** An attacker staying under 5 attempts per 5 minutes is not caught. A longer window or per-account counts would help.
- **Password spraying.** Spraying tries one password across many accounts, so each account sees few failures. Search C in `detection.spl` (distinct accounts per host, untested) is aimed at that pattern.
- **Source visibility.** This lab simulation was local, so the source address is the machine itself. A real detection should group by the source IP too.
- The simulation does not exercise Kerberos or LDAP failure events, which is where domain attacks would appear.

## Tuning ideas

- Group by source address as well as host.
- Use different thresholds for service accounts and for interactive accounts.
- Alert on a run of failures followed by a success (4624) for the same account.

## Saved alert

Settings are in [`../ALERTS.md`](../ALERTS.md).

<!-- TO CAPTURE -->

## Cleanup

Nothing to clean up. The attempts failed and created no accounts or files.
