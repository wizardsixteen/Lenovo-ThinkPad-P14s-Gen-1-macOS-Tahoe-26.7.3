# macOS Tahoe 26.7.3 on the Lenovo ThinkPad P14s Gen 1 (OpenCore EFI)

A ready-to-use OpenCore EFI for the **Lenovo ThinkPad P14s Gen 1** (i7-10610U), dual-booting macOS Tahoe with Windows 11. Download, unzip, copy the `EFI` folder to your EFI partition, set your own SMBIOS, reboot.

> Built and tested on one machine only. Use at your own risk and back up your existing EFI first.

## Tested configuration

| Item              | Value                                                                         |
| ----------------- | ----------------------------------------------------------------------------- |
| Laptop            | ThinkPad P14s Gen 1, model 20S5S07X06                                         |
| BIOS              | N2XET46W 1.36 (other versions untested)                                       |
| CPU               | Intel Core i7-10610U                                                          |
| Graphics          | Intel UHD 630 (drives the panel and HDMI); NVIDIA Quadro P520 is off in macOS |
| Wi-Fi / Bluetooth | Intel AX201                                                                   |
| Audio             | Realtek ALC257                                                                |
| Trackpad          | Synaptics (SMBus/RMI)                                                         |
| Storage           | Samsung PM981a NVMe                                                           |
| macOS             | **26.7.3, build 25G342**                                                      |
| OpenCore          | 1.0.8 (REL-108)                                                               |
| SMBIOS            | MacBookPro16,2                                                                |

## Supported models

| Model                                      | Status                                                                                                             |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------ |
| ThinkPad P14s Gen 1 (Intel, i7-10610U)     | Tested, everything in the table below                                                                              |
| ThinkPad T14 Gen 1 (Intel)                 | Untested. Same chassis and 10th-gen (Comet Lake) platform as the P14s Gen 1, so it should need little or no change |
| Other 10th-gen Intel ThinkPads (P15s, T15) | Untested. Same platform family, but different hardware; expect to adjust the HDMI connector patch and the USB map  |

Notes for siblings:

- Other CPUs in the same U-series family (i5 or i7) should work; the CPU power-management SSDT is generic.
- The NVIDIA GPU (Quadro P520, MX330 and similar) has no macOS driver on any model. It stays off with `-wegnoegpu`.
- The USB map (`UTBMap`) and the HDMI connector patch were made on the P14s Gen 1. If a port or HDMI does not work on your model, remap USB with USBToolBox and recheck the HDMI bus ID.
- The APFS boot-trim patch applies to any machine on macOS 26.7.3 (25G342); it only saves time on drives with slow free-space TRIM.
- If you test another model, open an issue with what worked and what did not.

## What works

| Feature                                           | Status                                                                                      |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| Boot to macOS, ~21 s (FileVault password to Dock) | Works                                                                                       |
| Intel UHD 630 graphics                            | Works                                                                                       |
| Brightness and Fn brightness keys                 | Works                                                                                       |
| Wi-Fi                                             | Works                                                                                       |
| Bluetooth                                         | Works                                                                                       |
| Audio output                                      | Works                                                                                       |
| Trackpad and keyboard                             | Works                                                                                       |
| Camera                                            | Works                                                                                       |
| Battery                                           | Works                                                                                       |
| Sleep, lid close and open                         | Works                                                                                       |
| Clamshell with an external HDMI monitor           | Works (AC power and an external keyboard or mouse needed)                                   |
| HDMI output                                       | Works after one cable replug once the desktop has loaded                                    |
| Windows 11 dual boot                              | Works                                                                                       |
| Internal microphone                               | **Does not work** (no macOS driver for the Intel DSP mic array; use a USB or Bluetooth mic) |
| NVIDIA Quadro P520                                | **Not usable in macOS** (no driver; kept off with `-wegnoegpu`). Use Windows for it         |
| Ethernet (I219-LM)                                | Driver included, not tested                                                                 |

## Before you start

1. **macOS 26.7.3 (25G342) must be installed.** This EFI boots an installed system and does not install macOS by itself. Starting from Windows only? Follow **Dual boot with Windows** below.
2. **BIOS:** Secure Boot **Off**. Graphics mode **Hybrid** (recommended). Thunderbolt was left disabled on the tested machine.
3. **SMBIOS:** `PlatformInfo` in `config.plist` holds placeholder values (serial `W00000000001`, MLB `M00000000000001`, zero UUID and ROM). The Mac boots with them, but replace them before signing in to Apple services. Generate your own **MacBookPro16,2** serial, MLB, UUID and ROM (for example with GenSMBIOS) and paste them in. Never reuse another machine's values.

## Dual boot with Windows

This repo is made for a Windows 11 + macOS dual boot on one NVMe drive, the same setup as the tested machine. The EFI only supplies the OpenCore boot menu; you set up the two systems yourself.

**Starting from a Windows-only laptop**

1. Keep Windows installed in UEFI mode on a GPT disk. Turn BitLocker off and disable Fast Startup in Windows before touching partitions.
2. In Windows Disk Management, shrink a volume to free space for macOS (at least 100 GB recommended), then create a **New Simple Volume** in it (exFAT is fine) and name it for macOS.
3. Make a standard OpenCore USB installer for macOS Tahoe and boot it. This EFI was tested for booting an installed system, not as an installer: if the installer does not start with it, use a stock OpenCore installer EFI first and switch to this one after the install.
4. In the macOS installer, open Disk Utility, erase only that new volume as **APFS**, and install macOS 26.7.3 on it. Do not erase the Windows partitions or any EFI partition.
5. After the install finishes, copy this EFI to your internal EFI partition (see **Install** below) and set your own SMBIOS.
   **How the two EFI options work**

| Option                              | How                                                                         | Notes                                                                                                                                                                   |
| ----------------------------------- | --------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Share Windows's EFI partition       | Copy `EFI/OC` and `EFI/BOOT` into the existing EFI partition                | Simplest. Windows keeps its own `EFI/Microsoft` folder there                                                                                                            |
| Separate EFI partition for OpenCore | Create a small FAT32 EFI-type partition (about 300 MB) and copy `EFI` there | What the tested machine uses. Windows's EFI partition stays untouched. With `LauncherOption` set to Full, OpenCore registers one "OpenCore" entry in the BIOS boot menu |

**Day to day**

- The OpenCore picker lists macOS and Windows. After 5 seconds it boots the default.
- To make macOS the default, use **System Settings > Startup Disk**, or press **Ctrl+Enter** on the entry in the picker. To make Windows the default, select it in the picker and press Ctrl+Enter.
- Windows can also be started from F12 > **Windows Boot Manager**, which bypasses OpenCore. Use that if Windows behaves oddly when started through OpenCore (for example asking you to reset a PIN or sign in again).
- A Windows update can change the BIOS boot order. If the picker does not show, press F12 and choose **OpenCore**, or put it first in the BIOS boot order.
- Windows and macOS treat the hardware clock differently. In Windows, set the `RealTimeIsUniversal` DWORD to 1 under `HKLM\SYSTEM\CurrentControlSet\Control\TimeZoneInformation`, then re-sync the time.
- The NVIDIA card works only in Windows. macOS keeps it off.

## Install

1. Click **Code > Download ZIP** on this page, then unzip it. You will see an `EFI` folder inside.
2. Find your EFI partition in Terminal:

```
diskutil list
```

Look for the partition of type `EFI` on your internal disk (for example `disk0s1`). 3. Mount it:

```
sudo diskutil mount disk0s1
```

Replace `disk0s1` with yours. It appears as `/Volumes/EFI` (or the volume's own name). 4. Back up your current EFI, then replace it. Adjust the paths to match your volume name and where you unzipped:

```
cp -R /Volumes/EFI/EFI ~/Desktop/EFI_backup
sudo rm -rf /Volumes/EFI/EFI
sudo cp -R ~/Downloads/<unzipped-folder>/EFI /Volumes/EFI/
ls /Volumes/EFI/EFI/OC
```

The last command should list `ACPI Drivers Kexts OpenCore.efi Resources Tools config.plist`. 5. Set your SMBIOS values in `/Volumes/EFI/EFI/OC/config.plist` (see above). 6. Reboot, press F12, choose **OpenCore**, and boot macOS.

> The `EFI` folder must sit at the root of the EFI partition, not inside another folder. If Archive Utility reports error 513 on the EFI partition, use `sudo cp` or `sudo unzip` in Terminal as above.

## After the first boot

1. Open **System Settings > Startup Disk**, pick your macOS volume. Or highlight macOS in the OpenCore picker and press **Ctrl+Enter**.
2. From then on, the picker waits 5 seconds and boots macOS by itself. Windows stays in the picker, and F12 > Windows Boot Manager also works.
3. HDMI: unplug and replug the cable once after the desktop loads. The monitor is not detected at boot.

## The boot-time patch

macOS re-trims all free space on every boot, and on the Samsung PM981a that costs about 22 seconds. This EFI skips it with a Kernel > Patch on the APFS kext.

- It is written for **macOS 26.7.3, build 25G342 only**.
- On any other build the pattern will not match; OpenCore skips the patch and the Mac boots normally, just slower (about 47 s).
- Check that it is active:

```
log show --last boot --predicate 'eventMessage contains "spaceman_scan_free_blocks"' | tail -3
```

A line ending in `(no trims)` means it is working.

## Notes

- Boot arguments: `-wegnoegpu` only.
- After clearing NVRAM, boot through the OpenCore firmware entry (F12), then set Startup Disk again so macOS auto-boots.
- If a clock mismatch appears in Windows, set the `RealTimeIsUniversal` DWORD to 1 under `HKLM\SYSTEM\CurrentControlSet\Control\TimeZoneInformation`.
- Test any change from a USB copy of the EFI before writing it to the internal disk.

## Credits

OpenCore and Lilu, WhateverGreen, AppleALC, VirtualSMC, ECEnabler, RestrictEvents, BrightnessKeys (Acidanthera); VoodooPS2, VoodooRMI, VoodooSMBus, VoodooInput; AirportItlwm and IntelBluetoothFirmware (OpenIntelWireless); IntelBTPatcher; BlueToolFixup; IntelMausi; NVMeFix; USBToolBox; YogaSMC; SimpleHDA. Each project keeps its own license.
