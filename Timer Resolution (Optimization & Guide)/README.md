# Content

```
SetTimer.ps1                 <- main tool
MeasureSleep.exe             <- measurement tool (optional but recommended)
Content\
  SetTimerResolution.exe     <- TimerResolution Executable
  Windows10Fix\
    Dpclat.exe               <- DPC fix for Windows 10 only
```

---

## Launch

1. **Right-click** on `SetTimer.ps1`
2. Click **"Run with PowerShell"** or **"Run as Administrator"**
3. If Windows blocks the script, open PowerShell as admin and type:
   ```powershell
   Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
   ```

---

## Which value to choose?

> ⚠️ There is no universal value. Every system reacts differently.

The ideal value is **the lowest possible without spikes**, measured with `MeasureSleep.exe`.
A **spike** = a latency peak caused by a driver, a background software, or an unstable hardware component.

**Recommended method:**
1. Apply `0.5200ms` first (option `2`) — most stable for the majority of systems
2. Watch `MeasureSleep.exe` which launches automatically
3. If measurements are stable and consistent -> try a lower value (option `1`, custom value)
4. If you see spikes -> go one step up and stay at that value

---

## Windows 10 Fix (option `0`)

On Windows 10, modifying the timer resolution requires `Dpclat.exe` to be running.
Without it, any resolution change will have no effect.
Option `0` installs it as a scheduled task so it runs automatically at every startup.

> This option is not needed on Windows 11.

---

## Reducing DPC spikes — general tips

  Wi-Fi / Bluetooth
    -> Disable in Device Manager if you are using a wired connection.

  Audio software (Nahimic, Sonic Studio, Realtek HD Manager)
    -> Disable or completely uninstall them.

  Real-time antivirus (Windows Defender, Malwarebytes, Avast, Norton...)
    -> Antivirus software generates constant kernel interrupts that cause spikes.
       Disable real-time protection or exclude the relevant folders.

  Outdated GPU / chipset drivers
    -> Update them, do a clean install using DDU.

  Windows power plan
    -> Switch to High Performance or Ultimate Performance.
       (Control Panel -> Power Options)

---

## Uninstall (option `9`)

The script removes:
- The scheduled task `SetTimerResolution`
- The scheduled task `DpcLatFix` (if installed)
- The folder `C:\Program Files\TimerResolutionFix\` and its contents

> ⚠️ The registry key `GlobalTimerResolutionRequests` is kept intentionally.
> Without an active process, it is harmless and allows Windows to honor resolution requests from other applications if needed.

---

## Status displayed at launch

```
[ GlobalTimerResoKey ]    Active / Not applied
[ SetTimerResolution ]    Running (0.5200ms) / Not running
[ Task Scheduler STR ]    Active / Not configured
[    Fix (Win 10)    ]    Active + Running / Configured / Not configured
```

  [ GlobalTimerResoKey ]
    Active       -> Registry key is in place, Windows accepts custom resolution requests.
    Not applied  -> Key is missing, apply a resolution to create it.

  [ SetTimerResolution ]
    Running (Xms) -> The process is actively running with the displayed resolution.
    Not running   -> No custom resolution is active, Windows runs at 15.6ms by default.

  [ Task Scheduler STR ]
    Active         -> A scheduled task will automatically restart the process at every boot.
    Not configured -> No persistence, the setting will be lost on next reboot.

  [ Fix (Win 10) ]
    Active + Running -> Dpclat is currently running and the scheduled task is in place.
    Configured       -> The task is installed, Dpclat will start on next boot.
    Not configured   -> The DPC fix is not installed.

---

## Contact & support

- Social  : (https://guns.lol/inso.vs)
- Discord : (https://discord.gg/fayeECjdtb)
- GitHub  : (https://github.com/insovs)

> This script is for personal use only. Any modification, copy or redistribution is prohibited.
