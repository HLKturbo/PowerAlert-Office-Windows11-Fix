**PowerAlert Office installer (v20.2.1.942) fails on Windows 11: erroneously requires PowerShell 2.0**

Environment:
OS version: Windows 11 24H2
PowerAlert installer file name: poweralertoffice-20.2.1.942
PowerShell versions present (Windows PowerShell 5.1, PowerShell 7.x).

Symptoms (Expected vs Actual):
Expected: Installer proceeds or uses modern PowerShell.
Actual: Installer shows dialog “Windows PowerShell needs to be installed for this installation to continue.”

Steps to reproduce:
Download file poweralertoffice-20.2.1.942.exe from https://tripplite.eaton.com/products/power-alert-office-home-medical
Run on Windows 11 -> error dialog appears.
Extracted MSI via https://github.com/Bioruebe/UniExtract2 

Root cause diagnosis:
Installer performs a hard MSI pre-req check for PowerShell 2.0 (LaunchCondition / RegLocator for HKLM\SOFTWARE\Microsoft\PowerShell\1\PowerShellEngine\PowerShellVersion).
PS2.0 has been removed/disabled on modern Windows 11, so the check always fails even when PS 5.1/7.x are installed.

**Workarounds & fix:**
Extract MSI using https://github.com/Bioruebe/UniExtract2
  - Download App and run UniExtract.exe follow defaults
  - Open the .exe installer  poweralertoffice-20.2.1.942.exe leave destination directory as default then click ok
  - On next step select **InstallShield /b switch** as  Extraction Method
  - after waiting a few seconds the InstallShield Wizard will show "Unable to Save File **** The system cannot find the path specified" click on OK
  - Using File Explorer itself create folder c:\temp then on window that popped up in UniExtract2 Browse for Folder c:\temp then click ok (by this point you can close installer, the .msi is already extracted, copy and paste somewhere else just in case)
  - This will drop the .MSI file installer inside the temp folder

I used Orca from Microsoft's Software Development Kit to remove the LaunchCondition/AppSearch/RegLocator rows Attached is the .mst file that was created (which will basically bypass the check for powershell)

  - Drop the PowerAlertOffice.msi and also the BypassPS2.mst on c:\temp
  - Open CMD prompt as admin and run msiexec /i "C:\temp\PowerAlert Office.msi" TRANSFORMS="C:\temp\BypassPS2.mst" /l*v C:\Temp\pao-install.log
  - This will run the app using the .mst file bypassing checks for PowerShell and if there's any issues will save a log.
