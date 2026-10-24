# 🐧 Yocto & BitBake Architecture Lab (`yocto-bitbake-lab`)

> **Hands-on Embedded Linux Engineering & Architecture Lab**  
> Guided by Principal Embedded Linux & Yocto Architect principles.  
> Targeting automotive ECUs, heterogeneous SoCs (Renesas R-Car, TI TDA4, NXP S32G), and Autonomous Robotics (`meta-ros`).

---

## 🎯 Architecture & Lab Objectives

This laboratory is designed to build a deep, first-principles understanding of:
- **BitBake Task Scheduling Engine:** Understanding DAG (Directed Acyclic Graph) task dependency execution, parsing mechanisms, and recipe lifecycles.
- **Yocto Metadata & Layer Hierarchy:** Strict separation between BSP, Core distro layers, and proprietary application layers.
- **Cross-Compilation & Toolchains:** Multi-stage sysroots (`recipe-sysroot`, `recipe-sysroot-native`), cross-linking, and dynamic loaders.
- **Rootfs Generation & Systemd Integration:** User/group management, systemd service units, read-only root filesystems, and security hardening.
- **QEMU Target Emulation:** Rapid target prototyping and headless testing on `qemuarm64`.

---

## 🖥️ Target Host & Virtualization Environment

- **Host Machine:** macOS (Apple Silicon M-Series)
- **Virtualization Engine:** UTM Virtual Machine (Apple Hypervisor Framework)
- **Guest OS:** Ubuntu 24.04 / 22.04 LTS (`aarch64` / `ARM64`)
- **Virtual Hardware Resources:**
  - **vCPU:** 4 – 6 Cores
  - **RAM:** 8 GB+
  - **Swap Space:** 8 GB – 16 GB (mandatory to prevent OOM panic on parallel compilation)
  - **Storage:** 100 GB+ allocated on an internal `ext4` partition (never use macOS shared folders/APFS mounts for BitBake builds).

---

## 🗺️ Master Curriculum & Progress Checklist

### [ ] Phase 1: Environment & Host Setup (UTM ARM64)
- [ ] **1.1 Storage & Memory Hardening:**
  - [ ] Validate disk partitioning (`ext4`) and free storage (minimum 70–100 GB).
  - [ ] Configure and persist an 8 GB `/swapfile` to eliminate silent OOM kills (`SIGKILL` 137).
- [ ] **1.2 Host Build Dependencies:**
  - [ ] Install essential host packages (`build-essential`, `gawk`, `diffstat`, `texinfo`, `chrpath`, `zstd`, `lz4`, etc.).
  - [ ] Understand why each host utility is required by OpenEmbedded-Core.
  - [ ] Configure Git identity and set host locale to `en_US.UTF-8`.
- [ ] **1.3 Poky Reference Distribution Checkout:**
  - [ ] Select and checkout Poky LTS release (e.g., `scarthgap` or `kirkstone`).
  - [ ] Inspect upstream Poky directory structure (`bitbake/`, `meta/`, `meta-poky/`, `scripts/`).

---

### [ ] Phase 2: The First Minimal Bootable Image (`core-image-minimal`)
- [ ] **2.1 Workspace Initialization & Shell Hook:**
  - [ ] Dissect the `oe-init-build-env` shell script and environment variables (`PATH`, `BUILDDIR`, `BBPATH`).
  - [ ] Inspect the generated `build/` workspace anatomy.
- [ ] **2.2 Deep Dive into Configuration Files:**
  - [ ] Dissect `conf/bblayers.conf`: layer boundaries and evaluation order.
  - [ ] Dissect `conf/local.conf`:
    - [ ] `MACHINE = "qemuarm64"`
    - [ ] `DL_DIR` (centralized source download cache)
    - [ ] `SSTATE_DIR` (shared-state build accelerator)
    - [ ] Parallelism tuning: `BB_NUMBER_THREADS` & `PARALLEL_MAKE`.
- [ ] **2.3 First BitBake Build Execution:**
  - [ ] Trigger execution: `bitbake core-image-minimal`.
  - [ ] Monitor task generation, parsing phase, setscene tasks, and task compilation.
  - [ ] Inspect generated artifacts inside `tmp/deploy/images/qemuarm64/` (Kernel, Device Tree, Rootfs ext4).
- [ ] **2.4 Headless QEMU Emulation:**
  - [ ] Run target emulator: `runqemu qemuarm64 nographic slirp`.
  - [ ] Validate target kernel boot logs, userspace login, and networking.

---

### [ ] Phase 3: Writing Custom Recipes (`.bb`) & Task Lifecycle
- [ ] **3.1 Recipe Syntax & Anatomy:**
  - [ ] Learn core variables: `DESCRIPTION`, `HOMEPAGE`, `LICENSE`, `LIC_FILES_CHKSUM`.
  - [ ] Working directories demystified: `${WORKDIR}`, `${S}`, `${B}`, `${D}`.
- [ ] **3.2 The BitBake Task Lifecycle:**
  - [ ] Trace: `do_fetch` ➔ `do_unpack` ➔ `do_patch` ➔ `do_configure` ➔ `do_compile` ➔ `do_install` ➔ `do_package`.
- [ ] **3.3 Hands-on Recipe: Standalone C Application:**
  - [ ] Write a clean POSIX C sample (`hello-yocto.c`).
  - [ ] Implement recipe `hello-yocto_1.0.bb` using cross-compiler variables (`${CC}`, `${CFLAGS}`, `${LDFLAGS}`).
  - [ ] Build standalone recipe using `bitbake hello-yocto`.
  - [ ] Inspect recipe staging sysroot and package output (`tmp/work/.../packages-split/`).

---

### [ ] Phase 4: Production-Grade Custom Layer (`meta-custom-lab`)
- [ ] **4.1 Layer Architecture & Best Practices:**
  - [ ] The Open-Closed Principle: why upstream layers remain pristine.
  - [ ] Layer design principles for automotive and robotics platforms.
- [ ] **4.2 Scaffolding the Custom Layer:**
  - [ ] Create layer: `bitbake-layers create-layer meta-custom-lab`.
  - [ ] Register layer in `conf/bblayers.conf` using `bitbake-layers add-layer`.
  - [ ] Dissect `conf/layer.conf`: `BBPATH`, `BBFILES`, `BBFILE_COLLECTIONS`, `BBFILE_PRIORITY`, and `LAYERSERIES_COMPAT`.
- [ ] **4.3 Organizing Layer Structure:**
  - [ ] Set up recipe categories: `recipes-core/`, `recipes-apps/`, `recipes-extended/`.
  - [ ] Migrate `hello-yocto` into `meta-custom-lab`.
  - [ ] Integrate package into image via `IMAGE_INSTALL:append = " hello-yocto"`.
  - [ ] Boot target in QEMU and execute `/usr/bin/hello-yocto`.

---

### [ ] Phase 5: Image Customization, Systemd & Daemons
- [ ] **5.1 Custom Image Recipe:**
  - [ ] Create `recipes-core/images/custom-image-minimal.bb` inheriting `core-image`.
  - [ ] Manage user accounts, default passwords, and sudo permissions via `extrausers`.
- [ ] **5.2 Systemd Automation & Service Units:**
  - [ ] Configure `INIT_MANAGER = "systemd"`.
  - [ ] Write a custom background daemon in C / Python.
  - [ ] Package systemd `.service` unit and register via `systemd.bbclass` (`SYSTEMD_SERVICE`, `SYSTEMD_AUTO_ENABLE`).
- [ ] **5.3 Static Device Nodes & Networking:**
  - [ ] Configure `systemd-networkd` for static/DHCP IP assignment.
  - [ ] Validate autostart services inside QEMU boot logs.

---

### [ ] Phase 6: Debugging, Profiling & Extensible SDK (eSDK)
- [ ] **6.1 The BitBake Debugging Playbook:**
  - [ ] Inspecting task scripts: `run.do_compile`, `run.do_install`.
  - [ ] Inspecting task logs: `log.do_compile`, `log.do_install`.
  - [ ] Exploring environment variables: `bitbake -e <recipe> | grep ^VAR`.
  - [ ] Interactive task debugging shell: `bitbake -c devshell <recipe>`.
- [ ] **6.2 Live Development with `devtool`:**
  - [ ] Add workspace: `devtool modify <recipe>`.
  - [ ] Live patch source code, compile, and deploy to live target: `devtool deploy-target`.
  - [ ] Export patches back to layer: `devtool finish`.
- [ ] **6.3 Generating & Testing the Cross-SDK:**
  - [ ] Build standard SDK: `bitbake <image> -c populate_sdk`.
  - [ ] Install SDK on host and cross-compile standalone applications out-of-tree.

---

### [ ] Phase 7: Automotive & Production Readiness
- [ ] **7.1 Build Caching & Infrastructure:**
  - [ ] Configure local and remote `SSTATE_MIRRORS` and `PREMIRRORS`.
  - [ ] Build speed benchmarking & storage cleanup (`tmp/` hygiene).
- [ ] **7.2 Recipe Appending with `.bbappend`:**
  - [ ] Modify upstream configuration without touching upstream recipes.
  - [ ] Apply out-of-tree kernel configuration fragments (`.cfg`).
- [ ] **7.3 Path to Automotive & Autonomous Systems:**
  - [ ] Layer architecture overview for Renesas R-Car / NXP S32G BSPs.
  - [ ] Integration concepts for `meta-ros` (ROS 2 Humble / Jazzy) with Yocto.

---

## 📂 Repository Layout

```text
yocto-bitbake-lab/
├── README.md                      # Architecture guide & master checklist
├── docs/                          # In-depth architectural notes & debugging logs
│   ├── 01-environment-setup.md
│   ├── 02-bitbake-fundamentals.md
│   ├── 03-recipe-anatomy.md
│   └── 04-layer-management.md
└── meta-custom-lab/               # Custom OpenEmbedded/Yocto layer (Phase 4+)
    ├── conf/
    │   └── layer.conf
    └── recipes-apps/
        └── hello-yocto/
            ├── files/
            │   └── hello-yocto.c
            └── hello-yocto_1.0.bb
```

---

## 🛠️ Quick Reference Commands

| Operation | Command |
|---|---|
| Initialize Build Environment | `source poky/oe-init-build-env build/` |
| Build Target Image | `bitbake core-image-minimal` |
| Run Target in QEMU (No GUI) | `runqemu qemuarm64 nographic slirp` |
| View Recipe Environment | `bitbake -e <recipe-name> \| grep ^S=` |
| Open Development Shell | `bitbake -c devshell <recipe-name>` |
| Clean Build Artifacts | `bitbake -c cleansstate <recipe-name>` |
| List Registered Layers | `bitbake-layers show-layers` |

---

*Authored and maintained as part of the Yocto/BitBake Architecture Mastery Series.*
