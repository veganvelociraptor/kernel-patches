## PATCHES

#### Patches for [CachyOS Kernels](https://github.com/CachyOS/linux-cachyos).

###### Patches are only for the last, stable line (not EOL and not LTS). 
###### I offer no support. This repository is purely for my own experimentation.

###### And yes, LLM/AI/SI was used to help get patches to work with the latest kernel version. If you have a problem with that, feel free to re-base the original patches yourself.


Sources used:

- [CachyOS](https://github.com/CachyOS/linux)

- [xanmod-linux](https://github.com/xanmod/linux)

- [AUFS](https://github.com/sfjro/aufs5-standalone)

- [zstd](https://github.com/facebook/zstd/commits/dev/)

- [firelzrd](https://github.com/firelzrd)

- [The T2 SDE Project](https://svn.exactcode.de/t2/trunk/package/kernel/linux/)

- [Linux Kernel Mailing List Archive](https://lkml.org) / [Linux Kernel Mailing List Archive](https://lore.kernel.org/lkml/) / [Linux Kernel Mailing List Archive](https://patchwork.kernel.org)

### NOTES. The repository does not contain all patches from the above-mentioned sources - only those that I test myself or find interesting. I have not included those that are used and built into the CachyOS kernel by default (for example BBRv3, ADIOS)


### CPU Idle & Frequency

- **`0001-6.12-nap-v0.6.6.patch`** — **NAP** (Neural Adaptive Predictor) is an experimental **cpuidle governor**. It uses a small neural network (MLP) to predict how long the CPU will stay idle and selects the deepest suitable C-state.. Source: [Firelzrd](https://github.com/firelzrd/nap). Last update: 2026-11-01.

- **`0001-linux7.3-reflex0.4.0.patch`** — **Reflex** is a **cpufreq scaling governor** based on schedutil. It adds a fast path that measures actual demand from idle time and quickly ramps frequency when the CPU is falling behind, instead of waiting for PELT. Source: [Firelzrd](https://github.com/firelzrd/reflex). Last update: 2026-10-12.

- **`0001-cpuidle-Prefer-teo-over-menu-governor.patch`** — Makes **TEO** the default cpuidle governor instead of the classic *menu* governor. TEO focuses on timer events and generally performs better on modern systems after menu lost its interactivity boost. Source: [Christian Loehle (LKML)](https://lore.kernel.org). Last update: 2024-09-05.  
  *TEO is simpler and lighter than NAP; NAP is experimental and ML-based.*

- **`0001-bore-cachy.patch`** — **BORE** (Burst-Oriented Response Enhancer) scheduler modifications. Prioritizes short-burst interactive tasks (compositor, games, terminal) under background load. Source: [Firelzrd](https://github.com/firelzrd/bore-scheduler) / packaged by [Sirlucjan]. Last update: 2026-11-04.


### Filesystems & Memory

- **`0001-aufs-7.3-merge-v20260914.patch`** — Full **AUFS** (Advanced Multi-Layered Unification Filesystem) merge for 7.3. AUFS is a stackable union filesystem used by Docker, live systems, and some distributions to overlay multiple directories. Source: [CachyOS kernel-patches](https://github.com/CachyOS/kernel-patches/). Last update: 2026-09-29.

- **`0001-xswap-7.3.patch`** — **xswap (extendable virtual swap).** Adds a backing-free swap device whose pages live only in zswap. The cluster metadata is mapped lazily via a sparse vmalloc area so the device can grow and shrink with actual usage instead of reserving a fixed size. Source: [Baoquan He & Chris Li](https://lore.kernel.org/linux-mm/20261003003136.750633-1-hebaoquan@kylinos.cn/T/). Last update: 2026-10-03.

- **`0001-linux-7.3-kcompressd-unofficial-0.5.patch`** — **kcompressd** offloads anonymous folio compression and writeout from kswapd to a dedicated thread, reducing latency during reclaim. Source: [Firelzrd](https://github.com/firelzrd/kcompressd-unofficial) (forked from Qun-Wei Lin / MediaTek). Last update: 2026-11-07.

- **`0001-linux7.3-zram-ir-1.3.patch`** — **ZRAM Immediate Recompression (ZRAM-IR).** When CONFIG_ZRAM_MULTI_COMP is enabled, it tries higher-priority compressors immediately during the write path instead of accepting the first result and recompressing later. Controlled via /proc/sys/vm/zram_recomp_immediate. Source: [Firelzrd](https://github.com/firelzrd/zram-ir). Last update: 2026-09-07.

- **`0001-linux7.3-Re-swappiness-v1.3.patch`** — **Fixes MGLRU’s broken vm.swappiness behavior** by splitting the shared generation counter into per-type (anon/file) sequences, restoring swappiness as a proper global anon:file reclaim ratio. Source: [Firelzrd](https://github.com/firelzrd/re-swappiness). Last updated: 2026-05-01.


### Block I/O

- **`0001-block-patches.patch`** — Reduces lock contention in the **mq-deadline** and **BFQ** IO schedulers by skipping expensive merge lookups when the queue lock is already contended and by serializing BFQ dispatch. Source: Jens Axboe. Last update: 2024-01.

- **`0001-xanmod-block-rq_affinity.patch`** — **Force full multithreading of block I/O.** Sets `rq_affinity` so that I/O completion is always processed on the submitting CPU, improving IOPS on multi-queue devices. Source: [XanMod](https://github.com/xanmod/linux/commit/d0bc09370b4ad746f867a4d8e8cfd689859bbd6e). Last update: 2020.


### Compiler & CPU Optimizations

- **`0001-clang-patches.patch`** — Enables Clang’s **Polly** polyhedral loop optimizer. Polly reorders loops for better cache locality. Real-world gains are usually small (roughly 0–3% on kernel workloads); it is not a large speedup. Source: [CachyOS](https://github.com/CachyOS) (Peter Jung). Last update: 2024-12.

- **`0001-cpu-7.1-merge-graysky-s-lite-patchset.patch`** — Adds selectable **x86-64 ISA levels** (v2/v3) so the kernel can be built with `-march=x86-64-v3` etc. Source: Graysky / merged by Sirlucjan. Last update: 2025-08-20.

- **`0001-zstd-dev-patches.patch`** — Updates the in-kernel **Zstd** library from the upstream dev tree (1.5.7 → 1.6.0) plus a workaround for older GCC segfaults. Source: Zstd upstream / Sirlucjan. Last update: 2026-09-18.


### Network & TCP throughput

- **`0001-xanmod-xt_FLOWOFFLOAD.patch`** — **xt_FLOWOFFLOAD target.** Adds a netfilter target that offloads established connections to the hardware flow offload engine (or software fallback) to reduce CPU overhead on high-throughput traffic. Source: [XanMod](https://github.com/xanmod/linux/commit/7b97383ce2367b7e46cdd753670b3d269b5fad67) (original by Felix Fietkau). Last update: 2024.

- **`0001-xanmod-xt_FULLCONENAT-7.3.patch`** — **Full-cone NAT (xt_FULLCONENAT).** Implements full-cone NAT behaviour for netfilter so that any external host can send packets to a mapped port once the internal host has initiated outbound traffic. Source: [XanMod](https://github.com/xanmod/linux/commit/efc5ff078ac966ac851c9697e52d22259eed82e5). Last update: 2021.

- **`0001-cloudflare-tcp-collapse-max-bytes.patch`** — **Cloudflare TCP collapse limit.** Adds `net.ipv4.tcp_collapse_max_bytes` sysctl. When non-zero, the kernel skips the expensive TCP receive-queue collapse if the allocated memory already exceeds the threshold, avoiding latency spikes under high bandwidth-delay product. Source: [XanMod](https://gitlab.com/xanmod/linux/-/commit/2fa1527cf92ee7342ae4906fd953670225f3161b) (original by Cloudflare). Last update: 2022.


### Miscellaneous

- **`0001-acpi-call.patch`** — Embeds the **acpi_call** module. It allows userspace to call arbitrary ACPI methods (e.g. via `/proc/acpi/call`) for fan control, battery tweaks, or debugging on laptops. Source: [CachyOS](https://github.com/CachyOS) (Eric Naim). Last update: 2026-02-11.

- **`0001-futex-7.3-Add-entry-point-for-FUTEX_WAIT_MULTIPLE-op.patch`** — Adds the **FUTEX_WAIT_MULTIPLE** opcode. It lets a process wait on several futexes at once (used by Wine/Proton and some runtimes for more efficient synchronization). Source: Sirlucjan. Last update: 2024-10-25.

### T2 SDE Hotfixes & Other

These are mostly small compatibility, diagnostic, or hardware-specific fixes from the [T2 SDE](https://t2sde.org) project or community:

- `0108-smpboot-reuse-timer-calibration.patch` — Reuses TSC calibration on secondary CPUs. Source: Clear Linux.
- `amdgpu-vm_update-cpu.patch` — Disables doorbells for SDMA 5.2 (hang workaround).
- `cap_sys_nice_begone.patch` — Allows any AMDGPU context priority.
- `fixed-cmdline-limit.patch` — Fixes `MAX_ARG_STRLEN` scaling.
- `hotfix-clang-lto.patch.x86.patch` — Clang LTO target fix.
- `hotfix-devtmp-none-dev.patch` — Mounts devtmpfs as “none”.
- `hotfix-diagnostics.patch` — Quiets noisy kernel messages.
- `hotfix-fbcon-fonts.patch` — Better default framebuffer font selection.
- `hotfix-radeon-printk.patch` — Compacts Radeon logging.
- `hotfix-vesa-kern-cont.patch` — Fixes KERN_CONT in vesafb.
- `oom_kill_allocating_task.patch` — Prefer killing the allocating task on OOM.
- `up-next-dref-open.patch` — Avoids extra dentry ref/unref on open (performance). Source: Mateusz Guzik (LKML).
