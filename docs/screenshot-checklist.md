# Screenshot checklist

Put each image in the folder shown, with exactly this name. The READMEs already link to these filenames.

Status key: **HAVE** (captured, use as is), **RECAPTURE** (captured but needs fixing), **TO CAPTURE** (not captured yet), **OPTIONAL**.

## setup/

| File | Status | Note |
|---|---|---|
| `01-sysmon-whoami-logs-verified.png` | HAVE | Sysmon whoami events with User, ParentImage, CommandLine |
| `02-sysmon-eventcode-counts.png` | RECAPTURE | Rerun on a normal time range (no `earliest=0 latest=+2d`), so it does not show a "10/13/26" date |

## setup/troubleshooting/ (all optional)

| File | Status | Note |
|---|---|---|
| `unattended-install-failure.png` | OPTIONAL | Ubuntu "Installation failed" screen |
| `splunk-root-warning.png` | OPTIONAL | "Running Splunk Enterprise as root is deprecated" |
| `forwarder-service-account.png` | OPTIONAL | `StartName: NT SERVICE\SplunkForwarder` |
| `wineventlog-only-no-sysmon.png` | OPTIONAL | Search showing only `wineventlog` for the Windows host |
| `internal-hosts.png` | OPTIONAL | `index=_internal | stats count by host` |
| `fields-blank-before-fix.png` | OPTIONAL | 4720 search with blank TargetUserName / SubjectUserName |

## detections/T1136.001-create-account/

| File | Status | Note |
|---|---|---|
| `00-attack-run.png` | TO CAPTURE | The Atomic test running |
| `01-atomic-test-details.png` | HAVE | `-ShowDetails` output |
| `02-sysmon-net-user-add.png` | HAVE | Show the full query if it was cut off |
| `03-security-4720-account-created.png` | HAVE | The `mvindex` version with account names filled in |
| `04-saved-alert.png` | TO CAPTURE | Saved alert settings |
| `05-alert-triggered.png` | TO CAPTURE | Triggered Alerts list |
| `06-cleanup.png` | TO CAPTURE | `-Cleanup` output |

## detections/T1059.001-encoded-powershell/

| File | Status | Note |
|---|---|---|
| `00-attack-run.png` | TO CAPTURE | The encoded command running |
| `01-sysmon-encodedcommand.png` | RECAPTURE | CommandLine column was cut off; zoom out or hide the sidebar |
| `02-scriptblock-4104-decoded.png` | HAVE | Decoded `whoami` script block |
| `03-saved-alert.png` | TO CAPTURE | |
| `04-alert-triggered.png` | TO CAPTURE | |

## detections/T1053.005-scheduled-task/

| File | Status | Note |
|---|---|---|
| `00-attack-run.png` | TO CAPTURE | The `schtasks /create` command running |
| `01-sysmon-schtasks-create.png` | HAVE | |
| `02-saved-alert.png` | TO CAPTURE | |
| `03-alert-triggered.png` | TO CAPTURE | |
| `04-cleanup.png` | TO CAPTURE | `schtasks /delete /tn LabTask /f` |

## detections/T1110-brute-force/

| File | Status | Note |
|---|---|---|
| `01-atomic-ad-prereq-note.png` | HAVE | Atomic T1110.001 tests target AD |
| `02-failed-logon-simulation.png` | HAVE | Six error 1326 results |
| `03-threshold-search-4625.png` | HAVE (optional recapture) | Last 15 minutes instead of All time |
| `04-raw-4625-events.png` | TO CAPTURE | Raw events with the `labfake` account name |
| `05-saved-alert.png` | TO CAPTURE | |
| `06-alert-triggered.png` | TO CAPTURE | |

## dashboards/

| File | Status |
|---|---|
| `01-dashboard-overview.png` | TO CAPTURE (after importing and testing the dashboard) |

## Before you commit any screenshot

- Check that no password, hash, or key is visible. Do not include the Ubuntu installer screenshot that showed password hashes.
- The Splunk username in the top-right corner appears in most Splunk screenshots. Crop it if you would rather not show it.
- Make sure the query, the results, and the time range are all readable.
- Remove any README image link whose screenshot you decide not to include, so the page has no broken-image icons.

## Facts to confirm before publishing

- The Windows version and edition of the victim VM (the README says "Windows VM" without a number).
- Which Splunk add-ons you actually installed (Sysmon, Windows).
- The ~02:15 timestamps and the account names in the incident report, against your screenshots.
- Whether the 5 extra failed-logon events in the T1110 README were investigated (and what they were).
