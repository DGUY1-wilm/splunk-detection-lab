# Lessons learned and future work

## What worked

- Pairing a process-based search with a log-based one (for example 4720 plus `net user /add`) caught more than either alone, and each covered the other's blind spot.
- Script block logging (4104) revealed the decoded content of an encoded PowerShell command that the process command line hid.
- Keeping notes while troubleshooting turned setup problems into documentation (see [`../setup/troubleshooting.md`](../setup/troubleshooting.md)).

## What I would do differently

- Skip VirtualBox's unattended install from the start. It caused two separate failures (Ubuntu and Windows).
- Set the forwarder's service account to Local System during install, so the Sysmon log is readable from day one.
- Fix the Windows time zone and clock before collecting any evidence, so timestamps and screenshots are consistent.
- Capture the attack running and the cleanup as screenshots at the time, not afterward.

## Known limits of this lab

- One endpoint, no domain, so Active Directory attacks (including the T1110.001 atomics) could not be run.
- Small data volume, so thresholds and false-positive rates are not representative of production.
- Detections were validated against my own test activity, not against real adversary behavior or large baselines.
- The Splunk trial license expires after 60 days. Scheduled alerting is lost on the Free license.

## Future work

- Save and trigger scheduled alerts for all four detections
- Validate the two draft detections (registry Run keys, log clearing)
- Add a second endpoint (Linux) and a domain controller to enable the AD-based Atomic tests
- Add network visibility (Zeek or Suricata) and detections on network data
- Add LOLBin abuse and LSASS credential-access detections
- Convert and test the Sigma rules with sigma-cli
- Tune detections against a baseline of normal activity
