---
sidebar_position: 1
---

# Basic Shared Pc configuration in Intune (Part 1)

Fact : The current situation is that all PCs used by multiple users have a software called DeepFreeze. This software allows for the redirection of everything writen to the local drive (app installations, settings changes, saved data) to a virtual overlay. This overlay is temporary and it is cleared when a reboot is initiated. A maintenance schedule is between 0001 and 0600 everyday. This allows to run Windows update and other update if planned and pushed to the devices in question. (Not automaticaly)

Problem : This software is hosted localy and seems to have intermittent issues. Also, the additional complaints are : problems updating applications via Intune (no schedule), the server freezes often and problems going back in hibernation while Windows update is in the works.

Hypothese : To find a new solution with the tools that are given, the approach will be to utilize Intune, right click tools and Windows 10/11 features as an alternative to Deepfreeze.

<!-- truncate -->

# Plan :

##   Policy in Intune
        - The Shared Pc policy in Intune allows for a few configurations

            ![SharedPCPolicy](Shared-PC-Policy.png)

        - First option is to enable Shared PC mode
        - Second, you can use a guess, a domain account or a combination of both. For this configuration, domain only.
        - Third, Account management allows the deletion of accounts at sign-off time and during system maintenance time periods. And this scenario, it will not be configured.
        - Fourth, Local storage. With local storage, users can save and view files on the device's hard drive. ** *From previous tests, the Restrict Local Storage in the Shared Pc policy does not provide proper protection of drives (It removes the C: drive from the file explorer but doesn't block commands via Terminal or Powershell)* **. To remediate this issue, Unified Write Filter (UWF) will be use (Windows feature).
        - Fifth, Power Policies. Power policies prevent users from changing power settings, turns off hibernate, and overrides all power state transitions—such as lid close—to sleep. This will be enable since we want, for theses tests, to have the devices working non stop.
        - Sixth, Spleep time out (in seconds). Self explanatory. We will keep the default (60 minutes).
        - Seventh, Sign-in when PC wakes... yes.
        - Eighth, Maintenance start time. Let's put it for 0001 everyday. So this case, this equals 1.
            The goal : See if applications, when needed, will be updated during this time frame.
        - Ninth, Education policies. A series of configurations that will not be used in this setup.

###     Here's what it looks like :
            ![SharedPCConfigured](SharedPCConfigured.png)

##  Groups in Intune
        - I'll create 4 groups for 4 machines. 
            - First group will have UWF managed and configured by CSP in Intune and have the SharedPC policy applied.
            - Second group will have UWF installed and configured locally and have the SharedPC policy applied.
            - Third group will have UWF managed and configured by CSP in Intune and not have the SharePC policy applied.
            - Fouth group will have UWF installed and configured locally and not have the SharedPC policy applied.
            ![Groupsexample](Groups.png)

## Device configuration in Intune CSP
        - It seems to be able to managed UWF with Intune, you need to create a custom CSP with Multiple OMA-URI settings
        
            ![CSPUWFCONFIG](CSPUWFCONG.png)

        - Right from the start, there's an error in the configuration :

            ![UWFError](UWFerror.png)

        - Doesn't seem to work or it's a management/supervision type of service via Intune

        - Ok, I'll take a new approach. Let's make a PS Script :

            ```jsx title="UWFsetupIntune.ps1"
                # Enable Unified Write Filter (UWF)
                uwfmgr filter enable

                # Protect the C: Drive
                uwfmgr volume protect C:

                # Set Overlay Size (2GB)
                uwfmgr overlay set-size 2048

                # Enable UWF Servicing Mode
                uwfmgr servicing enable

                # Set Daily Servicing Start Time (8:00 PM UTC = 3:00 PM ET)
                $servicingStartTime = "20:00:00"
                $servicingDuration = 300 # 5 hours in minutes

                # Apply Servicing Schedule
                reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\UWF\Servicing" /v StartTime /t REG_SZ /d $servicingStartTime /f
                reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\UWF\Servicing" /v Duration /t REG_DWORD /d $servicingDuration /f

                # File Exclusions (Ensure logs and Intune-related files are not wiped on reboot)
                uwfmgr file add-exclusion C:\Windows\Logs
                uwfmgr file add-exclusion C:\ProgramData\Microsoft\Intune

                # Apply changes and restart to enable UWF
                Restart-Computer -Force

            ```
        
        - Well this approach might work but it seems that we need to enable the feature prior to executing the script. Here's a new version :

            ```jsx title="UWFsetupIntune2.ps1"
                # Logging Function
                $logFile = "C:\Windows\Temp\UWF_Setup.log"
                function Write-Log {
                    param ([string]$message)
                    $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
                    Add-Content -Path $logFile -Value "$timestamp - $message"
                }

                Write-Log "Starting Unified Write Filter (UWF) Setup..."

                # Enable Unified Write Filter Feature (if not already installed)
                $uwfFeature = Get-WindowsOptionalFeature -Online -FeatureName "Client-UnifiedWriteFilter"
                if ($uwfFeature.State -ne "Enabled") {
                    Write-Log "Enabling UWF Feature..."
                    try {
                        Enable-WindowsOptionalFeature -Online -FeatureName "Client-UnifiedWriteFilter" -All -NoRestart -ErrorAction Continue
                        Write-Log "UWF Feature enabled successfully."
                    } catch {
                        Write-Log "Failed to enable UWF Feature: $_"
                        exit 1
                    }
                } else {
                    Write-Log "UWF Feature is already enabled."
                }

                # Enable Unified Write Filter (UWF)
                Write-Log "Enabling UWF..."
                uwfmgr filter enable

                # Protect the C: Drive
                Write-Log "Protecting C: drive..."
                uwfmgr volume protect C:

                # Set Overlay Size (2GB)
                Write-Log "Setting overlay size to 2GB..."
                uwfmgr overlay set-size 2048

                # Enable UWF Servicing Mode
                Write-Log "Enabling servicing mode..."
                uwfmgr servicing enable

                # Set Daily Servicing Start Time (3:00 PM ET = 8:00 PM UTC)
                $servicingStartTime = "20:00:00"
                $servicingDuration = 300 # 5 hours in minutes

                Write-Log "Applying servicing schedule..."
                reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\UWF\Servicing" /v StartTime /t REG_SZ /d $servicingStartTime /f
                reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\UWF\Servicing" /v Duration /t REG_DWORD /d $servicingDuration /f

                # File Exclusions (Ensure logs and Intune-related files are not wiped on reboot)
                Write-Log "Adding file exclusions..."
                uwfmgr file add-exclusion C:\Windows\Logs
                uwfmgr file add-exclusion C:\ProgramData\Microsoft\Intune
                uwfmgr file add-exclusion C:\Windows\Temp

                # Apply changes and restart to enable UWF
                Write-Log "Restarting system to apply UWF settings..."
                Restart-Computer -Force

            ```
        - Roughly... I want the Windows feature enable, protect the C: drive with some exclusions and do maintenance (servicing) starting at 1500 and for 5 hours. Every step loged in Windows\Temp
        - It works!!! As soon as the device sync's with Intune, the script is applied. Note **When working with scripts and Intune, if you have an error, you just have to upload a different version of the script and sync to retry.

