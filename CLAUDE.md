# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

An OpenCore EFI configuration for running macOS Tahoe on an HP EliteBook 840 G6 (Intel i5-8365U / UHD 620). It is not a software project — there is no build system, no tests, and nothing to compile. All ACPI tables are pre-compiled `.aml` binaries; all kexts are pre-built bundles.

## The Only Editable File

**`EFI/OC/config.plist`** is the central OpenCore configuration and the only plain-text file you should edit. It is an Apple XML plist with these top-level sections:

| Section | Purpose |
|---|---|
| `ACPI` | Loads custom SSDTs; ACPI binary patches (RTC fix, `_OSI`→`XOSI` rename) |
| `Booter` | Memory/boot quirks; Skip Board ID check patch |
| `DeviceProperties` | Intel UHD 620 framebuffer injection (`AAPL,ig-platform-id`, `device-id`) |
| `Kernel` | Kext load order and enable/disable state; kernel quirks |
| `Misc` | Boot picker settings (OpenCanopy, 5s timeout), debug, security |
| `NVRAM` | boot-args, CSR config, Bluetooth variables |
| `PlatformInfo` | SMBIOS spoof as `MacBookPro15,2` |
| `UEFI` | EFI driver list, APFS support, input/output quirks |

When editing kext entries under `Kernel > Add`, the entries must match the actual `.kext` bundles present in `EFI/OC/Kexts/`. Use ProperTree or OCAuxiliaryTools for editing if possible; the XML structure must stay valid.

## Key Configuration Details

**boot-args:**
```
debug=0x100 keepsyms=1 rtcfx_exclude=0E-FF -amfipassbeta -igfxblt -vi2c-force-polling alcverbs=1 -alcoff
```

**SMBIOS:** `MacBookPro15,2` — the serial numbers in the repo are placeholders and must be regenerated with GenSMBIOS before using iCloud or Apple services.

**Security:** `SecureBootModel = Disabled`, `Vault = Optional`, `ScanPolicy = 0`.

## Hardware Target

| Component | Detail |
|---|---|
| CPU | Intel Core i5-8365U (Whiskey Lake-U) |
| iGPU | Intel UHD 620 (8086:3EA0) |
| Ethernet | Intel I219-LM (8086:15BD) |
| Wi-Fi | Intel Dual Band Wireless-AC 8265 (8086:24FD) |
| Audio | Realtek ALC215 (10EC:0215) via Cannon Point-LP HDA |
| USB | Intel Cannon Point-LP XHCI (8086:9DED) |

## Notable Configuration Choices

**Audio:** Both `AppleALC.kext` and `VoodooHDA.kext` are disabled in `config.plist`. Audio requires a post-install manual VoodooHDA installation following [chris1111/VoodooHDA-Tahoe](https://github.com/chris1111/VoodooHDA-Tahoe).

**Wi-Fi:** Uses `itlwm.kext` (not `AirportItlwm`), which requires the HeliPort app for a Wi-Fi menu. `HeliPort-1.5.dmg` is included in `resources/`. `AirportItlwm.kext` is present in `Kexts/` but disabled.

**Touchpad:** Dual stack — `VoodooI2C` + `VoodooRMI` for I2C, and `VoodooPS2Controller` for PS/2 fallback. The duplicate `VoodooInput` inside `VoodooRMI` and `VoodooPS2Controller` bundles are disabled to avoid conflict with the standalone `VoodooInput` loaded by `VoodooI2C`.
