# Saved alert settings

None of these alerts have been saved and fired yet. Use these settings, then capture the saved alert and a triggered alert for each detection folder.

**Timing:** Scheduled alerting needs the Enterprise trial license. The Free license drops it. The trial lasts 60 days from install (around December 10, 2026 if installed October 10 to 11; confirm under **Settings > Licensing**). Capture alert screenshots before then.

## Common settings

| Setting | Value |
|---|---|
| Alert type | **Scheduled** (not Real-time) |
| Schedule | **Run on Cron Schedule**, `*/5 * * * *` |
| Time range | **Last 15 minutes** (set the picker before using Save As, and remove any `earliest=0 latest=+2d` from the search) |
| Trigger when | Number of Results, is greater than, `0` |
| Trigger | Once |
| Throttle | Suppress for 10 minutes |
| Expires | 7 day(s) |
| Permissions | Private |
| Trigger action | Add to Triggered Alerts, severity Medium |

## Per alert

| Detection | Title | Search |
|---|---|---|
| T1136.001 | `T1136.001 - Local Account Created` | Search B (Security 4720). Optionally add Search A-tuned as a second alert. |
| T1059.001 | `T1059.001 - Encoded PowerShell Launch` | Search A |
| T1053.005 | `T1053.005 - Scheduled Task Created` | Search A (consider low severity) |
| T1110.001 | `T1110.001 - Failed Logon Burst` | Search A |

## Test each one end to end

1. Run the simulation on the Windows VM.
2. Wait up to 5 minutes for the schedule to run.
3. Open **Alerts > Triggered Alerts** (or search `index=_audit action=alert_fired`) and screenshot it.
4. Clean up.

If an alert does not fire, run its search manually over Last 15 minutes right after the simulation. If that returns your event, only the schedule or time range needs fixing.

## Optional: `savedsearches.conf` (untested)

An alternative to the UI. Place it in `$SPLUNK_HOME/etc/apps/search/local/savedsearches.conf` and restart Splunk. Test it before relying on it.

```ini
[T1110.001 - Failed Logon Burst]
search = index=wineventlog EventCode=4625 | bucket _time span=5m | stats count by _time host | where count>=5
enableSched = 1
cron_schedule = */5 * * * *
dispatch.earliest_time = -15m
dispatch.latest_time = now
counttype = number of events
relation = greater than
quantity = 0
alert.severity = 3
alert.suppress = 1
alert.suppress.period = 10m
alert.track = 1
```
