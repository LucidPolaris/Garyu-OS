<p align="center"><img src="garyu-logo.png" alt="Garyu OS" width="160"></p>

<h1 align="center">Garyu OS</h1>

<p align="center">A lightweight, security-focused Linux distribution based on Debian 13 "Trixie" with the LXQt desktop.</p>

> **Status: experimental.** Garyu OS is under active development and has so far only been tested in virtual machines. Do not rely on it for anything important yet.

## Features

- **Debian 13 (Trixie) base**, x86_64 only, with Debian's security updates.
- **LXQt desktop** with SDDM, a minimal package set and a low memory footprint.
- **Calamares installer** with full-disk encryption (LUKS) available during install.
- **Hardened by default** (see below).
- **Fully branded:** boot splash, GRUB theme, login screen, installer, wallpaper, shell prompt, `os-release`, and login banner.
- **UEFI and legacy BIOS** boot, with Secure Boot via Debian's signed shim and GRUB.

## Security

| Area | What is configured |
|---|---|
| Mandatory access control | AppArmor enabled, with extra profiles installed |
| Firewall | nftables, default-deny inbound, all outbound allowed |
| Disk | LUKS full-disk encryption offered by the installer |
| Accounts | Root locked, sudo only, `su` limited to the `sudo` group, 12-char / 3-class password policy, `umask 027`, private home directories |
| Kernel parameters | `slab_nomerge init_on_alloc=1 init_on_free=1 page_alloc.shuffle=1 randomize_kstack_offset=on vsyscall=none debugfs=off` |
| sysctl | `kptr_restrict`, `dmesg_restrict`, restricted BPF and ptrace, `rp_filter`, no ICMP redirects, no core dumps and more (`99-garyu-hardening.conf`) |
| Kernel modules | Rarely used protocols, filesystems and FireWire disabled |
| Filesystems | `/tmp` is a tmpfs mounted `noexec,nosuid,nodev` |
| Browser | Firefox ESR launched through Firejail, with telemetry off and HTTPS-only mode on |
| Updates | `unattended-upgrades` for Debian security updates |
| Services | Bluetooth, Avahi, CUPS and SSH are not enabled |

### Trade-offs

- `init_on_free=1` costs a few percent of performance.
- `noexec` on `/tmp` can break some installers and build tools.
- `ptrace_scope=2` stops non-root users from using `gdb` or `strace` on their own processes.
- Firejail is itself a setuid program, which some consider a risk of its own.
- The live session user has passwordless sudo (standard live-system behaviour). The installed system does not.

## Download

Grab the latest ISO from the **[Releases page](https://github.com/<LucidPolaris>/garyu-os/releases)**.

Each release includes a `SHA512SUMS` file. Verify your download before use:

```bash
sha512sum -c SHA512SUMS --ignore-missing
```

Write the ISO to a USB stick with [Ventoy](https://www.ventoy.net/), [balenaEtcher](https://etcher.balena.io/), or `dd`, or boot it directly in a virtual machine.

## Testing

Boot the downloaded ISO in VirtualBox, VMware, Hyper-V or QEMU.

- Give the VM at least **2 GB RAM** and a **25 GB+ virtual disk** (the installer requires 10 GiB).
- Enable **EFI** in the VM settings to test the UEFI / Secure Boot path.

QEMU example:

```bash
qemu-img create -f qcow2 test.qcow2 30G
qemu-system-x86_64 -enable-kvm -m 4G -smp 2 -bios /usr/share/ovmf/OVMF.fd \
  -cdrom garyu-os.iso -drive file=test.qcow2,if=virtio
```

**Live session login:** user `garyu`, password `live`. An **Install Garyu OS** icon is on the desktop.

## Repository layout

| Path | Purpose |
|---|---|
| `auto/config` | live-build settings and live boot parameters |
| `build.sh` | Builds the ISO and brands the live boot menus |
| `config/package-lists/` | Packages installed in the image |
| `config/hooks/live/` | Build hooks (branding, hardening, cleanup) |
| `config/includes.chroot/` | Files copied into the image (sysctl, nftables, themes, Calamares config, LXQt defaults) |
| `branding-src/` | Source logo and wallpaper |
| `tools/gen_assets.py` | Generates icons, wallpapers, splash screens and installer art from `branding-src/` |

### Common changes

- **Add or remove packages:** edit `config/package-lists/garyu.list.chroot`.
- **Kernel parameters, live:** `KERNEL_HARDENING` in `auto/config`.
- **Kernel parameters, installed system:** `config/includes.chroot/etc/calamares/modules/grubcfg.conf`.
- **Change the logo or wallpaper:** replace the files in `branding-src/`, then run `python3 tools/gen_assets.py`.
- **Default language, keyboard and timezone:** `--bootappend-live` in `auto/config` (currently `en_GB`, `gb`, `Europe/London`).

## Known issues

- The installer, especially the encrypted install and first boot of the installed system, is still being tested.
- The custom SDDM login theme may fall back to SDDM's default theme if a Qt module is missing.
- The project has been verified in virtual machines only; real hardware and Secure Boot enrolment are untested.
- Project URLs in `os-release` and the installer are placeholders (`garyu-os.invalid`) and need replacing.

## Contributing

Issues and pull requests are welcome. When reporting a problem, say whether you ran Garyu OS in a VM or on real hardware, and include the output of `journalctl -b` for boot or login issues.

## License

The build scripts, configuration, hooks and themes in this repository are released under the [MIT License](LICENSE).

The **Garyu OS name, logo and wallpaper** are not covered by that license. All rights reserved.

Garyu OS images include software from Debian and other projects, each under its own license. This repository contains build instructions only.

## Acknowledgements

Built with [Debian](https://www.debian.org/), [live-build](https://wiki.debian.org/DebianLive), [LXQt](https://lxqt-project.org/), [Calamares](https://calamares.io/), [SDDM](https://github.com/sddm/sddm) and [Plymouth](https://www.freedesktop.org/wiki/Software/Plymouth/).
