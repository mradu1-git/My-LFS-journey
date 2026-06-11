# Linux From Scratch (LFS) Automation Journey

Welcome to my Linux From Scratch (LFS) project repository. This project tracks my journey of building a completely custom, independent Linux operating system from the source code up. 

My ultimate goal for this repository is to build a highly optimized, modular **staged automation suite** (written in Bash) that can reliably partition, download, configure, and compile the entire LFS system from scratch.

---

## 💻 Host & Virtualization Environment

To guarantee complete isolation and protect my daily-driver host operating system, the build environment is engineered inside a headless Virtual Machine optimized for resource constraints.

* **Host OS:** Arch Linux (Kernel: Rolling)
* **Guest OS (VM):** 26.04 (Headless Server Console)
* **Workflow Integration:** Managed completely via **SSH terminal connection** from the Arch Host to preserve mouse, touchpad, and clipboard functionality.
* **Hardware Allocation Architecture:**
    * **Total System RAM:** 16 GB
    * **VM Allocated RAM:** 8 GB (Leaving 8 GB free for active host operations like CLion/KDE)
    * **VM Swap Space:** 4 GB (Allocated on `/dev/sda2` as an Out-Of-Memory safety net)
    * **Compilation Strategy:** CPU throttled (`make -j3` or `make -j4`) to balance fast compilation passes against physical hardware limits.

---

## 🛠️ System Architecture Blueprint

Following modern Linux standards, the target filesystem employs a merged `/usr` directory layout. The root directories are symbolically linked to their `/usr` counterparts to ensure a clean, modern binary environment:

```text
/mnt/lfs/
├── bin -> usr/bin
├── lib -> usr/lib
├── sbin -> usr/sbin
└── usr/
    ├── bin
    ├── lib
    └── sbin
```

## MileStones Reached

### Phase 1: Host Preparation & Workspace Initialization (Chapters 2 & 4)
* Established secure SSH bridging between Arch Host and Ubuntu Guest.
* Verified host system prerequisite tools (`bash`, `gcc`, `gawk`, `make`, etc.).
* Created target workspace directory (`/mnt/lfs`).
* Configured disk partitions (`/dev/sda3` for LFS root storage, `/dev/sda2` for Swap).
* Remounted workspace with standard `suid` permissions enabled (overriding default Ubuntu `nosuid` flags).
* Constructed initial merged-usr system symbolic links (`bin`, `lib`, `sbin` pointing to `usr/`).

### Phase 2: Source Asset Acquisition & Cryptographic Verification (Chapter 3)
* Deployed upstream download lists for core system tarballs and patches.
* Resolved upstream mirror connection failures and dead URLs (Texinfo 7.2 and Ncurses 6.5 snapshots successfully redirected to official LFS mirrors).
* Executed absolute cryptographic validation via `md5sum -c md5sums`.
* **Result:** 100% of packages and kernel patches verified intact and uncorrupted.

### Phase 3: Sandbox & Cross-Toolchain Construction (Chapter 5 Complete)
* Created isolated `lfs` compilation group and user profile.
* Handed over directory permissions (`chown`) for `/mnt/lfs/sources` and `/mnt/lfs/tools` to the `lfs` user.
* Initialized sandboxed environmental profiles (`.bash_profile` and `.bashrc`) to prevent host environment bleeding.
* Manually compiled the primary cross-toolchain (GCC Pass 1, Binutils Pass 1, Linux API Headers, Glibc, Libstdc++, and essential auxiliary build tools).