# Publishing guide

## Before you push

1. Add your screenshots using the names in [`screenshot-checklist.md`](screenshot-checklist.md). Remove README image links for any you skip.
2. Resolve every `<!-- TO CAPTURE -->`, `<!-- RECAPTURE -->`, `<!-- TODO -->`, and `<!-- VERIFY -->` comment (search for `<!--`).
3. Scan for anything sensitive:
   ```bash
   git grep -niE "password|passwd|secret|token|hash"
   ```
   Matches in the docs describing lab steps are fine. Real credentials are not.
4. Make sure no VM images or installers are in the folder (the `.gitignore` blocks common ones).

## Create the repo and push

```bash
cd splunk-detection-lab
git init
git branch -M main
```

Suggested commit sequence, so the history tells the story:

```bash
git add README.md LICENSE .gitignore
git commit -m "chore: add README, license, and gitignore"

git add setup docs/architecture.md
git commit -m "docs(setup): lab build guide, forwarder config, troubleshooting log"

git add detections/README.md detections/ALERTS.md detections/T1136.001-create-account
git commit -m "feat(detection): T1136.001 local account creation"

git add detections/T1059.001-encoded-powershell
git commit -m "feat(detection): T1059.001 encoded PowerShell"

git add detections/T1053.005-scheduled-task
git commit -m "feat(detection): T1053.005 scheduled task creation"

git add detections/T1110-brute-force
git commit -m "feat(detection): T1110.001 failed logon burst (manual simulation)"

git add detections/_drafts
git commit -m "docs(detection): add untested draft detections"

git add dashboards incident-report docs
git commit -m "docs: dashboard, incident report, lessons learned"
```

Then create an empty repo named `splunk-detection-lab` on GitHub (no README, license, or .gitignore, since you already have them) and push:

```bash
git remote add origin https://github.com/DGUY1-wilm/splunk-detection-lab.git
git push -u origin main
```

## GitHub settings

- **Description:** `Home SOC lab: Splunk detections for Windows Sysmon telemetry, mapped to MITRE ATT&CK`
- **Topics:** `splunk`, `sysmon`, `detection-engineering`, `mitre-attack`, `sigma`, `soc`, `blue-team`, `atomic-red-team`, `threat-detection`
- **Pin it:** go to your profile, click **Customize your pins**, and select the repo (you have a free sixth slot).

## Resume bullets (adjust the numbers to match what you finish)

Pick the one that is true when you apply:

- "Built a Splunk detection lab (Sysmon, Windows Event Logs, Universal Forwarder) and authored 4 detections mapped to MITRE ATT&CK, each documented with evidence, false positives, and evasion limits."
- "Troubleshot a Splunk forwarding failure (limited service account unable to read Sysmon logs) and tuned a detection after finding a false positive in my own data."
- "Wrote detections in SPL and as Sigma rules for local account creation, encoded PowerShell, scheduled tasks, and brute-force activity."

Do not claim alerts, dashboards, or the draft detections on a resume until they are saved, imported, and tested.

## Project blurb for LinkedIn or your GitHub profile

"Splunk Detection Engineering Lab: a home SOC lab that ships Windows endpoint telemetry into Splunk, simulates attacker techniques, and documents detections mapped to MITRE ATT&CK, with evidence, false-positive analysis, and a troubleshooting log."
