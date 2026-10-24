# ASUS ROG Strix G15 G513QC — OpenCore EFI

> OpenCore EFI for running **macOS Sequoia 15.8.1** on the ASUS ROG Strix G15 (G513QC), Ryzen 7 5800H + Radeon iGPU.

| | |
|---|---|
| **macOS** | Sequoia 15.8.1 (24H32) |
| **OpenCore** | 1.0.9 (REL-109-2026-10-21, Dortania nightly) |
| **SMBIOS** | MacBookPro16,2 |
| **Boot picker** | OpenCanopy — `Acidanthera\GoldenGate` |
| **Last updated** | 2026-10-24 |

> **Warning**
> SMBIOS is **redacted**. Generate your own values before booting — see [SMBIOS](#smbios).

> **Critical**
> System **freezes on battery power** — also happens in **Windows**, so it is not caused by macOS/OpenCore. This is a **known issue on the 2021 AMD G513 series** (G513QE / QM / QY reports on the ROG forum). Use on **AC power**. See [Known issues](#known-issues).

---

## Hardware

| Component | Model | Status | Notes |
|---|---|:---:|---|
| CPU | AMD Ryzen 7 5800H | ✅ | ForgedInvariant |
| iGPU | AMD Radeon (Cezanne) | ✅ | NootedRed, Metal 3, 1080p 144 Hz, **4 GB VRAM** (UMA) |
| dGPU | NVIDIA RTX 3050 Laptop | ❌ | No macOS driver |
| Audio | Realtek ALC294 | ✅ | AppleALC, layout **66** |
| Wi-Fi | Intel AX200 | ✅ | AirportItlwm |
| Bluetooth | Intel AX200 | ✅ | Pairs & streams audio (needs UTBMap, SBC codec) — tested BT speaker |
| Keyboard | PS/2 (N-KEY) | ✅ | VoodooPS2 — Alt = ⌘, Win = ⌥ |
| Trackpad | I2C HID | ✅ | VoodooI2C |
| USB | 2× USB-A, 1× USB-C | ✅ | Custom `UTBMap.kext` |
| HDMI | — | ✅ | Tested LG 23MP75 1080p 60 Hz · Samsung Odyssey G50D 2560×1440 144 Hz |
| Brightness | — | ✅ | SSDT-PNLF + BrightnessKeys |
| Battery | — | ⚠️ | Detected/charges; **freezes on battery** (known G513 issue, also in Windows) |
| Webcam | — | ➖ | Not present on this model |
| Continuity / AirDrop | — | ❌ | Requires Broadcom Wi-Fi card |

---

## EFI structure

```text
EFI
├── BOOT
│   └── BOOTx64.efi
└── OC
    ├── ACPI
    ├── Drivers
    ├── Kexts
    ├── Resources
    ├── Tools
    ├── OpenCore.efi
    └── config.plist
```

> **Note**
> `EFI/BOOT/BOOTx64.efi` is not included. Copy it from the matching [OpenCorePkg](https://github.com/acidanthera/OpenCorePkg) build.

---

## Configuration

### Boot-args

```text
-v debug=0x100 keepsyms=1 -vi2c-force-polling npci=0x3000 -NRedDPDelay alcid=66
```

| Arg | Purpose |
|---|---|
| `-v debug=0x100 keepsyms=1` | Verbose boot + debug (kept on — removing caused black screen) |
| `-vi2c-force-polling` | VoodooI2C polling mode for trackpad |
| `npci=0x3000` | PCI configuration fix |
| `-NRedDPDelay` | NootedRed display link delay |
| `alcid=66` | Audio layout for ALC294 |

### Kexts

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

### UEFI drivers

`HfsPlus.efi` · `OpenRuntime.efi` · `ResetNvramEntry.efi` · `OpenCanopy.efi`

### ACPI

`SSDT-ALS0` · `SSDT-EC` · `SSDT-PLUG-ALT` · `SSDT-PNLF` · `SSDT-USB-Reset` · `SSDT-USBX` · `SSDT-XOSI`

### Key settings

| Setting | Value | Reason |
|---|---|---|
| `Misc > Boot > PickerMode` | `External` | OpenCanopy GUI |
| `Misc > Boot > PickerAttributes` | `145` | Icons + mouse + flavour icons |
| `Misc > Boot > LauncherOption` | `Full` | Boot entry on internal disk |
| `Misc > Boot > HideAuxiliary` | `false` | Show Recovery / Reset NVRAM |
| `SSDT-Disable_GPU_GPP0` | Disabled | Powering off dGPU resets at *PCI configuration begin* |

---

## Hidden BIOS settings

Changed with [Smokeless_UMAF](https://github.com/DavidS95/Smokeless_UMAF) (`UniversalAMDFormBrowser`, booted from FAT32 USB) on BIOS **G513QC.335**. Use at your own risk.

| Path | Setting | Value | Why |
|---|---|---|---|
| Device Manager → AMD CBS → NBIO Common Options → GFX Configuration | iGPU Configuration | `UMA_SPECIFIED` | Unlock frame buffer size |
| | UMA Frame buffer Size | `4G` | Default 512 MB causes screen corruption / GPU hangs (32 GB RAM system) |
| Device Manager → AMD CBS → CPU Common Options | Global C-state Control | `Disabled` | Idle kernel panic (cores not waking) |

Recovery: BIOS (F2) → F9 → F10, or unplug AC and hold power 40 s.

---

## SMBIOS

1. Download [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS).
2. Generate for **MacBookPro16,2**.
3. Fill `PlatformInfo > Generic` in `config.plist`:

| Key | Placeholder |
|---|---|
| `SystemSerialNumber` | `REPLACE_WITH_YOUR_SERIAL` |
| `MLB` | `REPLACE_WITH_YOUR_MLB` |
| `SystemUUID` | `00000000-0000-0000-0000-000000000000` |
| `ROM` | `000000000000` → use your NIC MAC address |

---

## Installation

1. Format a USB stick as FAT32 (GUID).
2. Copy `EFI` to the USB root and add the macOS recovery.
3. Fill in SMBIOS.
4. Boot from USB → install macOS → first boot can take 15–20 min on a grey screen.
5. Copy `EFI/OC` to the internal EFI partition (keep `Microsoft` / `Boot` folders if dual-booting).
6. In Windows (admin):

```cmd
bcdedit /copy {bootmgr} /d "OpenCore"
bcdedit /set {GUID} path \EFI\OC\OpenCore.efi
```

7. BIOS (F2) → move **OpenCore** to first boot priority.

---

## Known issues

| Issue | Workaround |
|---|---|
| 🔴 **System freeze on battery** (macOS **and** Windows) | Known G513 (2021, AMD) issue, no official fix. Reported causes: Silent/battery power profile underclocking the CPU, GPU switching, high temps. Use AC power; set **Performance** mode with the fan key (not Silent) in Windows; update BIOS; clean fans. Battery replacement did **not** fix it for some owners |
| 🔴 **Kernel panic at idle** — screen freezes, pointer still moves (`mp_cpus_call_wait() timeout`, 14/16 CPUs no ACK) | AMD Vanilla patches verified (8 cores, `ProvideCurrentCpuInfo`, ForgedInvariant). **Global C-state Control → Disabled** via Smokeless_UMAF (see [Hidden BIOS settings](#hidden-bios-settings)) + `sudo pmset -a tcpkeepalive 0 womp 0 powernap 0`. *Under test.* |
| Bluetooth audio drops 1–2 s | Force SBC: `sudo defaults write bluetoothaudiod "Enable AAC codec" -bool false` and `"Enable AptX codec" -bool false`. Apple Music: **Lossless off**, **Dolby Atmos off** (downloaded songs & YouTube play clean on Wi-Fi and Ethernet) |
| Bluetooth won't connect | Map the Intel BT USB port as internal (`UsbConnector 255`) in UTBMap.kext |
| Screen corruption in Chromium/Electron apps | Main fix: 4 GB VRAM (see [Hidden BIOS settings](#hidden-bios-settings)). Fallback: `open -a <App> --args --disable-gpu --disable-gpu-compositing` |
| UI glitches / random black screen | Reduce transparency, 60 Hz, disable auto-brightness |
| iGPU VRAM only 512 MB | ✅ Raised to 4 GB via Smokeless_UMAF (see [Hidden BIOS settings](#hidden-bios-settings)) |
| External display flickers on connect | Normal — stabilises in seconds |
| No fan / performance control | Set mode in Windows (G-Helper); EC handles fans |

---

## Credits

[Acidanthera](https://github.com/acidanthera) · [ChefKiss — NootedRed](https://github.com/ChefKissInc/NootedRed) · [OpenIntelWireless](https://github.com/OpenIntelWireless) · [VoodooI2C](https://github.com/VoodooI2C) · [USBToolBox](https://github.com/USBToolBox) · [OpCore Simplify](https://github.com/lzhoang2801/OpCore-Simplify) · [Dortania](https://dortania.github.io)
