# T1059.001: Command and Scripting Interpreter: PowerShell

| | |
|---|---|
| **Tactic** | Execution |
| **ATT&CK** | [T1059.001](https://attack.mitre.org/techniques/T1059/001/) |
| **Data sources** | Sysmon Event ID 1, PowerShell Operational Event 4104 (script block logging) |
| **Severity (suggested)** | Medium |
| **Status** | Validated against lab data. Scheduled alert not yet saved. |

## Summary

Attackers encode PowerShell commands in base64 to hide their intent from casual log review. This detection catches the launch of PowerShell with an encoded-command flag, then uses script block logging to see the decoded content.

## Simulation

A manual, harmless test: a base64 string encoding `whoami`, run through `powershell.exe -EncodedCommand` (or an equivalent command).

```powershell
$e = [Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes("whoami"))
powershell.exe -EncodedCommand $e
```

<!-- TO CAPTURE: the command above running in PowerShell -->

This was a manual test, not an Atomic Red Team test. (The Atomic T1059.001 test 1 is a Mimikatz download, which was not run.)

## Evidence

**How it was launched (Sysmon EID 1).** `powershell.exe` started from another PowerShell session, with `-encodedcommand` and the base64 string in the command line:

<!-- RECAPTURE: the CommandLine column was cut off at the right edge. Zoom out so the full command line shows. -->
![Sysmon encoded command](01-sysmon-encodedcommand.png)

**What it actually ran (event 4104).** Script block logging recorded the decoded text, `whoami`, one second after the process started. The ScriptBlock ID in the event can be used to pivot between related events:

![Script block 4104](02-scriptblock-4104-decoded.png)

## Detection logic

See [`detection.spl`](detection.spl) and [`rule.yml`](rule.yml).

- **Search A** finds the launch by looking for `-enc` or `-EncodedCommand` on `powershell.exe`.
- **Search B** reads the decoded content from event 4104, which is where the defender sees what the encoded string really did.

Pairing the two is the useful part: A tells you something was hidden, B tells you what.

## False positives

Not observed in the lab beyond the test itself. In real environments, deployment and management tools sometimes pass encoded commands, so expect to allowlist known parent processes.

## Limitations and evasion

- Search A matches the literal strings `-enc` and `-EncodedCommand`. Other abbreviations (`-e`, `-ec`, `-en`) or obfuscated spellings can slip past it. The broader search in `detection.spl` covers more, with more noise (untested).
- Attackers can avoid the command-line flag entirely by decoding inside a script, which only the 4104 data would show.
- Script block logging must be enabled (done via Group Policy in this lab). Without it, Search B returns nothing.
- Time stamps differ by 4 hours between the raw Windows event (local time) and Splunk's display, which is a time zone difference, not an error.

## Tuning ideas

- Alert when the decoded content in 4104 contains known-bad strings (download cradles, `Invoke-Expression`).
- Raise severity when the parent process is an Office app, a browser, or a script host.
- Allowlist approved management tools by parent image.

## Saved alert

Settings are in [`../ALERTS.md`](../ALERTS.md).

<!-- TO CAPTURE -->


## Cleanup

Nothing to clean up. The test only runs `whoami`.
