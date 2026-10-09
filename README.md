# Windows 11 install (autounattend.xml)
 
Unattended Windows 11 install for preparing customer PCs. Finnish language and region, no user account during prep, automatic updates, and a handover script that resets the PC to the normal welcome screen (OOBE) for the customer.
 
Originally based on the [schneegans.de unattend generator](https://schneegans.de/windows/unattend-generator/), now maintained by hand. Do not regenerate it from the generator, as that would drop all changes.
 
## Usage
 
- **USB stick:** put `autounattend.xml` in the root of the Windows install stick.
- **VM:** attach `unattend.iso` as a second CD drive next to the Windows ISO.
- **Optional:** storage or Wi-Fi drivers (Intel VMD/RST, RAID, etc.) go in a `$WinPEDriver$` folder in the root of the stick.
## Flow
 
1. **Setup (WinPE)**
   - Choose the edition, then the target disk. USB disks are hidden.
   - The confirm screen shows the disk's existing volumes. Nothing is erased until you type `Y`.
   - The rest runs unattended.
2. **Prep (audit mode)**
   - Built-in admin, no user account.
   - Shop Wi-Fi profile added. Automatic BitLocker device encryption blocked.
   - Update window runs Windows Update (drivers included) until nothing is left:
     - restarts automatically as needed;
     - one confirmation restart, then a final check for background installs;
     - ends with a device driver check.
3. **Handover:** run `Finish Setup.cmd` on the desktop.
   - Checks: updates finished, activation, device problems.
   - Asks whether to allow device encryption (key goes to the customer's Microsoft account).
   - Cleans up and runs Sysprep to OOBE. The customer creates their own account.
   - For a local account in OOBE, keep the PC offline and choose "I don't have internet".
## Files on the installed PC
 
| Path | Purpose |
|---|---|
| `C:\Windows\Setup\Scripts\OEMSpecialize.ps1` / `.log` | Setup-time settings (Wi-Fi profile, BitLocker block, BypassNRO) |
| `C:\Windows\Setup\Scripts\OEMAudit.ps1` | Runs once at the first audit logon, starts the helpers |
| `C:\Windows\Setup\Scripts\OEMUpdate.ps1` | Update loop |
| `C:\Windows\Setup\Scripts\OEMUpdate.log` | Update log, with times |
| `C:\Windows\Setup\Scripts\OEMHideSysprep.ps1` | Closes the Sysprep window in audit mode |
| `C:\Windows\Setup\Scripts\OEMFinishChecks.ps1` | Checks run by Finish Setup |
| `C:\Users\Public\Desktop\Finish Setup.cmd` | Handover script, deletes itself |
 
Startup items during prep: `OEMUpdate.cmd` (until updates are done) and `OEMHideSysprep.lnk` (until Finish Setup).
 
## Notes
 
- No VBScript. Setup uses only cmd, DISM and diskpart.
- Windows Update gives no percentage progress, so elapsed timers are shown instead.
- The disk and edition lists parse DISM and diskpart output. A future ISO may need a tweak if that output changes.
## Status (9.10.2026)
 
- Tested in a VM: Home and Pro installs, audit mode, updates, Finish Setup to OOBE.
- Tested on real hardware: install, edition and disk screens, update loop.
- To verify on real hardware: final update check, hidden Sysprep window, Finish Setup, Wi-Fi-only PC, two-disk PC.
- Final version: own namespace (`urn:oem:install`), generator leftovers removed, OEM file names. No OEM support info.
