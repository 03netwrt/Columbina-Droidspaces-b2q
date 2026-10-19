# BUILD NOTES — Columbina-Droidspaces-b2q v1.0

Kernel custom 03NET untuk Galaxy Z Flip3 (SM-F711B, b2q, SM8350).
Build: 2026-10-19. Status: **SUKSES** (BUILD_EXIT:0).

## 1. Basis & versi

- Source: `AlexFurina/android_kernel_samsung_sm8350`, branch `Flip3-Android16`
  (commit `c9116916`, merge OpenELA linux-5.4.y 2026-09-25)
- Tree kerja: `~/workspace/alexfurina-sm8350/` (defconfig baru + patch KSU di tree ini)
- Versi hasil: `Linux version 5.4.304-openela-Columbina-03net`
  (clang 10.0.0, #1 SMP PREEMPT)
- `FAKE_UNAME_5_10=y` dipertahankan (bawaan maintainer) → `uname -r` tampil 5.10
- Knox/FASTUH/RKP: sudah dicabut maintainer (tidak disentuh)

## 2. Toolchain (terbukti dari build 5.4.86 lama)

- Lokasi: `~/workspace/b2q-kernel/build/toolchain/`
- Clang: `llvm-arm-toolchain-ship/10.0/bin/clang` → **clang 10.0.0**
  (butuh `LD_LIBRARY_PATH=$HOME/workspace/b2q-kernel/build/toolchain/lib` untuk libtinfo.so.5)
- GCC cross: `gcc/linux-x86/aarch64/aarch64-linux-android-4.9/bin/aarch64-linux-android-`
- `CLANG_TRIPLE=aarch64-linux-gnu-`, `REAL_CC=<clang>`
- Tambahan: `bin/python` → symlink ke `/usr/bin/python3` (di `~/workspace/b2q-kernel-03net/bin/`,
  harus di PATH — script dtbo.img memanggil `python`)

## 3. Defconfig: `arch/arm64/configs/vendor/b2q_03net_defconfig`

Turunan dari `b2q_kor_singlex_defconfig` + delta berikut (semua `=y`):

| Kategori | Opsi |
|---|---|
| Namespaces (Droidspaces) | `PID_NS`, `USER_NS`, `IPC_NS`, `UTS_NS`, `NET_NS` |
| Cgroups | `CGROUPS`, `CGROUP_SCHED`, `CGROUP_CPUACCT`, `CGROUP_DEVICE`, `CGROUP_FREEZER`, `CGROUP_PIDS`, `CGROUP_BPF`, `CGROUP_NET_CLASSID`, `MEMCG`, `BLK_CGROUP`, `NET_CLS_CGROUP` |
| Net/container | `VETH`, `BRIDGE`, `BRIDGE_NETFILTER`, `OVERLAY_FS`, `DEVTMPFS` |
| TTL/BFR | `NETFILTER_XT_TARGET_HL`, `NETFILTER_XT_MATCH_HL` |
| USB/modem | `USB_USBNET`, `USB_NET_CDCETHER`, `USB_NET_CDC_MBIM`, `USB_NET_QMI_WWAN`, `USB_SERIAL`, `USB_SERIAL_GENERIC`, `USB_SERIAL_OPTION` |
| IPC (Droidspaces) | `SYSVIPC`, `POSIX_MQUEUE` (dependensi wajib `IPC_NS` — tanpa ini `IPC_NS` ter-drop diam-diam oleh Kconfig!) |
| Root | `KSU=y`, `KSU_MANUAL_HOOK=y` |
| Identitas | `CONFIG_LOCALVERSION="-Columbina-03net"` |
| Dimatikan | `CONFIG_IKHEADERS` (butuh `cpio` yang tidak ada di VM; fitur /proc/kheaders tidak esensial) |

**DROPPED:** `CONFIG_CGROUP_NET_PRIO` — header `include/net/netprio_cgroup.h` di tree ini
stale (pakai `cgroup->id` yang sudah tidak ada di 5.4.304) → build error.
Tidak dibutuhkan Droidspaces (resep lama juga tidak pakai).

## 4. Integrasi KernelSU-Next v3.2.0-legacy (manual hooks)

**Temuan penting:** tree AlexFurina TIDAK mengandung KernelSU sama sekali
(tidak ada `drivers/kernelsu/`, tidak ada `CONFIG_KSU`, tidak ada di git history —
di semua branch). Klaim thread XDA "kernel ROM sudah include KSU-Next" berarti
AlexFurina mengintegrasikannya saat build (out-of-tree). Karena root ROM ini
bergantung pada KSU, integrasi dilakukan manual:

- Sumber: `rifsxd/KernelSU-Next`, tag **`v3.2.0-legacy`**
  (commit `9b08e88862000d5c50fb2e43a5b75123cf472e54`) — varian yang dipakai ROM XDA
- `drivers/kernelsu/` = copy dari `KSU-src/kernel/` + `KSU-src/uapi/` (header uapi
  wajib ada — `#include "uapi/app_profile.h"` tidak resolve tanpanya)
- `drivers/Makefile` += `obj-$(CONFIG_KSU) += kernelsu/`
- `drivers/Kconfig` += `source "drivers/kernelsu/Kconfig"`
- `drivers/kernelsu/` dijadikan git repo sendiri + tag `v3.2.0-legacy`
  → versi driver terdeteksi benar: **KSU 30151**, tag v3.2.0-legacy
  (tanpa ini fallback ke versi 1/v0.0.1 yang bisa membingungkan Manager APK)
- Manual hooks (mode `KSU_MANUAL_HOOK`, wajib untuk kernel < 5.10;
  terdeteksi build: `-- KernelSU-Next: Hook mode: Manual`):
  - `kernel/reboot.c` → `ksu_handle_sys_reboot()` (jalur komunikasi Manager APK!)
  - `fs/exec.c` (`do_execveat_common`) → `ksu_handle_execveat()`
  - `fs/open.c` (`do_faccessat`) → `ksu_handle_faccessat()`
  - `fs/stat.c` (`vfs_statx`) → `ksu_handle_stat()`
  - `fs/read_write.c` (`vfs_read`) → `ksu_handle_vfs_read()`
  - `drivers/input/input.c` (`input_handle_event`) → `ksu_handle_input_handle_event()`
  - Semua deklarasi `extern` di file-scope (bukan di dalam fungsi) agar lolos
    `-Werror=declaration-after-statement` (C90) yang diaktifkan Samsung.

## 5. Perintah reproduce persis

```bash
cd ~/workspace/alexfurina-sm8350
export PATH=$HOME/workspace/b2q-kernel-03net/bin:$PATH
export LD_LIBRARY_PATH=$HOME/workspace/b2q-kernel/build/toolchain/lib
TC=$HOME/workspace/b2q-kernel/build/toolchain
make -C $(pwd) O=$(pwd)/out ARCH=arm64 \
  CROSS_COMPILE=$TC/gcc/linux-x86/aarch64/aarch64-linux-android-4.9/bin/aarch64-linux-android- \
  REAL_CC=$TC/llvm-arm-toolchain-ship/10.0/bin/clang CLANG_TRIPLE=aarch64-linux-gnu- \
  DTC_EXT=$(pwd)/tools/dtc CONFIG_BUILD_ARM64_DT_OVERLAY=y \
  CONFIG_SECTION_MISMATCH_WARN_ONLY=y vendor/b2q_03net_defconfig
make -j2 -C $(pwd) O=$(pwd)/out \
  DTC_EXT=$(pwd)/tools/dtc CONFIG_BUILD_ARM64_DT_OVERLAY=y ARCH=arm64 \
  CROSS_COMPILE=$TC/gcc/linux-x86/aarch64/aarch64-linux-android-4.9/bin/aarch64-linux-android- \
  REAL_CC=$TC/llvm-arm-toolchain-ship/10.0/bin/clang CLANG_TRIPLE=aarch64-linux-gnu- \
  CONFIG_SECTION_MISMATCH_WARN_ONLY=y
# Image.gz dibuat manual (target make Image.gz butuh `bc` yang tidak ada):
gzip -9n -c out/arch/arm64/boot/Image > out/arch/arm64/boot/Image.gz
```

Output: `out/arch/arm64/boot/Image` (48 MB), `Image.gz` (18,7 MB).

## 6. Packaging AnyKernel3

- Template: `~/workspace/b2q-kernel/Shadow-Droidspaces-b2q-v1.0.zip` (AK3)
- `anykernel.sh`: `device.name1=b2q`, `supported.versions=15-16`
  (kernel.string menyebut 5.4.304-openela-Columbina-03net + KSU-Next v3.2.0-legacy)
- Isi ZIP: `Image.gz` (kernel saja — DTB/DTBO dipertahankan dari ROM via AK3),
  bukan `Image.gz-dtb` (target tidak ada di tree ini; DTB b2q berupa overlay dtbo)
- Hasil: `~/workspace/b2q-kernel-03net/Columbina-Droidspaces-b2q-v1.0.zip` (21,7 MB)

## 7. Masalah yang ditemui & solusi

1. **KSU tidak ada di tree** → integrasi manual penuh (poin 4). Kritis: tanpa ini kernel
   tidak punya root (ROM bergantung pada KSU-Next).
2. **`CONFIG_CGROUP_NET_PRIO` merusak build** (`cgroup->id` tidak ada di 5.4.304)
   → opsi di-drop (tidak dibutuhkan Droidspaces).
3. **`python: not found`** saat build `dtbo.img` → shim `bin/python` → python3.
4. **`-Werror=declaration-after-statement` (C90)** menolak `extern` di dalam fungsi
   → deklarasi dipindah ke file-scope, pemanggilan hook setelah deklarasi lokal.
5. **Const-qualifier** di `fs/exec.c` (`const char *const *` → `void *`)
   → cast eksplisit `(void *)`.
6. **Header `uapi/app_profile.h` tidak ditemukan** → copy `KSU-src/uapi/` ke driver.
7. **`bc`/`python` tidak ada** (`dtbo.img`, `timeconst.h`) → shim di `bin/` + copy ke
   `/usr/local/bin`; `Image.gz` dibuat manual via `gzip -9n` (target make butuh `bc`).
8. **`CONFIG_IPC_NS` ter-drop diam-diam** — dependensi `(SYSVIPC || POSIX_MQUEUE)`
   tidak terpenuhi (keduanya off di defconfig maintainer) → Kconfig menghapus
   `IPC_NS` tanpa warning. Solusi: aktifkan `SYSVIPC=y` + `POSIX_MQUEUE=y`
   (sesuai resep lama). Kritis untuk Droidspaces!
9. **`cpio: not found`** untuk target `kheaders_data.tar.xz` (`CONFIG_IKHEADERS`)
   → `IKHEADERS` dimatikan (non-esensial).

## 8. Catatan risiko

- **Belum diuji di HP** (tidak ada akses flash). Risiko bootloop tetap ada walau
  basis source sama dengan yang dipakai ROM XDA.
- Defconfig basis = varian Korea (`b2q_kor_singlex`); HP target XID — hardware sama,
  tapi belum terbukti di varian internasional.
- Manager APK harus dari release **`v3.2.0-legacy`** agar kompatibel dengan driver.
- `Image.gz` (tanpa dtb) mengandalkan DTB/DTBO bawaan ROM — benar untuk X-ROM,
  tapi kalau dipakai di ROM lain perlu verifikasi.
