# TimerResolution-Optimization

Configure and apply a custom **system timer resolution** via `SetTimerResolution.exe` to **drastically reduce** DPC latency and stabilize performance.  
**Videos:** [▶ 1](https://youtu.be/xfWaux1z2Yk) / [▶ 2](https://youtu.be/DhtpBtnFoCI) &nbsp;|&nbsp; **Source:** [Microsoft Learn – Timer Resolution](https://learn.microsoft.com/windows/win32/api/timeapi/nf-timeapi-timebeginperiod)

<details>
  <summary>▶ Click to show script preview</summary>

![preview1](https://i.imgur.com/wK7qsnW.png)

</details>

<details>
  <summary>▶ Click to show Before / After results</summary>

**❌ Off — Default Windows (15.6250ms)**
![off](https://i.imgur.com/JcPjqDy.png)

**🟡 GlobalTimerResolutionRequests only (1.0000ms)**
![with-key](https://i.imgur.com/rvklGn6.png)

**✅ SetTimerResolution + GlobalTimerResolutionRequests (0.5060ms)**
![active](https://i.imgur.com/HXwq0XC.png)

</details>

---

## What is Timer Resolution?

> By default, Windows runs at **~15.6ms** — the system clock refreshes every 15.6ms. Lowering this improves precision and reduces latency, which matters in competitive gaming where every millisecond counts.

---

## Folder structure

Place the folder in your Downloads and make sure it looks like this before launching `SetTimer.ps1`:

```
../Downloads/Timer Resolution/
├── Content/
│   ├── SetTimerResolution.exe
│   └── Windows10Fix/
│       └── Dpclat.exe
├── MeasureSleep.exe
├── SetTimer.ps1
└── README.md
```

> [!CAUTION]
> Do not move or rename any files — the script uses relative paths.

---

## `1` Apply the registry key

```powershell
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Session Manager" /v "GlobalTimerResolutionRequests" /t REG_DWORD /d 1 /f
```

> [!IMPORTANT]
> A **restart is required** after applying this key.

---

## `2` Choose your resolution

Start with **0.5200ms** (option 2) — most stable for most systems. There is no universal best value, it depends on your hardware.

| Option | Value | Info |
|---|---|---|
| **`1`** | Custom | Any value between `5000` and `5200` |
| **`2`** | `0.5200ms` | ⭐ Recommended |
| **`3`** | `0.5120ms` | Stable |
| **`4`** | `0.5086ms` | Most optimal (if stable) |
| **`5`** | `0.5060ms` | Perfect (if stable) |
| **`6`** | `0.5000ms` | Experimental |

> [!NOTE]
> Values are in **100-nanosecond units**. Example: `5120` = `0.5120ms`.

---

## `3` Validate with MeasureSleep.exe

Launches automatically after applying a resolution. It measures how long Windows actually sleeps when asked to sleep 1ms — use it to confirm your resolution is active and stable.

✅ **Good** — low and consistent delta (`~0.01` to `0.02ms`):
```
Resolution: 0.5086ms, Sleep(1) slept 1.0180ms (delta: 0.0180)
Resolution: 0.5086ms, Sleep(1) slept 1.0148ms (delta: 0.0148)
```

❌ **Bad** — delta spiking (`0.47ms+`), sign of a system interrupt:
```
Resolution: 0.5086ms, Sleep(1) slept 1.4752ms (delta: 0.4752)
```

> Occasional spikes are normal. Frequent ones signal an issue — see **Reduce DPC Spikes** below.

---

## Fix — Windows 10

Windows 10 reverts to default resolution when no process actively holds a high-precision request. Keep both `SetTimerResolution.exe` and `dpclat.exe` running in the background at all times.

> [!IMPORTANT]
> [DPC Latency Checker](https://www.thesycon.de/eng/latency_check.shtml) must stay open to maintain the resolution on Windows 10.

---

## Reduce DPC Latency Spikes

| Cause | Fix |
|---|---|
| **Power Plan** | Use High / Ultimate Performance with proper C-States |
| **Wi-Fi & Bluetooth** | Disable in Device Manager if using Ethernet |
| **Audio overlays** | Remove Nahimic, Sonic Studio, Realtek Audio Console |
| **GPU & Chipset drivers** | Clean install (DDU recommended) |
| **Background software** | Disable iCUE, Synapse, Armoury Crate and antivirus real-time scan |

---

<p align="center">
  <sub>©insopti — <a href="https://guns.lol/inso.vs">guns.lol/inso.vs</a> | For personal use only.</sub>
</p>

