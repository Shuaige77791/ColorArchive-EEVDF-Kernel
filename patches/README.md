# ColorArchive Pure EEVDF Kernel — Source Changes

This is the "EEVDF-only" (阉割版) kernel for OnePlus 10T / Ace Pro (PGP110, SM8475).
It runs upstream EEVDF scheduling **without** the OPLUS unfair-scheduling layer.

## What changed vs stock kernel

### 1. `kernel/sched/fair.c` — EEVDF scheduling

The core EEVDF port. Key functions added/modified:

- `se->slice` / `se->deadline` / `se->vlag` — EEVDF entity fields
- `pick_eevdf()` — pick the earliest-eligible virtual deadline
- `update_entity_lag()` — lag tracking
- `entity_eligible()` — eligibility check (vlag >= 0)
- `place_entity()` — placement of new/waking entities
- `update_deadline()` — deadline recomputation
- `wakeup_preempt_entity()` — wakeup preemption

**This variant (`#33` / `EEVDF_only`)**:
- `sysctl_sched_eevdf_unfair = 0` (unfair scheduling OFF)
- `sysctl_sched_eevdf_slice_weight = 0`
- `sysctl_sched_eevdf_vruntime_weight = 0`
- No uclamp injection
- No watermark clamp
- No swap clamp

### 2. `kernel/sched/debug.c` — debugfs nodes

Added `base_slice_ns` debugfs node:
```c
debugfs_create_u32("base_slice_ns", 0644, NULL, &sysctl_sched_base_slice);
```

### 3. `kernel/sched/sched.h` — declarations

```c
extern unsigned int sysctl_sched_base_slice;
```

## Build

```bash
make ARCH=arm64 SRCARCH=arm64 LLVM=1 LLVM_IAS=1 \
     CROSS_COMPILE=aarch64-linux-gnu- \
     KCFLAGS=-Wno-error -j$(nproc) Image
```

## Flash

### Fastboot (recommended)
```bash
adb reboot bootloader
fastboot flash boot_a boot_PGP110_kernel6_EEVDF_only_ksu.img   # with KernelSU
# or
fastboot flash boot_a boot_PGP110_kernel6_EEVDF_only_noksu.img # without KernelSU
fastboot reboot
```

### AnyKernel3 (recovery)
Flash the AK3 zip via TWRP/OrangeFox recovery.
**Note: KernelSU Manager cannot flash AK3 zips.**

## Kernel version

`#33 SMP PREEMPT` — 5.10.236-ColorArchive

## Files

| File | Size | Description |
|------|------|-------------|
| `boot_PGP110_kernel6_EEVDF_only_ksu.img` | 18,972,672 | EEVDF + KernelSU |
| `boot_PGP110_kernel6_EEVDF_only_noksu.img` | 18,493,440 | EEVDF, no KernelSU |
