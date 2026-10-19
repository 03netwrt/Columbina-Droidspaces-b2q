# Columbina-Droidspaces-b2q

Kernel custom **5.4.304-openela-Columbina-03net** untuk **Samsung Galaxy Z Flip3 5G (SM-F711B, codename `b2q`)** — varian lain JANGAN.

Basis: [AlexFurina/android_kernel_samsung_sm8350](https://github.com/AlexFurina/android_kernel_samsung_sm8350) branch `Flip3-Android16` (Samsung OSS + OpenELA LTS, Knox/FASTUH/RKP dicabut maintainer).

**Yang ditambahkan 03NET:**
- **Full Droidspaces support** — `PID_NS`, `USER_NS`, `IPC_NS`, `NET_NS`, `UTS_NS`, cgroup penuh, `VETH`, `BRIDGE`, `OVERLAY_FS`, `DEVTMPFS`
- **KernelSU-Next v3.2.0-legacy terintegrasi** — root tanpa Magisk (driver + hooks di-build ke dalam kernel)
- **BFR TTL modifier** — `NETFILTER_XT_TARGET_HL` + `NETFILTER_XT_MATCH_HL`
- **Driver modem USB** — `USB_USBNET`, `USB_NET_CDCETHER`, `USB_NET_CDC_MBIM`, `USB_NET_QMI_WWAN`, `USB_SERIAL` (+OPTION)

Format: **AnyKernel3** ZIP — flash via TWRP, DTB/DTBO dipertahankan dari ROM. Khusus ROM Android 15–16 (diuji untuk X-ROM ZFlip5 Port / OneUI 8.0).

*Powered by 03NET — t.me/O3netwrt77*

---

# Tutorial: X-ROM ZFlip5 Port V3.2 + Kernel 03NET + KernelSU-Next di Galaxy Z Flip3

**Hasil akhir:** Android 16 / OneUI 8.0 (port Z Flip5) + root KernelSU-Next + kernel 03NET dengan dukungan penuh Droidspaces.

> ⚠️ **Peringatan:** Proses ini menghapus SELURUH data HP (Format Data). Backup dulu!
> Knox trip permanen (0x1): Samsung Pay, pairing Galaxy Watch, dan Google Wallet/NFC tidak akan bisa dipakai lagi.
> Risiko bootloop selalu ada — pastikan baterai > 60% dan ikuti langkah berurutan.

## 1. File yang dibutuhkan (download di laptop)

| File | Ukuran | Sumber |
|---|---|---|
| `XROM-ZFlip5-V3.2-ZFlip3.zip` (ROM) | ±5,44 GB | [Google Drive](https://drive.usercontent.google.com/download?id=1AFcXi04k7olEQYgWdt6QIr8oFifPub7X&export=download&confirm=t) (link dari thread XDA) |
| `twrp-b2q.tar` (TWRP) | ±96 MB | Lampiran postingan pertama [thread XDA](https://xdaforums.com/t/rom-16-oneui8-0-zflip3-x-rom-zflip5-port-v3-2.4741371/) — **harus login akun XDA** |
| `Columbina-Droidspaces-b2q-v1.1.zip` (kernel 03NET) | ±17,8 MB | [Release repo ini](https://github.com/03netwrt/Columbina-Droidspaces-b2q/releases) |
| Odin3 v3.14.4 | ±3 MB | samfw.com |
| `KernelSU-Next` Manager APK (**v3.2.0-legacy**) | ±15 MB | [Release v3.2.0-legacy](https://github.com/rifsxd/KernelSU-Next/releases/tag/v3.2.0-legacy) — pilih APK manager |
| `droidspaces-webui` modul (karya Latifan) | — | Dari arsip 03NET / Latifan |

Cek ukuran file ROM setelah download: harus **5.840.639.714 bytes**. Kurang = download ulang.

**Syarat HP:** sudah di firmware **Android 15 / OneUI 7** stock, bootloader **unlock**, USB debugging aktif.

## 2. Flash TWRP via Odin

1. HP dalam keadaan mati. Tahan **Volume Up + Volume Down**, colok kabel USB ke laptop → masuk **Download Mode** → tekan Volume Up (Continue).
2. Buka **Odin3** di laptop → pastikan HP terdeteksi (kotak `ID:COM` biru).
3. Tab **Options**: **hilangkan centang `Auto Reboot`**.
4. Klik **AP** → pilih file `twrp-b2q.tar` → klik **Start** → tunggu sampai `PASS`.
5. Begitu `PASS`: **langsung** tahan **Volume Up + Power** sampai masuk TWRP.
   ⚠️ Jangan biarkan HP boot ke sistem — ROM stock akan menimpa TWRP.

## 3. Format Data

1. Di TWRP: **Wipe → Format Data** → ketik `yes` → konfirmasi.
2. Kembali → **Reboot → Recovery** (masuk TWRP lagi).

## 4. Flash ROM

1. Copy `XROM-ZFlip5-V3.2-ZFlip3.zip` ke memori HP (via MTP di Windows Explorer, atau `adb push XROM-ZFlip5-V3.2-ZFlip3.zip /sdcard/`).
2. Di TWRP: **Install** → pilih file ROM → **Swipe to Confirm Flash** → tunggu selesai (±5 menit).
3. **JANGAN reboot dulu** — lanjut ke langkah 5.

## 5. Flash kernel 03NET

1. Copy `Columbina-Droidspaces-b2q-v1.1.zip` ke HP (kalau belum).
2. Di TWRP: **Install** → pilih file kernel → **Swipe to Confirm Flash**.
   - Installer otomatis menanam **Auto-ADB** ke ramdisk: `adbd` selalu listen di **TCP:5555** sejak boot, tanpa dialog otorisasi RSA. Permanen, independen dari Magisk/KSU.
3. **Reboot → System**. Boot pertama 5–10 menit (wajar).

## 6. Verifikasi ROM + install KernelSU-Next Manager

1. Selesaikan setup awal Android (WiFi, akun Google).
2. Cek **Settings → About phone**: harus tertulis **OneUI 8.0 / Android 16**.
3. Copy APK `KernelSU-Next` (**v3.2.0-legacy**) ke HP → install (izinkan "Install unknown apps").
4. Buka aplikasi **KernelSU Next** → status harus **"Working"** (bukan "Unsupported").
   - Kalau tertulis "Unsupported": kernel tidak terpasang benar — ulangi langkah 5.
   - ⚠️ Manager HARUS versi v3.2.0-legacy agar cocok dengan driver di kernel.
5. Di aplikasi KernelSU: berikan akses root ke aplikasi yang diminta untuk tes.

## 7. Install modul droidspaces-webui (karya Latifan)

1. Copy zip modul `droidspaces-webui` ke HP.
2. Buka KernelSU Next → tab **Modules** → **Install from storage** → pilih zip modul → reboot.
   (KernelSU mendukung modul berformat Magisk.)
3. Setelah reboot, buka browser di HP/laptop: `http://<IP-HP>:84` → dashboard Droidspaces-webui harus tampil.
4. Buat container baru → jalankan **Droidspaces Kernel Check**: semua harus **GREEN** (namespace PID/USER/IPC, cgroup, veth, bridge, overlayfs).
   - Kalau ada yang merah: kernel tidak cocok — laporkan ke 03NET.

## 8. Selesai — pengembalian fungsi kerja

- Install **Clash 03NET** (APK dari GitHub 03netwrt) dan restore config-nya.
- Install **BFR-WEBUI-GO v1.2.2** (modul, via tab Modules di KernelSU) untuk band lock + TTL modifier.
  TTL modifier memakai target iptables `HL` yang sudah di-enable di kernel ini.
- Untuk cover screen: install **Samsprung** dari Play Store (aplikasi pihak ketiga pembuka aplikasi di cover screen, tidak butuh root).

---

## Troubleshooting

| Gejala | Solusi |
|---|---|
| Bootloop setelah flash ROM | Masuk TWRP (Vol Up + Power), ulangi Format Data + flash ROM ulang |
| TWRP tertimpa stock recovery | Ulangi langkah 2 — pastikan `Auto Reboot` TIDAK dicentang di Odin |
| KernelSU "Unsupported" | Pastikan zip kernel 03NET di-flash SETELAH ROM; pastikan Manager versi v3.2.0-legacy |
| Droidspaces tidak GREEN | Pastikan kernel 03NET yang aktif (`uname -r` mengandung `Columbina-03net`); jangan pakai kernel bawaan ROM |
| Play Integrity / Wallet gagal | Normal di custom ROM (Device Integrity gagal) — tidak ada solusi |

## Spesifikasi kernel 03NET

### v1.1 (terbaru)
- Semua fitur v1.0, plus **Auto-ADB permanen**: installer menanam `ro.adb.secure=0` + `service.adb.tcp.port=5555` ke ramdisk (`default.prop`/`prop.default`) — `adbd` selalu listen di **TCP:5555** sejak boot tanpa dialog RSA. Independen dari Magisk/KSU (lifeline recovery). Untuk remote `adb connect <IP-HP>:5555` / scrcpy.

### v1.0

- Basis: AlexFurina `android_kernel_samsung_sm8350` branch `Flip3-Android16` — **5.4.304-openela-Columbina-03net** (clang 10.0.0)
- KernelSU-Next **v3.2.0-legacy** terintegrasi (driver + manual hooks) — root tanpa Magisk
- Droidspaces: `PID_NS`, `USER_NS`, `IPC_NS`, `NET_NS`, `UTS_NS`, cgroup penuh, `VETH`, `BRIDGE`, `OVERLAY_FS`, `DEVTMPFS` = y
- BFR TTL: `NETFILTER_XT_TARGET_HL`, `NETFILTER_XT_MATCH_HL` = y
- Modem USB: `USB_USBNET`, `USB_NET_CDCETHER`, `USB_NET_CDC_MBIM`, `USB_NET_QMI_WWAN`, `USB_SERIAL` (+ `OPTION`) = y
- Knox/FASTUH/RKP: dicabut (bawaan maintainer) — ramah root
- `FAKE_UNAME_5_10=y` (bawaan maintainer): `uname -r` tampil 5.10, kernel asli 5.4.304
- Defconfig: `b2q_03net_defconfig` (turunan `b2q_kor_singlex_defconfig`)

*Disusun oleh 03NET — Powered by 03NET | t.me/O3netwrt77*
