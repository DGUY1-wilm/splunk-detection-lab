# Lab setup guide

This is the build order that works, with the fixes from [`troubleshooting.md`](troubleshooting.md) already applied. Download pages and version numbers change, so always take installers from the vendor's official site.

Where a step has a matching screenshot, the filename is shown. Put the image in this folder.

## 0. Requirements

- A host with virtualization (VT-x or AMD-V) enabled in BIOS
- About 16 GB RAM and 100 GB free disk (8 GB RAM works if you run only the two VMs)
- Oracle VirtualBox
- Ubuntu Desktop LTS ISO and a Windows evaluation ISO
- A free splunk.com account

## 1. Ubuntu VM (Splunk server)

1. In VirtualBox choose **New**, select the Ubuntu ISO, and **check "Skip Unattended Installation"** (see troubleshooting #1).
2. Hardware: 4 to 6 GB RAM, 2 CPUs, 40 GB dynamically allocated disk.
3. Before first boot, open **Settings**:
   - **Display:** Video Memory 128 MB. If you see `vmwgfx` errors or a black screen, set the graphics controller to **VBoxSVGA** and leave 3D acceleration off.
   - **Network** (switch to **Expert** mode to see all adapters): Adapter 1 = **NAT**, Adapter 2 = **Host-only Adapter**.
4. Run the normal Ubuntu installer by hand and restart.
5. Optional but useful:
   ```bash
   sudo apt update
   sudo apt install -y openssh-server virtualbox-guest-utils virtualbox-guest-x11
   sudo reboot
   ```
6. Run `ip a` and note the **host-only address** (this lab used `192.168.56.101` on `enp0s8`). Ignore the NAT address (`10.0.2.15`).

## 2. Install Splunk Enterprise (Ubuntu VM)

1. Log into splunk.com, open the Splunk Enterprise download page, choose **Linux, .deb**, and download it (about 1.3 GB).
2. Install:
   ```bash
   cd ~/Downloads
   sudo dpkg -i splunk-*.deb
   ```
3. Splunk 10.x refuses to run as root. Give your user ownership and start it without `sudo` (see troubleshooting #5):
   ```bash
   sudo chown -R splunk:splunk /opt/splunk
   /opt/splunk/bin/splunk start --accept-license
   ```
   Create the Splunk admin account when prompted and store the password somewhere safe.
4. Start on boot, and open the firewall if `ufw` is active:
   ```bash
   sudo /opt/splunk/bin/splunk enable boot-start -user splunk
   sudo ufw status
   sudo ufw allow 8000/tcp
   sudo ufw allow 9997/tcp
   ```
5. Open `http://localhost:8000` (or `http://<host-only-ip>:8000` from the host) and log in.

The Enterprise trial license lasts 60 days from install and indexes up to 500 MB/day. Afterward you can switch to the Free license, which drops some features, including scheduled alerting. Capture alert screenshots inside the trial window. Check **Settings > Licensing** for the exact expiry.

## 3. Configure Splunk to receive data

In Splunk Web:

1. **Settings > Indexes > New Index**: create `sysmon` and `wineventlog`.
2. **Settings > Forwarding and receiving > Configure receiving > New Receiving Port**: `9997`.
3. **Apps > Find More Apps**: install the **Splunk Add-on for Sysmon** and **Splunk Add-on for Microsoft Windows** so fields parse (Image, CommandLine, ParentImage, and so on).

## 4. Windows VM (victim)

1. Create the VM with **"Skip Unattended Installation"** checked: 4 GB RAM, 2 CPUs, 60 GB disk.
2. **Settings:** Display = VBoxSVGA with 128 MB video memory. Network (Expert mode) = Adapter 1 NAT, Adapter 2 Host-only (same host-only network as the Ubuntu VM).
3. Install Windows by hand. If asked for a product key choose **I don't have a product key**. At the "work or school" sign-in screen use **Sign-in options > Domain join instead** to create a **local account** (see troubleshooting #4).
4. Install VirtualBox Guest Additions (**Devices > Insert Guest Additions CD image**), reboot, and take a snapshot named `clean-install`.
5. Confirm the clock and time zone are right:
   ```powershell
   Set-TimeZone -Id "Eastern Standard Time"
   w32tm /resync /force
   ```
6. Verify connectivity to the Splunk server:
   ```powershell
   ping 192.168.56.101
   ```

## 5. Sysmon

Admin PowerShell on the Windows VM:

```powershell
mkdir C:\Sysmon
cd C:\Sysmon
Invoke-WebRequest -Uri "https://download.sysinternals.com/files/Sysmon.zip" -OutFile "C:\Sysmon\Sysmon.zip"
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml" -OutFile "C:\Sysmon\sysmonconfig-export.xml"
Expand-Archive -Path "C:\Sysmon\Sysmon.zip" -DestinationPath "C:\Sysmon" -Force
.\Sysmon64.exe -accepteula -i sysmonconfig-export.xml
Get-Service Sysmon64
```

## 6. Extra Windows logging

Run `gpedit.msc` (Pro or Enterprise editions only) and enable:

- **Computer Configuration > Administrative Templates > Windows Components > Windows PowerShell > Turn on PowerShell Script Block Logging**
- **Computer Configuration > Administrative Templates > System > Audit Process Creation > Include command line in process creation events**

Then run `gpupdate /force`.

## 7. Universal Forwarder

1. Download the **Windows 64-bit .msi** Universal Forwarder from splunk.com. If the Windows VM's browser cannot sign in (clock error), copy the wget link from your host's browser and download with `curl.exe -L -o <path> "<url>"`.
2. In the installer: accept the license, choose **on-premises Splunk Enterprise**, create a forwarder admin account, **leave the deployment server blank**, and set the receiving indexer to `192.168.56.101:9997`.
3. Create [`inputs.conf`](inputs.conf) at `C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf`. If you cannot paste into the VM, open Notepad **as administrator** and type it, saving as `inputs.conf` with type **All files**.
4. **Run the forwarder service as Local System.** The default virtual account cannot read the Sysmon log (see troubleshooting #9):
   ```powershell
   sc.exe config SplunkForwarder obj= LocalSystem
   Restart-Service SplunkForwarder
   Test-NetConnection 192.168.56.101 -Port 9997
   ```

## 8. Verify the pipeline

Run `whoami` and `ipconfig` on the Windows VM, wait a minute, then in Splunk:

```
index=sysmon | stats count by EventCode
index=sysmon EventCode=1 Image="*whoami.exe" | table _time host User ParentImage CommandLine
```

![Sysmon whoami events in Splunk](01-sysmon-whoami-logs-verified.png)

![Sysmon event counts by EventCode](02-sysmon-eventcode-counts.png)

Snapshot both VMs as `logging-working`.

## 9. Atomic Red Team (Windows VM, admin PowerShell)

```powershell
Add-MpPreference -ExclusionPath "C:\AtomicRedTeam"
Set-ExecutionPolicy Bypass -Scope Process -Force
IEX (IWR 'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing)
Install-AtomicRedTeam -getAtomics -Force
Import-Module "C:\AtomicRedTeam\invoke-atomicredteam\Invoke-AtomicRedTeam.psd1" -Force
```

Note the **hyphen** in `Install-AtomicRedTeam`. Re-run the `Import-Module` line in every new PowerShell window. Snapshot as `atomic-ready`, and revert to it between tests.

## 10. The attack, detect, tune loop

For each technique:

1. **Windows VM:** run `Invoke-AtomicTest <ID> -ShowDetails`, then the test, noting the time.
2. **Splunk:** find the evidence and write the search.
3. **Splunk:** save it as a scheduled alert (see [`../detections/ALERTS.md`](../detections/ALERTS.md)).
4. **Windows VM:** clean up (`-Cleanup`) or revert the snapshot.
5. **Repo:** document the technique, evidence, false positives, and limitations.
