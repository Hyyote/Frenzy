https://drive.google.com/file/d/1jazzneSkVz5B8JqquZKpuEyzNlSheeq2/view?usp=sharing

## Support Frenzy
If you find this useful, consider donating:
- **BTC:** `bc1qacd9jsje236mgphkpx0dycvv87kgem9dddxekg`
- **LTC:** `LTW5QwkKMCq6DThbWVo1YbayRZzP33gDsR`

# Frenzy

A stripped, optimized Windows 11 26H1 (build 28000.1575) installation built for competitive gaming and low-latency workloads. Frenzy removes over 1,500 unnecessary components at the image level, deploys custom kernel drivers for input latency and scheduling, and applies hardware-level tuning via MSR/MMIO writes on every boot — all without touching Driver Signature Enforcement or PatchGuard.<br>

There are lots of missing features that break programs and games with specific settings that do not work well on certain systems.

## What It is

### The Operating System

The base Windows image built through NTLite with an aggressive preset:

- **1,500+ components removed** — telemetry, Defender, Store, Xbox, Cortana, speech recognition, biometrics, printing, fax, Remote Desktop, Hyper-V, WSL, and hundreds more
- **2,000+ registry tweaks** baked directly into the offline image — service configurations, kernel parameters, scheduling, network stack tuning
- **147 services disabled at the image level** — only what's needed to boot and run games

### Kernel Drivers (KDU-Mapped)

Three custom kernel drivers are loaded at every boot via KDU (Kernel Driver Utility). They're manually mapped into kernel memory — no service entries, no files in System32, no PsLoadedModuleList footprint.

- **mousefix.sys** — legitimately signed, deployed as an auto-start kernel service for mouse input filtering
- **ProcessThrottleFix.sys** — walks the EPROCESS list on a polling timer, disabling Windows power throttling on target processes. No image load callbacks, no PatchGuard visibility
- **QuantumLock.sys** — sets the kernel quantum end timer to maximum, pushing DPC activity off Core 0 to keep it clean for mouse/input processing
- **SystemPriority.sys** — sets csrss.exe to Realtime priority and configures DWM thread priorities from kernel mode, bypassing PPL restrictions. Specifically targets the DWM Master Input Thread and Kernel Sensor Thread at Time-Critical (15)

### Hardware Tuning (RWEverything)

On first setup, an ApplyRWE script runs once to discover your hardware topology (xHCI controllers, GPU BAR addresses, PCH base) and generates a pure-cmd boot script. This script applies MSR and MMIO writes on every boot with zero dependencies — no PowerShell, no WMI, just Rw.exe talking directly to hardware.

**Intel systems** (15 categories enabled):
- xHCI IMOD interval zeroing (USB interrupt latency)
- SpeedStep, thermal throttling, and prefetcher disable (MSR 0x1A0)
- CPU package, core, iGPU, and DRAM power limit removal (MSR 0x610/0x638/0x640/0x618)
- Spectre/Meltdown mitigation disable (MSR 0x48/0x49)
- C1E auto-promotion disable (MSR 0x1FC)
- Constant TSC rate enforcement (MSR 0x19A)
- Memory scrambler disable (MSR 0x1A2)
- Performance energy bias to max performance (MSR 0x194)
- PCH deep C-state and power management disable
- PCIe completion timeout disable
- Memory controller refresh throttling and self-refresh disable
- GPU power gating, clock gating, DFS, GFXOFF, and DPM disable
- Display variable refresh and FIFO watermark tuning

**AMD systems** (5 categories enabled):
- xHCI IMOD interval zeroing
- Spectre/Meltdown mitigation disable
- GPU power gating, clock gating, DFS, GFXOFF, and DPM disable
- Display variable refresh and FIFO watermark tuning

### Device Manager

96 unnecessary devices are disabled after driver installation, including audio endpoints, WAN miniports, virtual adapters, WMI/ACPI bloat, Intel Management Engine, PCI controllers, Bluetooth, and unused system devices. Childless PCI-to-PCI bridges are pruned automatically. Audio enhancements are disabled on all render endpoints via registry.

### DWM Basic Themer (dwm-bs)

Runs silently at startup. Disables DWM non-client rendering, transitions, and Peek on every foreground window via EVENT_SYSTEM_FOREGROUND hooks. Reduces compositor overhead.

### Windows Settings

Privacy lockdown, desktop cleanup, and audio hardening applied during setup and after driver installation:

- All AI generation, passkey, screenshot, library access, speech recognition, activity history, feedback, and diagnostics features disabled
- Desktop icons hidden, Dynamic Lighting off, transparency off, Snap disabled
- Audio enhancements and exclusive mode disabled on all devices
- Startup sound disabled, visual effects set to best performance
- Page file disabled

### Open Shell

Installed silently at first logon with a custom theme and configuration. Replaces the Windows 11 Start menu.

## Install Flow

1. **Download the ISO, install it regularly**
2. **Automatic (SetupComplete)** — mousefix.sys deployed, KDU registered for boot, OpenShell staged, dwm-bs registered, Windows settings applied, vulnerable driver blocklist disabled
3. **Automatic (first logon)** — OpenShell installed with custom config
4. **Manual (Post-Driver)** — after installing GPU and NIC drivers, run PostDriverSetup.cmd: installs RWEverything, runs Device Manager Tweaks, discovers hardware and generates the boot .bat, registers it on the HKLM Run key
5. **Run FrenzyOptimizer** — apply your game-specific tuning
6. **Every boot** — KDU maps kernel drivers, ApplyRWE-Boot.bat writes MSRs/MMIO, dwm-bs hooks DWM

## Prerequisites

- Secure Boot: **OFF**
- Internet not required for installation
- NIC and GPU drivers and another browser if you don't like Brave portable

## Anti-Cheat Compatibility

- DSE (Driver Signature Enforcement) is fully enabled at all times — never patched
- No BCD flags set (testsigning and nointegritychecks are OFF)
- PatchGuard runs normally
- KDU-mapped drivers have no PsLoadedModuleList entry and no on-disk service registration
- KDU's vulnerable driver is only loaded momentarily during mapping and then unloaded
- Guarantees can't be made still
