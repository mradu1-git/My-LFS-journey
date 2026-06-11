# Linux From Scratch — Automation Suite

Building a completely custom, independent Linux operating system from source code up — fully automated, fully understood.

This repository tracks my LFS journey and houses a **staged Bash automation suite** that can reliably partition, download, configure, and compile the entire LFS system from scratch with a single command.

---

## 💻 Host & Virtualization Environment

The build environment is engineered inside a headless Virtual Machine to guarantee complete isolation from the daily-driver host OS.

| Property | Value |
|---|---|
| **Host OS** | Arch Linux (Kernel: Rolling) |
| **Guest OS** | Ubuntu 26.04 (Headless Server) |
| **Workflow** | SSH terminal from Arch host — full clipboard & mouse support |
| **VM RAM** | 8 GB (8 GB reserved for host: CLion, KDE Plasma) |
| **VM Swap** | 4 GB on `/dev/vda4` — OOM safety net |
| **Compilation** | `make -j4` — balanced against host resource limits |

---

## 🗂️ System Architecture Blueprint

Following modern Linux standards, the target filesystem uses a **merged `/usr` layout**. Root directories are symbolically linked to their `/usr` counterparts for a clean, BLFS-compatible binary environment:

```
/mnt/lfs/
├── bin  -> usr/bin
├── lib  -> usr/lib
├── lib64 -> usr/lib64
├── sbin -> usr/sbin
└── usr/
    ├── bin/
    ├── include/
    ├── lib/
    ├── lib64/
    ├── sbin/
    └── share/
├── sources/        # all source tarballs
└── tools/          # temporary cross-toolchain (deleted post-build)
```

---

## 🚀 Usage

```bash
# Clone the repo
git clone https://github.com/mradu1-git/linux-from-scratch.git
cd linux-from-scratch

# Set environment
export LFS=/mnt/lfs
su - lfs && source ~/.bashrc

# Run a stage
bash scripts/stage2/build-stage2.sh
```

---

## 📁 Repository Structure

```
lfs-build/
├── README.md
├── scripts/
│   ├── stage1/          # cross-toolchain (binutils, gcc, glibc, libstdc++)
│   ├── stage2/          # remaining temporary tools (bash, coreutils, xz...)
│   ├── stage3/          # final system inside chroot
│   └── chroot/          # chroot entry + virtual filesystem setup
├── patches/             # any custom or upstream patches applied
└── notes/
    └── troubleshooting.md
```

---

## 🏁 Build Progress

### Phase 1 — Host Preparation & Workspace Initialization
> *LFS Book: Chapters 2 & 4*

- [x] Established secure SSH bridge between Arch host and Ubuntu guest
- [x] Verified all host prerequisite tools (`bash`, `gcc`, `gawk`, `make`, `bison`, etc.)
- [x] Created target workspace at `/mnt/lfs`
- [x] Configured disk partitions — `/dev/vda3` for LFS root, `/dev/vda4` for swap
- [x] Remounted workspace with `suid` permissions (overriding Ubuntu's default `nosuid`)
- [x] Constructed merged-usr symbolic links (`bin`, `lib`, `sbin` → `usr/`)

---

### Phase 2 — Source Acquisition & Cryptographic Verification
> *LFS Book: Chapter 3*

- [x] Deployed upstream wget-list for all core tarballs and patches
- [x] Resolved mirror failures — Texinfo 7.2 and Ncurses 6.5 snapshot redirected to live mirrors
- [x] Upgraded Ncurses 6.5 → 6.6-20260608 (upstream snapshot rotation)
- [x] Executed full cryptographic validation via `md5sum -c md5sums`
- [x] **Result:** 100% of packages verified intact and uncorrupted

---

### Phase 3 — Cross-Toolchain Construction
> *LFS Book: Chapter 5 — Complete*

- [x] Created isolated `lfs` compilation user and group
- [x] Delegated ownership of `/mnt/lfs/sources` and `/mnt/lfs/tools` to `lfs` user
- [x] Initialized sandboxed shell profiles (`.bash_profile`, `.bashrc`) to prevent host environment bleeding
- [x] **Binutils pass 1** — cross assembler and linker (1.0 SBU baseline)
- [x] **GCC pass 1** — C/C++ cross-compiler targeting `x86_64-lfs-linux-gnu`
- [x] **Linux API headers** — kernel interface headers for glibc
- [x] **Glibc 2.42** — C standard library, the critical kernel-userspace bridge
- [x] **Libstdc++** — C++ standard library from GCC source tree
- [x] Sanity check passed — cross-compiler correctly links against `/lib64/ld-linux-x86-64.so.2`

---

### Phase 4 — Temporary Tool Suite (Cross-Compiled)
> *LFS Book: Chapter 6 — In Progress*

- [x] M4 — macro processor
- [x] Ncurses — terminal handling library
- [x] Bash — shell (with `/usr/bin/sh` symlink)
- [x] Coreutils — `ls`, `cp`, `mv`, `cat` and friends
- [x] Diffutils — file comparison tools
- [x] File — file type identification
- [x] Findutils — `find` and `xargs`
- [x] Gawk — pattern scanning and processing
- [x] Grep — pattern matching
- [x] Gzip — compression
- [x] Make — build automation
- [x] Patch — source patching
- [x] Sed — stream editor
- [x] Tar — archiving
- [x] Xz — LZMA compression
- [x] Binutils pass 2 — final cross linker and assembler
- [ ] GCC pass 2 — final cross compiler with full threading support

---

### Phase 5 — Chroot & Final System Construction
> *LFS Book: Chapters 7–10 — Pending*

- [ ] Virtual kernel filesystem setup (`/dev`, `/proc`, `/sys`, `/run`)
- [ ] Chroot environment entry
- [ ] Final directory layout (FHS compliant)
- [ ] ~65 final system packages inside chroot
- [ ] Linux kernel compilation and configuration
- [ ] GRUB bootloader installation
- [ ] System configuration (`/etc/fstab`, hostname, locale, network)
- [ ] **First boot**

---

## 🐛 Notable Issues & Resolutions

| Issue | Cause | Fix |
|---|---|---|
| `ftp.gnu.org` connection timeout | GNU FTP rate limiting | Switched to `ftpmirror.gnu.org` and `mirrors.kernel.org` |
| `ncurses-6.5-20250809.tgz` 404 | Upstream snapshot rotation | Downloaded latest `ncurses-6.6-20260608.tgz` and updated md5sums |
| `configure: cannot run test program while cross compiling` | gnulib runtime tests during cross-build | Passed `gl_cv_*` cache variables to skip runtime checks |
| `Permission denied` on `/mnt/lfs/usr` | Directories created as root, not delegated to `lfs` | `sudo chown -Rv lfs:lfs /mnt/lfs/usr` |
| Stray directory `{etc,` in `/mnt/lfs` | Brace expansion ran with `$LFS` unset | Deleted via `find /mnt/lfs -maxdepth 1 -name '{etc,' -exec rm -rf {} \;` |
| `make: No targets specified, no makefile found` | `configure` failed silently or ran in wrong directory | Always verify `echo $LFS_TGT` before configuring; run configure from inside `build/` subdirectory |

---

## 📖 References

- [Linux From Scratch Book (stable)](https://www.linuxfromscratch.org/lfs/downloads/stable/)
- [Beyond Linux From Scratch](https://www.linuxfromscratch.org/blfs/)
- [LFS Support Mailing List](https://www.linuxfromscratch.org/mail.html)