# Troubleshooting log

Problems hit while building this lab, written as symptom, cause, fix. Screenshots are optional and live in [`troubleshooting/`](troubleshooting/).

| # | Area | Problem |
|---|---|---|
| 1 | VirtualBox | Ubuntu unattended install fails |
| 2 | VirtualBox | Installer window cut off, no Install button |
| 3 | VirtualBox | `vmwgfx` graphics errors at boot |
| 4 | Windows | Product key / forced Microsoft account during setup |
| 5 | Splunk | Splunk 10 refuses to run as root |
| 6 | Terminal | Garbled paste (`^[[200~`) |
| 7 | Networking | No "Adapter 2" in VM settings |
| 8 | Windows | Splunk login fails with a clock error |
| 9 | Splunk | Sysmon index empty while other logs arrive |
| 10 | Splunk | Windows host appears under an unexpected name |
| 11 | Splunk | Blank account-name fields in 4720 events |
| 12 | PowerShell | Typos when typing commands by hand |
| 13 | Atomic Red Team | T1110 tests do not fit a standalone VM |
| 14 | Time | 4-hour difference between Windows and Splunk |

---

## 1. Ubuntu unattended install fails

**Symptom:** "Installation failed. A command in the autoinstall file failed to run." Technical details point at `/root/vboxpostinstall.sh`.

**Cause:** VirtualBox's Unattended Installation feature injects a post-install script (for Guest Additions) that failed on this Ubuntu release. Ubuntu itself was likely fine, but the failed script marks the whole install as failed.

**Fix:** Delete the VM and recreate it with **Skip Unattended Installation** checked, then run the normal interactive installer. Install Guest Additions manually afterward.


## 2. Installer window cut off

**Symptom:** The Ubuntu installer's Install button is below the bottom edge of the VM screen.

**Cause:** The installer window is larger than the VM's default display.

**Fix:** **View > Virtual Screen 1 > Resize** in VirtualBox, change the resolution in Ubuntu's display settings, or drag the window up with Alt. Raising Video Memory to 128 MB in VM settings also helps.

## 3. `vmwgfx` errors at boot

**Symptom:** Console shows `vmwgfx seems to be running on an unsupported hypervisor` and "Please switch to a supported graphics device."

**Cause:** Mismatch between the VMSVGA graphics controller and the Linux `vmwgfx` driver under VirtualBox. Often a harmless warning.

**Fix:** Wait a minute. If boot hangs, power off and change **Settings > Display > Graphics Controller** to **VBoxSVGA**, with 3D acceleration off.

## 4. Windows setup: product key and Microsoft account prompts

**Symptom:** The installer reports it cannot see the product key. Later, setup demands a Microsoft account ("Let's set things up for your work or school").

**Cause:** Unattended install generates an answer file with an empty product key. Windows setup also pushes online accounts by default.

**Fix:** Recreate the VM with **Skip Unattended Installation**, choose **I don't have a product key** if asked, and at the sign-in screen use **Sign-in options > Domain join instead** to create a local account. Fallbacks: disconnect the network adapter during setup, or press Shift+F10 and run `start ms-cxh:localonly`.

## 5. Splunk 10 refuses to run as root

**Symptom:** `Running Splunk Enterprise as root is deprecated and will be removed in a future release.` Splunk does not start.

**Cause:** Starting it with `sudo` runs it as root, which Splunk 10.x blocks by default.

**Fix:** Give the Splunk folder to a normal user and start without sudo:

```bash
sudo chown -R splunk:splunk /opt/splunk
/opt/splunk/bin/splunk start --accept-license
sudo /opt/splunk/bin/splunk enable boot-start -user splunk
```

Spell the flag `--accept-license`. A mistyped `--accept-licence` printed `Bad Option` in the startup output.


## 6. Garbled paste in the terminal

**Symptom:** A pasted `wget` command ran as `^[[200~wget ...~` and failed with "command not found."

**Cause:** Terminal bracketed-paste escape characters were pasted along with the text.

**Fix:** Paste with Ctrl+Shift+V, type the command prefix by hand and paste only the URL, or skip `wget` and download through the browser.

## 7. No "Adapter 2" in VM settings

**Symptom:** VirtualBox's Network settings show only one adapter.

**Cause:** Newer VirtualBox opens Settings in Basic mode, which hides extra adapters. The VM must also be powered off.

**Fix:** Power the VM off, switch Settings to **Expert** mode, and enable Adapter 2 as **Host-only**.

## 8. Splunk login fails on the Windows VM

**Symptom:** "Unable to sign in. Please check that your device's clock is accurate and try again" on the Universal Forwarder download page.

**Cause:** Clock and time zone drift on the VM (common after snapshot restores) breaks time-sensitive sign-in tokens.

**Fix:** Set the time zone and resync (`Set-TimeZone -Id "Eastern Standard Time"`, `w32tm /resync /force`). Workaround: copy the wget link from the host's logged-in browser and download on the VM with `curl.exe -L -o <path> "<url>"`.

## 9. Sysmon index empty while other logs arrive

**Symptom:** `index=wineventlog` held about 1,700 events from the Windows VM, but `index=sysmon` returned 0 events, even though Sysmon was running and writing events locally.

**Cause:** The forwarder service was running as `NT SERVICE\SplunkForwarder`, a limited virtual account. It could read the Security and PowerShell logs but not the Sysmon operational log.

**Diagnosis:**

```powershell
Get-CimInstance Win32_Service -Filter "Name='SplunkForwarder'" | Select-Object Name, StartName
```

**Fix:** Run the service as Local System, then restart it:

```powershell
sc.exe config SplunkForwarder obj= LocalSystem
Restart-Service SplunkForwarder
```

After the change, Sysmon events began arriving (1,584 events across 12 EventCodes in the first check). Keep the space after `obj=`.

Related note: Sysmon's own log existing locally (`Get-WinEvent -ListLog "*Sysmon*"` showed events) was what proved the problem was on the forwarder side, not in Sysmon.


## 10. Windows host appears under an unexpected name

**Symptom:** Searching `host=SplunkWin` returned nothing.

**Cause:** `SplunkWin` was only a default from VirtualBox's unattended-install wizard, which was skipped. The machine's real name was `DESKTOP-KF26PU8`.

**Fix:** Find the real host name with `index=_internal | stats count by host`, and use it (or filter by index) in searches.


## 11. Blank account-name fields in 4720 events

**Symptom:** A search for Security event 4720 showed events, but `TargetUserName` and `SubjectUserName` were empty.

**Cause:** In the plain-text event format used here, the creator and new-account names are two values of one multivalue field, `Account_Name`.

**Fix:**

```
index=wineventlog EventCode=4720
| eval creator=mvindex(Account_Name,0), new_account=mvindex(Account_Name,1)
| table _time host creator new_account
```


## 12. Typos when typing commands by hand

**Symptoms and causes** (the VM clipboard was not available, so commands were typed):

- `Install_AtomicRedTeam` is not recognized: the command uses a **hyphen**, not an underscore.
- `Microsoft-Windows_Sysmon/Operational` log not found: also a hyphen.
- `Variable reference is not valid. ':' was not followed by a valid variable name character`: a colon right after `$_` inside a string (`"BadPass$_:"`). Remove the colon, or write `${_}`.
- The prompt drops to a `>>` continuation line: an unclosed brace or quote. Press Ctrl+C and retype.

**Fix:** Use Tab completion for cmdlet names, and enable **Devices > Shared Clipboard > Bidirectional** (needs Guest Additions).

## 13. T1110 Atomic tests do not fit a standalone VM

**Symptom:** `Invoke-AtomicTest T1110` fails, and the T1110.001 tests describe attacking an Active Directory domain.

**Cause:** T1110 is a parent technique with no atomics of its own. The T1110.001 tests brute force domain users over SMB or LDAP, which needs a domain controller.

**Fix:** Run a manual simulation of password guessing against the local machine instead:

```powershell
1..6 | ForEach-Object { net use \\127.0.0.1\IPC$ /user:labfake BadPass$_ }
```

The README for that detection states plainly that it is a manual simulation.

![Atomic tests target AD](.../.../screenshots/Detection%25206/01-atomic-ad-prereq-note.png)

## 14. Timestamps differ by 4 hours

**Symptom:** A PowerShell script block event shows `10/10/2026 11:07:54 PM` in the raw text while Splunk's time column shows `2026-10-11 03:07:54`.

**Cause:** Most likely a time zone display difference: Windows records local time (US Eastern, UTC-4 at the time) and Splunk displays UTC. The 4-hour gap matches that.

**Fix:** Nothing is broken. Note the offset in write-ups. Set a consistent time zone under the Splunk user's preferences if you want them to match. Some early events also carry odd timestamps from before the clock and time zone were corrected during setup.
