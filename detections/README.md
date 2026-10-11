# Detections

One folder per technique. Every folder has the same layout:

```
T<ID>-<name>/
├── README.md       technique, simulation, evidence, logic, false positives, limitations
├── detection.spl   Splunk searches (validated and tuned variants are labeled)
├── rule.yml        the same logic as a Sigma rule
└── NN-*.png        numbered screenshots in the order a reader should follow them
```

| Technique | Folder | Status |
|---|---|---|
| T1136.001 Create Account: Local Account | [T1136.001-create-account](T1136.001-create-account/) | Validated |
| T1059.001 PowerShell | [T1059.001-encoded-powershell](T1059.001-encoded-powershell/) | Validated |
| T1053.005 Scheduled Task | [T1053.005-scheduled-task](T1053.005-scheduled-task/) | Validated |
| T1110.001 Password Guessing | [T1110-brute-force](T1110-brute-force/) | Validated (manual simulation) |
| T1547.001 Registry Run Keys | [_drafts/T1547.001-registry-run-key](_drafts/T1547.001-registry-run-key/) | Draft, untested |
| T1070.001 Clear Windows Event Logs | [_drafts/T1070.001-clear-event-logs](_drafts/T1070.001-clear-event-logs/) | Draft, untested |

**Validated** means the search returned the simulated activity in this lab. **Draft** means written but not yet run.

Saved-alert settings for all detections are in [`ALERTS.md`](ALERTS.md).

## Testing the Sigma rules

**What has been checked:** all six rules (four validated, two draft) were converted to Splunk syntax with sigma-cli 3.1.0 using the `splunk` backend and the `splunk_windows` pipeline, and every one converted without errors. That confirms the rules are well-formed.

**What has not been checked:** the converted queries have not been run in the lab Splunk, so it is not yet confirmed that they return the same events as the hand-written `detection.spl` searches. To do that:

```bash
pip install sigma-cli pysigma-backend-splunk pysigma-pipeline-sysmon pysigma-pipeline-windows
sigma convert -t splunk -p splunk_windows detections/T1136.001-create-account/rule.yml
```

The converter output has no index names, so add `index=sysmon` (or `index=wineventlog`) yourself, run it over the test window, and compare the results with `detection.spl`. The T1110 rule uses Sigma's correlation format and converts to a `bin`/`stats` query on `Computer`, so its field name may need adjusting to match your data.
