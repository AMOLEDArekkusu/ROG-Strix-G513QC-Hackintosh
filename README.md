<div align="center">

# ASUS ROG Strix G15 G513QC — OpenCore EFI

**macOS Sequoia on AMD Ryzen 7 5800H + Radeon iGPU**

![macOS](https://img.shields.io/badge/macOS-Sequoia_15.8.1-000000?style=for-the-badge&logo=apple&logoColor=white)
![OpenCore](https://img.shields.io/badge/OpenCore-1.0.9-0A84FF?style=for-the-badge)
![CPU](https://img.shields.io/badge/CPU-Ryzen_7_5800H-ED1C24?style=for-the-badge&logo=amd&logoColor=white)
![GPU](https://img.shields.io/badge/iGPU-NootedRed-ED1C24?style=for-the-badge)
![SMBIOS](https://img.shields.io/badge/SMBIOS-MacBookPro16%2C2-555555?style=for-the-badge)

![Status](https://img.shields.io/badge/status-daily_usable_on_AC-brightgreen?style=flat-square)
![Updated](https://img.shields.io/badge/updated-2026--10--29-blue?style=flat-square)

</div>

> [!WARNING]
> **SMBIOS is redacted.** Generate your own Serial, MLB, SystemUUID and ROM before booting — see [SMBIOS](#-smbios).

> [!CAUTION]
> **The system freezes on battery power** — this also happens in Windows and is a known issue on the 2021 AMD G513 series, not an OpenCore problem. Use on **AC power**.

---

## 📑 Contents

- [Overview](#-overview)
- [Hardware status](#-hardware-status)
- [EFI structure](#-efi-structure)
- [Configuration](#%EF%B8%8F-configuration)
- [Hidden BIOS settings](#-hidden-bios-settings)
- [SMBIOS](#-smbios)
- [Installation](#-installation)
- [Known issues](#-known-issues)
- [Credits](#-credits)

---

## 🧾 Overview

| Item | Value |
|---|---|
| Laptop | ASUS ROG Strix G15 **G513QC** (BIOS G513QC.335) |
| CPU | AMD Ryzen 7 5800H (8C / 16T) |
| iGPU | AMD Radeon (Cezanne) — **4 GB UMA** |
| dGPU | NVIDIA RTX 3050 Laptop — *unsupported* |
| RAM | 32 GB |
| Wi-Fi / BT | Intel AX200 |
| macOS | Sequoia 15.8.1 (24H32) |
| OpenCore | 1.0.9 (REL-109-2026-10-21, Dortania build) |
| Boot picker | OpenCanopy — `Acidanthera\GoldenGate` |

---

## ✅ Hardware status

| Component | Model | Status | Notes |
|---|---|:---:|---|
| CPU | Ryzen 7 5800H | ✅ | AMD Vanilla patches, ForgedInvariant |
| iGPU | Radeon (Cezanne) | ✅ | NootedRed, Metal 3, 1080p 144 Hz, 4 GB VRAM |
| dGPU | RTX 3050 Laptop | ❌ | No macOS driver |
| Audio | Realtek ALC294 | ✅ | AppleALC layout **66** |
| Wi-Fi | Intel AX200 | ✅ | AirportItlwm |
| Bluetooth | Intel AX200 | ✅ | Pairs & streams (UTBMap + SBC codec) |
| Ethernet | Realtek RTL8111 | ✅ | RealtekRTL8111 |
| Keyboard | PS/2 (N-KEY) | ✅ | VoodooPS2 — Alt = ⌘, Win = ⌥ |
| Trackpad | I2C HID | ✅ | VoodooI2C |
| USB | 2× USB-A, 1× USB-C | ✅ | Custom `UTBMap.kext` |
| HDMI | — | ✅ | 1080p 60 Hz, 1440p 144 Hz tested |
| Brightness | — | ✅ | SSDT-PNLF + BrightnessKeys |
| Battery | — | ⚠️ | Reads & charges; freezes on battery |
| Webcam | — | ➖ | Not present on this model |
| Continuity / AirDrop / Instant Hotspot / iPhone Mirroring | — | ❌ | Requires a Broadcom Wi-Fi card |

---

## 📁 EFI structure

```text
EFI
├── BOOT
│   └── BOOTx64.efi
└── OC
    ├── ACPI
    ├── Drivers
    ├── Kexts
    ├── Resources
    ├── OpenCore.efi
    └── config.plist
```

> [!NOTE]
> Download with **Code → Download ZIP** and copy the `EFI` folder to your USB stick or EFI partition.

---

## ⚙️ Configuration

### Boot-args

```text
-v debug=0x100 keepsyms=1 -vi2c-force-polling npci=0x3000 -NRedDPDelay alcid=66
```

| Arg | Purpose |
|---|---|
| `-v debug=0x100 keepsyms=1` | Verbose boot + debugging (kept — removing caused a black screen) |
| `-vi2c-force-polling` | VoodooI2C polling mode for the trackpad |
| `npci=0x3000` | PCI configuration fix |
| `-NRedDPDelay` | NootedRed display-link delay |
| `alcid=66` | Audio layout for ALC294 |

<details>
<summary><b>Kexts</b></summary>

| Category | Kexts |
|---|---|
| Core | Lilu, VirtualSMC, SMCBatteryManager, SMCLightSensor |
| Graphics | NootedRed |
| CPU | ForgedInvariant, AppleMCEReporterDisabler, RestrictEvents |
| Audio | AppleALC |
| Wi-Fi / BT | AirportItlwm, IntelBluetoothFirmware, IntelBTPatcher, BlueToolFixup |
| Ethernet | RealtekRTL8111 |
| USB | USBToolBox, UTBMap, GenericUSBXHCI |
| Input | VoodooPS2Controller, VoodooI2C, VoodooI2CHID, BrightnessKeys |
| Storage | NVMeFix |

</details>

<details>
<summary><b>UEFI drivers & ACPI</b></summary>

**Drivers:** `HfsPlus.efi` · `OpenRuntime.efi` · `ResetNvramEntry.efi` · `OpenCanopy.efi`

**ACPI:** `SSDT-ALS0` · `SSDT-EC` · `SSDT-PLUG-ALT` · `SSDT-PNLF` · `SSDT-USB-Reset` · `SSDT-USBX` · `SSDT-XOSI`

</details>

<details>
<summary><b>Key settings</b></summary>

| Setting | Value | Reason |
|---|---|---|
| `Misc > Boot > PickerMode` | `External` | OpenCanopy GUI |
| `Misc > Boot > PickerAttributes` | `145` | Icons, mouse, flavour icons |
| `Misc > Boot > LauncherOption` | `Full` | Boot entry on internal disk |
| `Misc > Boot > HideAuxiliary` | `false` | Show Recovery / Reset NVRAM |
| `SSDT-Disable_GPU_GPP0` | Disabled | Powering off dGPU resets at *PCI configuration begin* |

</details>

---

## 🔧 Hidden BIOS settings

Changed with [Smokeless_UMAF](https://github.com/DavidS95/Smokeless_UMAF) (`UniversalAMDFormBrowser`, booted from a FAT32 USB stick).

| Path | Setting | Value | Why |
|---|---|---|---|
| AMD CBS → NBIO Common Options → GFX Configuration | iGPU Configuration | `UMA_SPECIFIED` | Unlocks frame buffer size |
| | UMA Frame buffer Size | `4G` | 512 MB default causes screen corruption / GPU hangs |
| AMD CBS → CPU Common Options | Global C-state Control | `Disabled` | Idle kernel panic (cores not waking) |

> [!CAUTION]
> Change only these settings. Recovery: BIOS (**F2**) → **F9** → **F10**, or unplug AC and hold power for 40 s.

---

## 🔑 SMBIOS

1. Download [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS).
2. Generate for **MacBookPro16,2**.
3. Fill `PlatformInfo > Generic` in `config.plist`:

| Key | Placeholder |
|---|---|
| `SystemSerialNumber` | `REPLACE_WITH_YOUR_SERIAL` |
| `MLB` | `REPLACE_WITH_YOUR_MLB` |
| `SystemUUID` | `00000000-0000-0000-0000-000000000000` |
| `ROM` | `000000000000` → your NIC MAC address |

---

## 🚀 Installation

1. Format a USB stick as **FAT32 (GUID)**.
2. Copy `EFI` to the USB root and add the macOS recovery.
3. Fill in SMBIOS.
4. Boot from USB → install macOS. *The first boot can sit on a grey screen for 15–20 minutes.*
5. Copy `EFI/OC` to the internal EFI partition (keep `Microsoft` / `Boot` if dual-booting).
6. Add a boot entry from Windows (admin Command Prompt):

```cmd
bcdedit /copy {bootmgr} /d "OpenCore"
bcdedit /set {GUID} path \EFI\OC\OpenCore.efi
```

7. BIOS (**F2**) → move **OpenCore** to first boot priority.

---

## ⚠️ Known issues

### 🔴 Critical

| Issue | Workaround |
|---|---|
| **Freeze on battery** (macOS **and** Windows) | Known 2021 AMD G513 issue, no official fix. Use AC power; Performance mode (fan key) in Windows; update BIOS; clean fans. A new battery did **not** fix it for some owners. |
| **Kernel panic at idle** — screen freezes, pointer moves (`mp_cpus_call_wait() timeout`) | AMD patches verified. **Global C-state Control → Disabled** + `sudo pmset -a tcpkeepalive 0 womp 0 powernap 0`. *Under test.* |

### 🟡 Workarounds

| Issue | Workaround |
|---|---|
| Bluetooth won't connect | Map the Intel BT port as internal (`UsbConnector 255`) in UTBMap |
| Bluetooth audio drops 1–2 s | Force SBC (`bluetoothaudiod` AAC/AptX → `false`); Apple Music: Lossless **off**, Dolby Atmos **off** |
| Screen corruption in Chromium/Electron apps | Fixed by 4 GB VRAM. Fallback: `open -a <App> --args --disable-gpu --disable-gpu-compositing` |
| YouTube laggy in Safari | Likely software VP9 decode. Use 720p/1080p without 60 fps, or Chrome + **enhanced-h264ify** |
| iPhone hotspot won't connect | Instant Hotspot unsupported. iPhone → Personal Hotspot → **Maximize Compatibility ON**, join as WPA2 Personal — or use **USB tethering** |
| UI glitches / random black screen | Reduce transparency, 60 Hz, disable auto-brightness |
| External display flickers on connect | Normal — stabilises in seconds |
| No fan / performance control | Set the mode in Windows (G-Helper); the EC handles fans |

---

## 🙏 Credits

[Acidanthera](https://github.com/acidanthera) · [ChefKiss — NootedRed](https://github.com/ChefKissInc/NootedRed) · [OpenIntelWireless](https://github.com/OpenIntelWireless) · [VoodooI2C](https://github.com/VoodooI2C) · [USBToolBox](https://github.com/USBToolBox) · [OpCore Simplify](https://github.com/lzhoang2801/OpCore-Simplify) · [Smokeless_UMAF](https://github.com/DavidS95/Smokeless_UMAF) · [Dortania](https://dortania.github.io)

<div align="center"><sub>Use at your own risk. Not affiliated with Apple or ASUS.</sub></div>
