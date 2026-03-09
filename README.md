# TimerResolution-Optimization

Configure and apply a custom **system timer resolution** via `SetTimerResolution.exe`. This allows reaching resolutions far below Windows' default to **drastically reduce** DPC latency and stabilize performance.  
**Videos :** [▶ Preview & Info 1](https://youtu.be/xfWaux1z2Yk) / [▶ 2](https://youtu.be/DhtpBtnFoCI)  
**Source:** [Microsoft Learn – Timer Resolution](https://learn.microsoft.com/windows/win32/api/timeapi/nf-timeapi-timebeginperiod)

<details>
  <summary>▶ Click to show script preview</summary>

![preview1](https://i.imgur.com/wK7qsnW.png)

</details>

---

### Before / After

<details>
  <summary>▶ Click to show Before / After results</summary>

**❌ Off — Default Windows (15.6250ms)**
![off](https://i.imgur.com/JcPjqDy.png)

**🟡 With GlobalTimerResolutionRequests only (1.0000ms)**
![with-key](https://i.imgur.com/rvklGn6.png)

**✅ SetTimerResolution + GlobalTimerResolutionRequests (0.5060ms)**
![active](https://i.imgur.com/HXwq0XC.png)

</details>

---

## What is Windows Timer Resolution?

> Timer resolution refers to the precision of the system clock under Windows. It defines the time intervals used by processes running on your PC, including games. By default, Windows typically runs at **~15.6 ms**, meaning the system clock refreshes approximately every 15.6 milliseconds. While sufficient for general use, this precision can be inadequate in a competitive context where every millisecond counts.

---

## Folder structure

Before launching `SetTimer.ps1`, make sure your folder looks like this and is located in your Downloads:

```
../Downloads/Timer Resolution/
├── Content/
│   ├── SetTimerResolution.exe
│   └── Windows10Fix/
│       └── Dpclat.exe
│
├── MeasureSleep.exe
├── SetTimer.ps1
└── README.md
```

> [!CAUTION]
> Do not move or rename any files. The script uses relative paths to locate executables.

The script automates everything: it applies the registry key, installs `SetTimerResolution.exe` as a scheduled task service at startup, and lets you choose and test your resolution value.

---

## `1` Apply the GlobalTimerResolutionRequests registry key

This key instructs Windows to force a minimum timer resolution of **1.000ms** globally, ensuring that the request from `SetTimerResolution.exe` is properly acknowledged by the system.

**Run in Command Prompt (cmd) as Administrator:**

```powershell
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Session Manager" /v "GlobalTimerResolutionRequests" /t REG_DWORD /d 1 /f
```

> [!IMPORTANT]
> A **restart is required** after applying the registry key for it to take effect.

---

## `2` Choose your resolution

After applying the key, launch the script and select a value.  
There is no universally optimal value — the result depends on each system.  
It is recommended to start with **0.5200ms** (option 2), the most stable value for the majority of configurations.


| Option | Value | Info |
|---|---|---|
| **`1`** | Custom | Any value between `5000` and `5200` |
| **`2`** | `0.5200ms` | ⭐ Recommended — most stable for most systems |
| **`3`** | `0.5120ms` | Stable for most systems |
| **`4`** | `0.5086ms` | Most optimal (if stable!) |
| **`5`** | `0.5060ms` | Perfect (if stable!) |
| **`6`** | `0.5000ms` | Experimental |

> [!NOTE]
> Values are expressed in **100-nanosecond units**. Example: `5120` = `0.5120ms`.

---

## `3` Measure and validate with MeasureSleep.exe

`MeasureSleep.exe` launches automatically after applying a resolution. It performs `Sleep(1)` in a loop — asking Windows to sleep exactly **1ms** and measuring how long Windows actually slept. This lets you verify whether your timer resolution is active and stable.

**Example output:**
```
Resolution: 0.5086ms, Sleep(1) slept 1.0180ms (delta: 0.0180)
```

| Field | Meaning |
|---|---|
| **`Resolution`** | The timer resolution currently active on the system |
| **`Sleep(1) slept`** | The actual time Windows slept (should be close to 1ms) |
| **`delta`** | The gap between the requested sleep (1ms) and the actual sleep |

✅ **Good** — Low and consistent delta, around `0.01` to `0.02ms`:
```
Resolution: 0.5086ms, Sleep(1) slept 1.0180ms (delta: 0.0180)
Resolution: 0.5086ms, Sleep(1) slept 1.0173ms (delta: 0.0173)
Resolution: 0.5086ms, Sleep(1) slept 1.0148ms (delta: 0.0148)
```
Timer stable — resolution correctly applied and active.

❌ **Bad** — Delta spiking abruptly, sign of a system interrupt:
```
Resolution: 0.5086ms, Sleep(1) slept 1.4752ms (delta: 0.4752)
```
A delta of `0.47ms+` indicates an interrupt. This can happen occasionally but should not be frequent.

> **Summary:** a healthy system will have very close and stable deltas. Large frequent spikes signal an issue. See the **Reduce DPC Spikes** section below to address this.

---

## Fix — Windows 10

Under **Windows 10**, the timer resolution requested via `SetTimerResolution` does not persist if no active process is actively maintaining a high-precision request (`NtSetTimerResolution`). Timer resolution under Windows works on a **request-based model**: as long as at least one process holds an active request, the resolution stays low. Once all requests are released, the system dynamically reverts to its default (~1.0ms or higher).

Keeping [DPC Latency Checker](https://www.thesycon.de/eng/latency_check.shtml) open helps maintain a low resolution, as the tool continuously requests high precision itself.

> [!IMPORTANT]
> Windows 10 users must keep both `SetTimerResolution.exe` and `dpclat.exe` running in the background to maintain the applied resolution.

---

## Reduce DPC Latency Spikes

**DPC (Deferred Procedure Calls) latency spikes** are one of the main causes of timer irregularities and micro-stutters in gaming. They result from excess kernel interrupts or poorly optimized drivers. Here is how to reduce them effectively:

**`POWER PLAN:`** Use an optimized or properly configured power plan (High Performance / Ultimate Performance, with appropriate C-States and CPU management). A poorly configured plan can increase scheduling latency and CPU state transitions.

**`WI-FI & BLUETOOTH ADAPTERS:`** If using a wired connection (Ethernet), disable **Wi-Fi and Bluetooth** in Device Manager. Wireless drivers generate hardware interrupts (ISR/DPC) in the background even without an active connection.

**`AUDIO SOFTWARE:`** Uninstall or disable additional audio overlays: **Nahimic, Sonic Studio, Realtek Audio Console/Manager**, etc. These kernel filters can cause significant DPC spikes.

**`GPU & CHIPSET DRIVERS:`** Keep your drivers up to date and prefer a **clean installation** (DDU recommended for GPU). Corrupted or outdated drivers generate excessive DPCs and abnormal interrupts.

**`ANTIVIRUS & BACKGROUND PROCESSES:`** Real-time scanning heavily stresses the kernel (I/O). RGB software and manufacturer utilities **(iCUE, Synapse, Armoury Crate)** are also known to cause DPC spikes through their services and drivers.

**Goal:** reduce ISR/DPC load, stabilize timer resolution and minimize system jitter to achieve consistent latency in competitive play.

---

<p align="center">
  <sub>©insopti — <a href="https://guns.lol/inso.vs">guns.lol/inso.vs</a> | For personal use only.</sub>
</p>
