# Columbina-Droidspaces-b2q

Kernel custom **5.4.302-Columbina-03net** (rilis aktif v1.3.3) untuk **Samsung Galaxy Z Flip3 5G (SM-F711B, codename `b2q`)** — varian lain JANGAN.

Basis: [AlexFurina/android_kernel_samsung_sm8350](https://github.com/AlexFurina/android_kernel_samsung_sm8350) branch `Flip3-Android16` (Samsung OSS + OpenELA LTS, Knox/FASTUH/RKP dicabut maintainer).

**Yang ditambahkan 03NET:**
- **Full Droidspaces support** — `PID_NS`, `USER_NS`, `IPC_NS`, `NET_NS`, `UTS_NS`, cgroup penuh, `VETH`, `BRIDGE`, `OVERLAY_FS`, `DEVTMPFS`
- **KernelSU-Next v3.2.0-legacy terintegrasi** — root tanpa Magisk (driver + hooks di-build ke dalam kernel)
- **BFR TTL modifier** — `NETFILTER_XT_TARGET_HL` + `NETFILTER_XT_MATCH_HL`
- **Driver modem USB** — `USB_USBNET`, `USB_NET_CDCETHER`, `USB_NET_CDC_MBIM`, `USB_NET_QMI_WWAN`, `USB_SERIAL` (+OPTION)

Format: **AnyKernel3** ZIP — flash via TWRP, DTB/DTBO dipertahankan dari ROM. Khusus ROM Android 15–16 (diuji untuk X-ROM ZFlip5 Port / OneUI 8.0).

*Powered by 03NET — t.me/O3netwrt77*

> **Status 21 Oktober 2026 — v1.3.3 FINAL TERVERIFIKASI BERJALAN** ✅
> Kernel terpasang dan aktif di unit uji: `Linux version 5.4.302-Columbina-03net #3 SMP PREEMPT Tue Oct 20 21:50:44 WIB 2026` (cek via `/proc/version`). Sistem boot normal, KernelSU-Next siap dipakai.
> **Urutan install yang benar berubah** (lihat tutorial): provisioning/wizard WAJIB dengan kernel bawaan ROM dulu, kernel 03NET di-flash SETELAH masuk home screen. Kernel custom yang terpasang sejak boot pertama di data kosong memicu Rescue Party (bug #2 di bawah).
> **Dua bug terbuka, belum fix, perlu digarap lagi** — dirinci di bagian [Bug yang diketahui & pekerjaan lanjutan](#bug-yang-diketahui--pekerjaan-lanjutan):
> 1. Setelah reboot, user gagal unlock (`RUNNING_LOCKED` permanen) — cacat HAL **Weaver** di sisi ROM, bukan kernel.
> 2. Rescue Party pada boot pertama bila kernel custom ikut sejak awal provisioning.
> 3. Klaim **Auto-ADB DIBATALKAN** — terbukti mustahil di ROM ini (penjelasan teknis di bug #3).

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
| `Columbina-Droidspaces-b2q-v1.3.3.zip` (kernel 03NET, FINAL) | 17.864.035 B | [Release repo ini](https://github.com/03netwrt/Columbina-Droidspaces-b2q/releases) |
| Odin3 v3.14.4 | ±3 MB | samfw.com |
| `KernelSU_Next_v3.2.0_33129-release.apk` (Manager) | 9,8 MB | ⬇️ [Download langsung APK-nya di sini](https://github.com/KernelSU-Next/KernelSU-Next/releases/download/v3.2.0/KernelSU_Next_v3.2.0_33129-release.apk) (pilih yang **non-spoofed**; halaman [release v3.2.0 resmi](https://github.com/KernelSU-Next/KernelSU-Next/releases/tag/v3.2.0)) |
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
2. Di TWRP: **Install** → pilih file ROM → **Swipe to Confirm Flash** → tunggu selesai (±5 menit) — harus berakhir `script succeeded`.
3. **Reboot → System** dan **selesaikan wizard dengan kernel bawaan ROM** (langkah 6). ⚠️ **JANGAN flash kernel 03NET di sesi yang sama** — kernel custom sejak boot pertama di data kosong memicu Rescue Party (bug #2). Wizard WAJIB dilewati **tanpa membuat kunci layar sama sekali** (lewati PIN/pola/sidik jari/wajah).

## 5. Flash kernel 03NET (setelah wizard selesai)

1. Setelah HP masuk **home screen** (sistem ter-provisioning), kembali ke TWRP: matikan HP → **Volume Up + Power**, atau `adb reboot recovery` bila USB debugging sudah aktif.
2. Copy `Columbina-Droidspaces-b2q-v1.3.3.zip` ke HP (kalau belum) → **Install** → pilih file kernel → **Swipe to Confirm Flash** — harus berakhir `Done!`.
   - ⚠️ **Auto-ADB tidak ada lagi** (dibatalkan sejak v1.3.3 — terbukti mustahil di ROM ini, lihat bug #3). Aktifkan ADB manual: Opsi pengembang → Debugging USB, lalu otorisasi RSA **sekali** (tersimpan permanen di `/data`, tidak hilang oleh flash kernel).
3. **Reboot → System**.

## 6. Setup awal + verifikasi ROM

1. Selesaikan setup awal Android (WiFi, akun Google) — **tanpa kunci layar** (lihat peringatan langkah 4; bug Weaver #1 membuat unlock pasca-reboot bermasalah — sedang digarap).
2. Cek **Settings → About phone**: harus tertulis **OneUI 8.0 / Android 16**.
3. Download **`KernelSU_Next_v3.2.0_33129-release.apk`** (link download langsung ada di tabel file di atas ☝️) → copy ke HP → install (izinkan "Install unknown apps").
4. Buka aplikasi **KernelSU Next** → status harus **"Working"** (bukan "Unsupported").
   - Kalau tertulis "Unsupported": kernel tidak terpasang benar — ulangi langkah 5.
   - ⚠️ Pakai Manager dari **release v3.2.0 resmi** (satu keluarga dengan driver di kernel). Jangan ambil dari repo fork `rifsxd` (tidak ada rilis/APK di sana).
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

## Bug yang diketahui & pekerjaan lanjutan

### 1. 🔴 Setelah reboot, user tidak bisa unlock — `RUNNING_LOCKED` permanen (BELUM FIX)
- **Gejala:** boot berhenti di "Menyalakan ponsel..."; `dumpsys user` menunjukkan `State: RUNNING_LOCKED` dan `Unlock time: <unknown>` selamanya. Unlock hanya berhasil di dalam sesi wizard; setiap reboot berikutnya mengunci permanen.
- **Bukti (logcat):** `LockSettingsService: Unwrapping synthetic password for unsecured user 0` → `SyntheticPasswordManager: Using HIDL weaver service` → `WeaverHidlAdapter: Failed to get HIDL weaver config. status: 1, slots: 0` → `SyntheticPasswordManager: Protector uses Weaver, but Weaver is unavailable` → `Failed to unwrap synthetic password for unsecured user 0`.
- **Akar masalah:** protektor *synthetic password* terdaftar memakai **Weaver**, tetapi HAL Weaver port Flip5→Flip3 ini rusak (konfigurasi tidak valid, 0 slot). **Cacat sisi ROM — kernel 03NET terbukti tidak terlibat** (mekanisme framework/vendor; gejala identik di kernel bawaan).
- **Rencana perbaikan (belum dieksekusi):** dalam sesi yang masih ter-unlock (hasil provisioning) → root → matikan service HAL Weaver → buat PIN (protektor baru menjadi Gatekeeper-only, tanpa Weaver) → reboot → unlock via PIN → hapus PIN selagi Weaver mati → reboot harus auto-unlock seterusnya.

### 2. 🟠 Rescue Party pada boot pertama bila kernel custom terpasang sejak awal (WORKAROUND TERBUKTI)
- **Gejala:** install bersih (Format Data + ROM + kernel dalam satu sesi) → mentok logo → Rescue Party berjenjang → reboot berulang.
- **Bukti (`/proc/last_kmsg`):** nol kernel panic; `sec-camera-provider-4-0` abort (signal 6) sejak detik ke-7 (`exited 4 times before boot completed`); thread utama `system_server` macet — `WATCHDOG KILLING SYSTEM PROCESS: Blocked in handler on main thread (main) for 66s`; `boot_completed` tidak pernah tercapai. Jendela provisioning yang panjang di data kosong membuat kegagalan HAL + watchdog menjadi fatal.
- **Workaround (sudah jadi langkah resmi tutorial):** provisioning/wizard dengan **kernel bawaan ROM** lebih dulu; flash kernel 03NET **setelah** home screen. Di atas data yang sudah ter-provisioning, kernel 03NET boot normal (terverifikasi 21 Okt 2026).

### 3. ⚪ Auto-ADB — DIBATALKAN (mustahil di ROM ini)
- Ramdisk boot ROM ini hanya berisi `init` + `dpolicy` (tidak ada `default.prop`/`prop.default`), dan init **tidak memuat file prop apa pun dari ramdisk boot** (terbukti dari bedah image hasil flash v1.3.1). Satu-satunya sumber `ro.adb.secure=1` adalah `/system/build.prop`, dan `/system` berformat **erofs** (read-only, tidak bisa ditambal installer).
- v1.3.1 (buat prop file) dan v1.3.2 (path `/adb_debug.prop` + `/debug_ramdisk/`) keduanya **gagal di runtime**. Sejak **v1.3.3** seluruh tulisan Auto-ADB dicabut dari installer. **Plan B:** otorisasi RSA sekali (persisten di `/data/misc/adb`) + `adb tcpip 5555` per sesi bila perlu; ADB di TWRP tidak butuh RSA (lifeline tetap utuh).

### 4. ⚪ scrcpy wajib `--no-audio`
- ROM menolak `AudioRecord` untuk server scrcpy (`Cannot create AudioRecord`) — selalu jalankan `scrcpy --no-audio`. Link USB laptop yang tidak stabil juga membuat `adb push` file ROM 5,4 GB putus di tengah; salin via **MTP/Explorer** terbukti tuntas dan utuh (terverifikasi ukuran byte persis).

---

## Troubleshooting

| Gejala | Solusi |
|---|---|
| Bootloop setelah flash ROM | Masuk TWRP (Vol Up + Power), ulangi Format Data + flash ROM ulang |
| Rescue Party / mentok logo setelah install bersih | Urutan flash salah — kernel 03NET ikut sejak boot pertama. Ulangi dengan urutan tutorial: ROM → wizard dulu → kernel belakangan (bug #2) |
| Setelah reboot mentok "Menyalakan ponsel..." | Bug Weaver ROM (bug #1) — user tidak akan ter-unlock; perbaikan sedang digarap |
| TWRP tertimpa stock recovery | Ulangi langkah 2 — pastikan `Auto Reboot` TIDAK dicentang di Odin |
| KernelSU "Unsupported" | Pastikan zip kernel 03NET di-flash SETELAH ROM; pastikan Manager dari release v3.2.0 resmi (bukan dari repo fork) |
| Droidspaces tidak GREEN | Pastikan kernel 03NET yang aktif (`uname -r` mengandung `Columbina-03net`); jangan pakai kernel bawaan ROM |
| Play Integrity / Wallet gagal | Normal di custom ROM (Device Integrity gagal) — tidak ada solusi |

## Spesifikasi kernel 03NET

### v1.3.3 (terbaru — FINAL, TERVERIFIKASI BERJALAN 21 Okt 2026)
- Binary kernel **identik v1.3**; perubahan hanya di installer: seluruh tulisan Auto-ADB yang terbukti mati dicabut (diganti komentar penjelasan — lihat bug #3)
- ZIP 17.864.035 B, sha256 `a0f34ee7fee3413c0da6c5b5d43bb60f1a7784345aaded1f725eeb4a15982a68`
- Terverifikasi di unit uji: terpasang di atas sistem X-ROM yang sudah ter-provisioning → boot normal → `/proc/version` = `Linux version 5.4.302-Columbina-03net #3 SMP PREEMPT Tue Oct 20 21:50:44 WIB 2026`
- ⚠️ Ikuti urutan tutorial (wizard dulu, kernel belakangan) dan baca bagian Bug — bug Weaver pasca-reboot berasal dari ROM, bukan rilis ini

### v1.3 (perbaikan bootloop v1.2 — digantikan v1.3.3)
- **Perbaikan hook execveat KernelSU-Next**: v1.2 kernel panic di `ksu_handle_execveat_ksud` (terbukti dari `/proc/last_kmsg`: hook mengirim alamat userspace mentah `argv.ptr.native`, driver membacanya seolah struct kernel). v1.3 mengirim `&argv` sesuai interface resmi driver — terverifikasi sampai level kode mesin (objdump) sebelum rilis
- **Banner boot kernel**: baris kedua log kernel sekarang `Powered by 03NET-WRT` (string versi tidak diubah — vermagic modul vendor tetap cocok)
- Selebihnya identik v1.2: basis **5.4.302** @`9e6bd77c`, patch kABI 001+002, `CGROUP_PIDS` off, full Droidspaces + opsi netfilter kABI-aman, KernelSU-Next v3.2.0-legacy, TTL/HL, modem USB ~~Auto-ADB permanen~~ (**dibatalkan — lihat bug #3**)
- Kernel string: `5.4.302-Columbina-03net` — ⚠️ v1.2 **jangan di-flash** (bootloop); v1.0/v1.1 juga jangan (risiko kABI)

### v1.2 — ⚠️ JANGAN FLASH (bootloop, digantikan v1.3)
- Basis **5.4.302** (commit `9e6bd77c`) — basis persis kernel X-ROM v3.2 yang terbukti jalan
- **Patch kABI 001+002** (upstream DroidSpaces GKI): `SYSVIPC`/`POSIX_MQUEUE` aman tanpa geser struct
- `CGROUP_PIDS` **dimatikan** (tidak dibutuhkan Droidspaces)
- Opsi baru (kABI-aman): `MACVLAN`, `IP_SET` (+`HASH_IP`/`HASH_NET`, `XT_SET`), NAT66 (`IP6_NF_NAT`/`MASQUERADE`), `XT_MATCH_ADDRTYPE`, `REJECT` (v4+v6), `XT_TARGET_LOG`, `XT_MATCH_RECENT`, `TMPFS_POSIX_ACL`/`XATTR`
- Tetap: full Droidspaces, KernelSU-Next v3.2.0-legacy, TTL/HL (BFR), modem USB, ~~**Auto-ADB permanen** (TCP:5555)~~ (**dibatalkan — lihat bug #3**)
- Kernel string: `5.4.302-Columbina-03net` — ⚠️ v1.0/v1.1 **jangan di-flash** (risiko kABI)

### v1.1
- Semua fitur v1.0, plus **Auto-ADB permanen** *(dirancang; di ROM X-ROM terbukti tidak berfungsi — DIBATALKAN, lihat bug #3)*: installer menanam `ro.adb.secure=0` + `service.adb.tcp.port=5555` ke ramdisk (`default.prop`/`prop.default`) — `adbd` selalu listen di **TCP:5555** sejak boot tanpa dialog RSA. Independen dari Magisk/KSU (lifeline recovery). Untuk remote `adb connect <IP-HP>:5555` / scrcpy.

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
