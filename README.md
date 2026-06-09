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