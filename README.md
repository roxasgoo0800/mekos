# Hackintosh Backup - AMD Ryzen 7 5700X & NVIDIA GeForce RTX 3060

Repository ini berisi backup lengkap konfigurasi **EFI**, **Drivers (Kexts)**, **Tools**, dan bootloader untuk sistem Hackintosh **AMD Ryzen 7 5700X** dengan **NVIDIA GeForce RTX 3060** (NullMoth / Metal 3).

---

## 🖥 Spesifikasi Hardware

| Komponen | Spesifikasi / Model |
| :--- | :--- |
| **CPU** | AMD Ryzen 7 5700X 8-Core, 16-Threads @ 3.4 GHz |
| **GPU** | NVIDIA GeForce RTX 3060 12GB GDDR6 (NullMoth NVMTL Metal 3) |
| **RAM** | 16 GB DDR4 |
| **Motherboard** | AMD AM4 (PCIe Gen 4) |
| **Ethernet (LAN)** | Realtek RTL8111 Gigabit Ethernet (`RealtekRTL8111.kext`) |
| **Wi-Fi** | Intel Wi-Fi (`AirportItlwm.kext` / `itlwm.kext` + HeliPort) |
| **Bluetooth** | Intel Wireless Bluetooth (`IntelBluetoothFirmware.kext`, `IntelBTPatcher.kext`, `BlueToolFixup.kext`) |
| **Audio** | Realtek ALC (`AppleALC.kext` alcid=11 + `SimpleHDA.kext`) |
| **Storage** | NVMe M.2 SSD (`NVMeFix.kext`, `CtlnaAHCIPort.kext`) |
| **Bootloader** | OpenCore (DBG-109 / 1.0.x) |
| **SMBIOS** | `MacPro7,1` |
| **Dual Boot** | Windows 11 / macOS |

---

## 📁 Struktur Repository

```text
mekos/
├── EFI/
│   ├── BOOT/
│   │   └── bootx64.efi              # OpenCore fallback bootloader
│   ├── OC/
│   │   ├── ACPI/                    # SSDT-EC, SSDT-PLUG-ALT, SSDT-USB-Reset, SSDT-USBX
│   │   ├── Drivers/                 # OpenRuntime, apfs_aligned, HfsPlus, ResetNvramEntry
│   │   ├── Kexts/                   # Seluruh Kernel Extensions untuk macOS
│   │   ├── Resources/               # OpenCanopy GUI Icons, Fonts, Audio
│   │   ├── Tools/                   # NullMothSafe.efi
│   │   ├── config.plist             # Konfigurasi aktif OpenCore
│   │   └── config.plist.bak*        # Backup riwayat config.plist sebelumnya
│   └── Microsoft/                   # Bootloader Windows (Dual-Boot backup)
├── Drivers_System/                  # Driver GPU NVIDIA RTX 3060 (/Library/Extensions)
│   ├── NVAccel.kext
│   │   └── Contents/...
│   ├── NVRM.kext
│   │   └── Contents/...
│   ├── NVRMAGDC.kext
│   │   └── Contents/...
│   └── NVRMFB.kext
│       └── Contents/...
├── Tools/                           # Aplikasi pendukung Hackintosh
│   ├── HeliPort.dmg                 # GUI client Wi-Fi Intel untuk itlwm
│   └── 1401-Mac-2.6.4.zip           # Tool helper NullMoth macOS
└── Logs/
    └── opencore-boot-log.txt        # OpenCore boot debug log
```

---

## 🧩 Daftar Kexts (Drivers)

### OpenCore Kexts (`EFI/OC/Kexts/`):
- **Lilu.kext**: Kernel patcher utama untuk seluruh plugin macOS.
- **VirtualSMC.kext**: Emulasi chip Apple SMC untuk AMD Ryzen.
- **WhateverGreen.kext**: Manajemen output grafis dan framebuffer display.
- **AppleALC.kext** & **SimpleHDA.kext**: Audio codec driver dengan layout-id `11`.
- **AirportItlwm.kext** & **itlwm.kext**: Driver Wi-Fi Intel (AirportItlwm untuk native integration, itlwm untuk pairing dengan HeliPort).
- **IntelBluetoothFirmware.kext** & **IntelBTPatcher.kext** & **BlueToolFixup.kext**: Dukungan penuh Intel Bluetooth.
- **RealtekRTL8111.kext**: Driver LAN Realtek Gigabit.
- **NVMeFix.kext**: Manajemen daya dan stabilitas SSD NVMe.
- **CtlnaAHCIPort.kext**: Kompatibilitas kontroler SATA AHCI.
- **AMFIPass.kext**: Penanganan AMFI untuk kompatibilitas driver legacy/custom.
- **AppleMCEReporterDisabler.kext**: Mencegah kernel panic MCE pada CPU AMD Ryzen / SMBIOS MacPro7,1.
- **RestrictEvents.kext**: Manajemen CPU naming dan proses sistem.
- **USBToolBox.kext** & **UTBDefault.kext**: Pemetaan port USB motherboard.

### System NVIDIA RTX Drivers (`Drivers_System/`):
- `NVAccel.kext`
- `NVRM.kext`
- `NVRMAGDC.kext`
- `NVRMFB.kext`
> Driver ini diinstal di `/Library/Extensions` untuk akselerasi grafis NVIDIA RTX seri Ampere (RTX 3060) melalui proyek NullMoth / 1401 dengan dukungan Metal 3.

---

## 🚀 Panduan Restore / Pemasangan EFI

### 1. Memasang ke Partisi EFI (macOS)
1. Buka Terminal macOS.
2. Cek disk dan partisi EFI Anda:
   ```bash
   diskutil list
   ```
3. Mount partisi EFI (misal disk internal ada di `disk1s1` atau USB di `disk3s1`):
   ```bash
   diskutil mount /dev/disk1s1
   ```
4. Salin folder `EFI` dari repository ini ke root volume partisi EFI yang sudah di-mount:
   ```bash
   cp -R EFI/* "/Volumes/EFI/EFI/"
   # atau jika ter-mount sebagai "NO NAME":
   cp -R EFI/* "/Volumes/NO NAME/EFI/"
   ```

### 2. Memasang Driver NVIDIA RTX (`Drivers_System`) jika Instal Ulang macOS
Jika Anda baru melakukan install ulang macOS dan butuh mengembalikan driver NullMoth RTX 3060:
1. Pastikan boot-args di `config.plist` mencakup:
   `debug=0x100 keepsyms=1 -ibtcompatbeta revpatch=sbvmm nvfb=1 nvaccel=1 nvfbheads=4 -nvkmsnosmooth amfi_get_out_of_my_way=0x1 amfi=0x80 ipc_control_port_options=0 gfxrst=4 alcid=11 shda-lang=en`
2. Salin kexts dari folder `Drivers_System/` ke `/Library/Extensions/`:
   ```bash
   sudo cp -R Drivers_System/*.kext /Library/Extensions/
   sudo chown -R root:wheel /Library/Extensions/NV*.kext
   sudo chmod -R 755 /Library/Extensions/NV*.kext
   sudo kmutil install --update-volume
   ```
3. Ekstrak dan jalankan `Tools/1401-Mac-2.6.4.zip` untuk verifikasi dan diagnostik NullMoth.

---

> [!NOTE]
> Jika repository ini bersifat publik, disarankan untuk mengonversi repository menjadi **Private** melalui pengaturan GitHub untuk menjaga kerahasiaan serial number SMBIOS (`MLB`, `SystemSerialNumber`, `SystemUUID`) yang tertaut pada akun Apple Anda.
